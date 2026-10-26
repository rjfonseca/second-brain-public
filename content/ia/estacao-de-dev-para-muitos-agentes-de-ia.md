---
title: "Notebook que não trava com muitos agentes de IA"
description: "Receita prática para rodar vários agentes de código em paralelo num notebook Linux com pouca RAM: zram, um slice do systemd para os agentes, lock de comandos pesados, instruções para os agentes e monitoramento por PSI."
type: tecnologia
tags:
  - notes/ia
  - linux
  - performance
---

# 🖥️ Notebook que não trava com muitos agentes de IA

Rodar vários agentes de código ao mesmo tempo (Claude Code, Codex e afins) num notebook comum leva a um travamento
conhecido. Cada sessão sobe o próprio language server, servidores MCP, builds e testes. A RAM acaba, o swap em disco
enche, o disco satura e o desktop congela.

Esta é a receita que estabilizou um notebook Linux com **14 GB de RAM** e processador de 15 W **sem reduzir o número de
agentes**. Cada passo traz só o "como". O "porquê" está nas notas de [Infraestrutura & Linux](../infra/index.md).

---

## 1. Meça antes

```bash
cat /proc/pressure/memory /proc/pressure/io
free -h
swapon --show
```

O que olhar: a linha `full` e o campo `avg60` do PSI. Como ler está em
[PSI: Pressure Stall Information](../infra/psi-pressure-stall-information.md).

No pior momento, a máquina estava assim:

| Medida | Valor |
|---|---|
| PSI memória `full` avg60 | 30,9% |
| PSI IO `full` avg60 | 37% |
| Swap em disco (4 GB) | 100% em uso |
| RAM disponível | 2,8 GB |
| Load average | 42,8 em 12 threads |
| Disco `/` | 95% cheio |

Os maiores consumidores:

```bash
ps -eo rss,comm --sort=-rss | head -15
docker ps --format '{{.Names}}\t{{.Status}}'
docker system df
```

Cada sessão de agente custava ~1 GB (agente, `gopls`, MCPs). Além disso havia um antivírus varrendo todo arquivo aberto,
um Kafka esquecido num compose consumindo um núcleo inteiro e dezenas de GB em volumes Docker órfãos.

---

## 2. Ligue o zram

```bash
sudo apt install systemd-zram-generator
```

`/etc/systemd/zram-generator.conf`:

```ini
[zram0]
zram-size = min(ram / 2, 8192)
compression-algorithm = zstd
swap-priority = 100
```

`/etc/sysctl.d/90-zram.conf`:

```ini
vm.swappiness = 150
vm.page-cluster = 0
vm.watermark_boost_factor = 0
fs.inotify.max_user_watches = 524288
```

```bash
sudo systemctl daemon-reload
sudo systemctl start systemd-zram-setup@zram0.service
sudo sysctl --system
swapon --show   # /dev/zram0 com prioridade 100
```

O `max_user_watches` não é de memória: vários language servers e watchers ao mesmo tempo passam do limite padrão.

Por que cada valor: [zram: swap comprimido em RAM](../infra/zram-swap-comprimido-em-ram.md).

---

## 3. Coloque os agentes numa slice

`~/.config/systemd/user/agents.slice`:

```ini
[Unit]
Description=Agentes de IA e tudo o que eles disparam

[Slice]
MemoryHigh=8G
MemoryMax=10G
CPUWeight=50
```

```bash
systemctl --user daemon-reload
```

Uma função para abrir qualquer agente dentro da slice, em bash/zsh:

```bash
agent() {
  GOFLAGS=-p=4 GOMAXPROCS=6 GOMEMLIMIT=1536MiB \
    systemd-run --user --scope --quiet --slice=agents.slice -- "$@"
}
# uso: agent claude    |    agent codex
```

Em fish (`~/.config/fish/functions/agent.fish`):

```fish
function agent
    env GOFLAGS=-p=4 GOMAXPROCS=6 GOMEMLIMIT=1536MiB \
        systemd-run --user --scope --quiet --slice=agents.slice -- $argv
end
```

O efeito: os agentes somados, com tudo o que disparam, ficam com no máximo 10 GB. Se passarem disso, quem morre é um
processo de agente, não o desktop. As variáveis de Go limitam o paralelismo de builds e testes; troque pelo equivalente
da sua stack (`--max-workers` do Jest, `-j` do make).

Cuidados:
- Se o agente roda num sandbox (bubblewrap, firejail), o `systemd-run` fica **por fora** do sandbox.
- Sessões abertas antes da mudança, ou sem a função, ficam fora da slice. Confira com
  `cat /proc/$(pgrep -n claude)/cgroup`.
- Containers que o agente sobe **não** entram na slice. Limite-os no compose ou numa slice de sistema.

