---
title: "zram: swap comprimido em RAM"
description: "Swap, page cache e zram explicados do zero: por que o swap em disco trava uma máquina com pouca RAM, como configurar o zram com systemd-zram-generator e quais parâmetros de sysctl ajustar."
type: tecnologia
tags:
  - notes/infra
  - linux
  - memoria
  - performance
---

# 🗜️ zram: swap comprimido em RAM

## 1. Memória anônima, page cache e swap

O kernel divide a RAM ocupada em dois tipos:

| Tipo | O que é | Analogia | Como liberar |
|---|---|---|---|
| **Memória anônima** | Dados que os programas criaram: heap, variáveis, abas do navegador | Papéis que você escreveu na mesa | Só guardando em outro lugar (swap) |
| **Page cache** | Cópia de arquivos do disco que o kernel guarda na RAM para ler rápido de novo | Livros da biblioteca na mesa | Descartar: o original continua no disco |

**Swap** é a área para onde o kernel manda memória anônima que ninguém está usando agora. É como guardar papéis numa
gaveta. Quando o programa precisa de volta, o kernel traz para a RAM (**swap-in**).

### Por que o swap em disco trava a máquina

1. Os programas pedem mais memória do que a RAM tem.
2. O kernel manda memória anônima para o swap em disco e descarta page cache.
3. Logo os programas precisam de volta o que saiu. O kernel lê do disco, falta espaço, manda outra coisa para o disco.
4. Swap e releitura de arquivos disputam o mesmo disco. O IO satura.
5. A CPU fica parada esperando o disco e o desktop congela.

O [PSI](psi-pressure-stall-information.md) mostra esse quadro: memória `full` e IO `full` altas ao mesmo tempo.

## 2. O que é zram

O **zram** é um módulo do kernel que cria um **dispositivo de bloco** (algo que o sistema trata como disco) cuja área
fica na **própria RAM, comprimida**. O `/dev/zram0` é usado como swap.

Analogia: em vez de levar os papéis para o arquivo no porão, você os dobra bem pequeno e guarda na gaveta da própria
mesa. Ocupa um pouco de mesa, mas cabe muito mais e você pega de volta na hora.

Por que vale gastar RAM para guardar RAM:
- Memória de programa comprime bem: muitos zeros, ponteiros e texto repetidos. Taxas de 2x a 4x são comuns.
- Comprimir e descomprimir na CPU leva microssegundos. Ler do disco sob disputa leva muito mais e trava o IO de todos.

### Medindo

```bash
zramctl --output-all
```

```text
NAME       DISKSIZE   DATA  COMPR ALGORITHM ZERO-PAGES  TOTAL COMP-RATIO MOUNTPOINT
/dev/zram0     7.4G 380.7M 126.7M zstd             454 130.1M     2.9256 [SWAP]
```

| Coluna | Significado |
|---|---|
| `DISKSIZE` | Tamanho máximo **antes** de comprimir |
| `DATA` | Quanto de memória foi guardado (original) |
| `COMPR` | Quanto isso ocupa comprimido |
| `TOTAL` | RAM real gasta, com as tabelas internas |
| `COMP-RATIO` | Original ÷ RAM gasta. Aqui, cada 1 MB de RAM guarda ~2,9 MB |
| `ZERO-PAGES` | Páginas só com zeros: o zram só marca, não guarda nada |

Os números crus, em bytes, ficam em `/sys/block/zram0/mm_stat`.

## 3. zram não é zswap

| | zram | zswap |
|---|---|---|
| O que é | Um **dispositivo de swap** separado, em RAM comprimida | Um **cache comprimido na frente** do swap em disco |
| Precisa de swap em disco? | Não | Sim: quando o cache enche, escreve no disco |
| Onde aparece | `swapon --show` lista `/dev/zram0` | `/sys/module/zswap/parameters/enabled` |

Se o problema é o disco, o zram é a escolha: o swap sai do disco. Use um ou outro. Com os dois, o zswap comprimiria de
novo o que já vai para um zram.

```bash
cat /sys/module/zswap/parameters/enabled   # N = desligado
```

## 4. Configurando com systemd-zram-generator

Um **generator** é um programa que o systemd roda bem cedo no boot para criar units a partir de arquivos de
configuração. O `systemd-zram-generator` lê `/etc/systemd/zram-generator.conf` e cria o `systemd-zram-setup@zram0.service`
(formata o dispositivo) e o `dev-zram0.swap` (ativa o swap).

```bash
sudo apt install systemd-zram-generator     # Ubuntu/Debian
sudo pacman -S zram-generator               # Arch
# Fedora já traz instalado e ligado
```

`/etc/systemd/zram-generator.conf`:

```ini
[zram0]
zram-size = min(ram / 2, 8192)
compression-algorithm = zstd
swap-priority = 100
```

- **`zram-size`**: tamanho **antes** de comprimir, em MiB. Aqui, metade da RAM com teto de 8 GiB. O zram só ocupa RAM
  conforme é usado; vazio, não gasta quase nada.
