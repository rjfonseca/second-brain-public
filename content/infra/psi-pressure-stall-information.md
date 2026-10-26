---
title: "PSI: medindo quanto tempo o Linux perde esperando memória, CPU e disco"
description: "O que é o Pressure Stall Information (PSI) do kernel Linux, como ler some/full e avg10/avg60, por que é melhor que o load average e como registrar a pressão com um timer do systemd."
type: tecnologia
tags:
  - notes/infra
  - linux
  - performance
  - observabilidade
---

# 📉 PSI: Pressure Stall Information

## O que é

O **PSI** (*Pressure Stall Information*) é um contador do kernel Linux (4.20+) que mede **a fração do tempo em que os
processos ficaram parados esperando um recurso**: CPU, memória ou disco. É a resposta mais direta para "a máquina está
lenta, e por quê?".

```bash
cat /proc/pressure/cpu /proc/pressure/memory /proc/pressure/io
```

```text
some avg10=1.28 avg60=0.85 avg300=0.29 total=2433626
full avg10=1.28 avg60=0.85 avg300=0.28 total=2389299
```

## Como ler

| Campo | Significado |
|---|---|
| `some` | Pelo menos um processo estava esperando o recurso. Há disputa, mas o resto da máquina ainda andava |
| `full` | **Todos** os processos ativos esperavam ao mesmo tempo. Nesse tempo a máquina não fez nada útil |
| `avg10`, `avg60`, `avg300` | Média, em porcentagem, dos últimos 10 s, 60 s e 300 s |
| `total` | Tempo acumulado de espera, em microssegundos, desde o boot |

Exemplo: memória `full avg60=30.9` quer dizer que, no último minuto, durante ~18 s nenhum processo andou porque todos
esperavam o kernel liberar memória (reclaim e swap).

Para decidir, use o `avg60`. O `avg10` oscila demais; o `avg300` demora a reagir.

## Por que é melhor que o load average

O **load average** conta processos na fila (rodando ou esperando disco). Ele diz que há fila, mas não diz qual recurso
falta, e um load de 12 numa máquina de 12 threads pode ser normal.

O PSI conta **tempo perdido** e separa por recurso. Memória `full` alta com IO `full` alta conta uma história clara: falta
RAM, o kernel está usando o disco para compensar (swap e releitura de arquivos) e o disco virou o gargalo.

## Limiares de partida

| Sinal | Atenção quando | Leitura |
|---|---|---|
| Memória `full` avg60 | > 5% | Já se sente como travamento |
| Memória `some` avg60 | > 20% | Disputa começando |
| IO `full` avg60 | > 10% | Disco saturado: swap em disco, disco cheio, builds em paralelo |
| CPU `some` avg60 | > 40% | Processos esperando CPU com frequência |

São valores de partida para uma estação de trabalho. Em servidores, ajuste ao SLO.

## PSI por cgroup

Com cgroups v2, cada grupo tem o próprio PSI. Dá para saber **quem** está sob pressão, e não só que a máquina está:

```bash
cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/app.slice/memory.pressure
```

Ver [Slices do systemd e cgroups](systemd-slices-e-cgroups.md).

O `systemd-oomd` usa justamente esse PSI por cgroup: quando a pressão de memória de um grupo passa do limite por um
tempo, ele mata o filho que mais pressiona, antes do OOM killer do kernel. Veja a configuração com `oomctl`.

## Registrando a pressão com um timer

A degradação costuma ser lenta e só fica visível quando a máquina já não responde. Um timer do systemd de usuário que
grava o PSI a cada 5 minutos basta para ver a tendência.

`~/.local/bin/psi-log`:

```bash
#!/usr/bin/env bash
set -euo pipefail
out="${XDG_STATE_HOME:-$HOME/.local/state}/psi.log"
mkdir -p "$(dirname "$out")"
mem=$(awk '/^full/ {split($3, a, "="); print a[2]}' /proc/pressure/memory)
io=$(awk '/^full/ {split($3, a, "="); print a[2]}' /proc/pressure/io)
avail=$(awk '/MemAvailable/ {printf "%.1f", $2/1048576}' /proc/meminfo)
swap_disk=$(awk 'NR > 1 && $1 !~ /zram/ {u += $4} END {printf "%.1f", u/1048576}' /proc/swaps)
printf '%s mem_full=%s io_full=%s avail_gb=%s swap_disk_gb=%s\n' \
  "$(date -Is)" "$mem" "$io" "$avail" "$swap_disk" >> "$out"
```

`~/.config/systemd/user/psi-log.service`:

```ini
[Service]
Type=oneshot
ExecStart=%h/.local/bin/psi-log
Nice=19
IOSchedulingClass=idle
```

`~/.config/systemd/user/psi-log.timer`:

```ini
[Timer]
OnCalendar=*:0/5
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
chmod +x ~/.local/bin/psi-log
systemctl --user daemon-reload
systemctl --user enable --now psi-log.timer
tail ~/.local/state/psi.log
```

- `Nice=19` e `IOSchedulingClass=idle`: a coleta nunca disputa CPU ou disco com o trabalho.
- `Persistent=true`: se o notebook estava dormindo no horário, o systemd roda uma coleta assim que ele acorda. O cron
  simplesmente pula.
- A saída de erro vai para o journal: `journalctl --user -u psi-log.service`.

## Referências

- [PSI — documentação do kernel](https://docs.kernel.org/accounting/psi.html)
- `man systemd-oomd`, `man systemd.timer`
- [zram: swap comprimido em RAM](zram-swap-comprimido-em-ram.md)
- [Slices do systemd e cgroups](systemd-slices-e-cgroups.md)