Como funciona, como ajustar os valores e as receitas para Docker e language servers:
[Slices do systemd e cgroups v2](../infra/systemd-slices-e-cgroups.md).

---

## 4. Um comando pesado por vez

Dois builds ou duas suítes e2e ao mesmo tempo levam a máquina ao swap, com ou sem slice. Um lock de arquivo põe os
comandos pesados em fila:

```bash
flock ~/.heavy.lock nice -n 10 go test ./...
flock ~/.heavy.lock nice -n 10 npx playwright test --workers=2
```

O `flock` espera a vez sozinho. Ponha o lock num caminho com escrita em **todos** os ambientes dos agentes: se cada
sandbox tem um `/tmp` privado, um lock em `/tmp` não é compartilhado.

---

## 5. Ensine os agentes

Os agentes só seguem a regra do lock se ela estiver nas instruções que eles leem. Trecho para `CLAUDE.md` ou `AGENTS.md`
(no global do usuário, para valer em todo projeto):

```markdown
# Comandos pesados: um de cada vez na máquina

- Comando pesado roda com o lock compartilhado: `flock ~/.heavy.lock nice -n 10 <comando>`.
- Pesado é: suíte e2e, `go test ./...` do repositório inteiro, lint do repositório inteiro, build de imagem Docker,
  `npm run build`, `docker compose up` de stack completa.
- Teste de um pacote ou de um arquivo roda sem lock.
- Playwright roda com no máximo `--workers=2`.
- Ao terminar a tarefa, derrube o que subiu: `docker compose down`, servidores de dev, watchers.
```

A última linha importa: o Kafka que consumia um núcleo inteiro tinha subido num compose de um agente e ficou dois dias
rodando.

Corte também o que cada sessão carrega sem precisar: um servidor MCP que duplica o que outro já faz (dois servidores de
LSP para a mesma linguagem, por exemplo) custa centenas de MB **por sessão**. Prefira MCPs instalados como binário a
`uvx`/`npx` a cada sessão, que sobem um processo intermediário extra.

---

## 6. Limpe disco e journal

```bash
docker builder prune -f
docker image prune -f
docker volume ls -f dangling=true    # revise antes de apagar
sudo journalctl --vacuum-size=500M
powerprofilesctl set balanced        # o modo econômico deixa um processador de 15 W lento demais
```

> ⚠️ `docker system prune --all --volumes` apaga **todo** volume sem container, inclusive bancos locais de projetos
> parados. Revise a lista e apague um por um quando houver dados que importam.

Para o journal não voltar a crescer, `/etc/systemd/journald.conf.d/size.conf`:

```ini
[Journal]
SystemMaxUse=500M
```

Se a máquina tem antivírus com varredura on-access, ele pode custar mais que um agente inteiro, porque varre cada
arquivo aberto por build e cache. Em máquina gerenciada, peça exclusões para pastas de código, caches e
`/var/lib/docker`, com varredura periódica como controle compensatório.

---

## 7. Acompanhe

Registre o PSI a cada 5 minutos com o timer de
[PSI: Pressure Stall Information](../infra/psi-pressure-stall-information.md) e
revise no fim do dia.

| Sinal | Atenção quando | O que fazer |
|---|---|---|
| PSI memória `full` avg60 | > 5% | Fechar sessões antigas de agente |
| PSI IO `full` avg60 | > 10% | Ver disco cheio e swap em disco; builds só com o lock |
| RAM disponível | < 2 GB | Fechar sessões; parar composes sem uso |
| Swap em disco usado | subindo | O zram não deu conta: liberar memória |
| `high` em `memory.events` da slice | sobe sempre | Agentes demais para a RAM |
| `oom_kill` em `memory.events` da slice | > 0 | Um agente ou build foi morto |

```bash
cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/agents.slice/memory.events
```

---

## Resultado

| Medida | Antes | Cinco dias depois |
|---|---|---|
| PSI memória `full` avg60 | 30,9% | 0,85% |
| PSI IO `full` avg60 | 37% | 0,00% |
| Swap em disco | 4 GB, 100% cheio | 0 B (só reserva) |
| zram | não existia | 1,9 GB em uso de 7,4 GB |
| Disco `/` | 95% | 63% (medido no dia da limpeza) |
| `oom_kill` na slice e no kernel | — | 0 |

Os números de "depois" são uma leitura pontual com vários agentes abertos, não uma média de vários dias. A melhora vem
do conjunto: zram, slice, lock, limpeza de disco e o fim dos containers esquecidos entraram juntos.

## Veja também

- [🐧 Infraestrutura & Linux](../infra/index.md)
- [Agentes de IA (AI Agents)](agentes-de-ia-ai-agents.md)
- [Model Context Protocol (MCP)](model-context-protocol-mcp.md)