- **`compression-algorithm`**: os disponíveis estão em `/sys/block/zram0/comp_algorithm`.

  | Algoritmo | Compressão | Velocidade | Quando usar |
  |---|---|---|---|
  | `zstd` | Maior | Mais lento que lz4 | Falta RAM antes de faltar CPU |
  | `lz4` | Menor | Mais rápido | CPU disputada, RAM sobrando |

- **`swap-priority = 100`**: com mais de uma área de swap, o kernel enche primeiro a de **maior** prioridade. Um swap em
  disco com prioridade -1 vira reserva, usada só quando o zram enche.

Ativar sem reiniciar:

```bash
sudo systemctl daemon-reload
sudo systemctl start systemd-zram-setup@zram0.service
swapon --show
```

```text
NAME       TYPE      SIZE USED PRIO
/swap.img  file        4G   0B   -1
/dev/zram0 partition 7.4G 1.9G  100
```

Páginas que já estavam no swap em disco antes do zram não se movem sozinhas: voltam para a RAM quando algum programa as
acessa.

## 5. Parâmetros do kernel para swap em memória

O `sysctl` muda parâmetros do kernel em tempo de execução; arquivos em `/etc/sysctl.d/` valem a cada boot.

`/etc/sysctl.d/90-zram.conf`:

```ini
vm.swappiness = 150
vm.page-cluster = 0
vm.watermark_boost_factor = 0
```

```bash
sudo sysctl --system
```

**`vm.swappiness`** (0 a 200): quando falta RAM, o kernel escolhe entre mandar memória anônima para o swap ou descartar
page cache. O valor diz quanto o kernel acha que o swap **custa** comparado a reler arquivos.

| Valor | O que o kernel assume | Comportamento |
|---|---|---|
| 60 (padrão comum) | Swap custa mais que reler arquivo | Prefere descartar page cache |
| 100 | Custos iguais | Equilíbrio |
| 150 | Swap custa **menos** | Prefere comprimir memória ociosa no zram |

Com swap em disco, os dois caminhos usam o disco, e um valor baixo faz sentido. Com zram, o swap é RAM comprimida, mais
barato que reler arquivo. A documentação do kernel cita valores acima de 100 exatamente para esse caso.

**`vm.page-cluster`**: quantas páginas o kernel lê por swap-in, em potência de 2. O padrão 3 lê 8 páginas, apostando que
as vizinhas serão usadas (*readahead*). Ajuda em disco; no zram cada página é descomprimida à parte, então ler vizinhas
é trabalho perdido. Com 0, lê 1 página.

**`vm.watermark_boost_factor`**: quando a memória fica fragmentada, o kernel sobe por um tempo as "marcas d'água" de RAM
livre e libera memória extra. Com zram, essa liberação extra manda páginas para o swap sem necessidade. Com 0, o reforço
fica desligado.

## 6. Como observar

```bash
zramctl --output-all          # original, comprimido e taxa
swapon --show                 # áreas de swap, uso e prioridade
free -h                       # olhe a coluna "available", não "free"
cat /proc/pressure/memory     # PSI: "full" perto de 0 é o objetivo
```

`available` é a RAM que os programas ainda podem pegar sem swap (livre + page cache descartável). **Swap em disco
subindo é o sinal de que o zram não deu conta.**

## 7. Riscos e limites

- **zram não cria RAM.** Ele troca CPU por espaço. Se todos os programas usarem toda a memória ao mesmo tempo, não há o
  que comprimir.
- **Com tudo cheio, alguém morre.** O `systemd-oomd` (por pressão) ou o OOM killer do kernel escolhe um processo. Para
  controlar **quem**, use limites por grupo: [Slices do systemd e cgroups](systemd-slices-e-cgroups.md).
- **CPU extra.** Sob muito swap, comprimir aparece no uso de CPU. Num processador de baixo consumo é um custo real, mas
  menor que travar no disco.
- **Swappiness alto com swap só em disco é ruim.** Se desligar o zram, volte o `vm.swappiness` para 60.

## 8. Como reverter

```bash
sudo rm /etc/systemd/zram-generator.conf /etc/sysctl.d/90-zram.conf
sudo systemctl daemon-reload
sudo systemctl stop systemd-zram-setup@zram0.service   # faça com RAM disponível
sudo sysctl vm.swappiness=60 vm.page-cluster=3 vm.watermark_boost_factor=15000
```

O `stop` desativa o swap do zram; as páginas voltam para a RAM ou vão para o swap em disco. Os valores 60, 3 e 15000 são
os padrões comuns; confira os da sua distribuição com `sysctl` antes.

## Referências

- [zram — documentação do kernel](https://docs.kernel.org/admin-guide/blockdev/zram.html)
- [Parâmetros de /proc/sys/vm](https://docs.kernel.org/admin-guide/sysctl/vm.html)
- [zram-generator](https://github.com/systemd/zram-generator)
- `man zram-generator.conf`, `man zramctl`
- [PSI: Pressure Stall Information](psi-pressure-stall-information.md)
- [Slices do systemd e cgroups](systemd-slices-e-cgroups.md)
