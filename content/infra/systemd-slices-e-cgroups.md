---
title: "Slices do systemd e cgroups v2: limitando memória e CPU por grupo de processos"
description: "Units (service, scope, slice), cgroups v2 e delegação de controllers explicados do zero, com receitas para colocar language servers, containers Docker e comandos avulsos em slices com teto de memória e peso de CPU."
type: tecnologia
tags:
  - notes/infra
  - linux
  - systemd
  - cgroups
---

# 🧩 Slices do systemd e cgroups v2

Quando falta memória, o kernel escolhe quem sofre: quem vai para o swap, quem é desacelerado, quem morre. Sem
configuração, a escolha vale para a máquina inteira, e o desktop trava junto com o build que causou o problema.

Slices do systemd separam a máquina em grupos com limites próprios. O grupo que exagera sofre sozinho.

## 1. Units: service, scope e slice

O **systemd** é o primeiro programa do boot (PID 1). Ele age como o **gerente de um prédio**: liga os serviços na ordem
certa, reinicia quem cai, guarda os logs e decide quanto recurso cada um pode usar. Tudo o que ele gerencia é uma
**unit**, descrita por um arquivo INI. A extensão diz o tipo.

| Tipo | O que é | Analogia | Exemplo |
|---|---|---|---|
| **service** | Programa que o próprio systemd **inicia** e acompanha | Funcionário contratado pelo gerente | `docker.service` |
| **scope** | Grupo de processos que **outro programa já iniciou** e que o systemd passa a acompanhar | Visitante que entrou com alguém e recebe crachá na portaria | `kitty-10852-28.scope` (uma aba do terminal) |
| **slice** | **Pasta** que agrupa services e scopes e aplica limites ao conjunto | Andar do prédio com disjuntor próprio | `app.slice`, `system.slice` |

- Num **service**, o arquivo diz qual comando rodar (`ExecStart=`).
- Num **scope**, o processo já existe. O systemd só o adota num grupo. É assim que cada aba do terminal vira um scope.
- Uma **slice** não roda nada. Tudo dentro dela soma os consumos, e os limites dela valem para a soma.

## 2. cgroups v2: a árvore do kernel

Quem limita de fato é o kernel, com os **cgroups** (*control groups*). Um cgroup é um grupo de processos com
**contadores** (quanto usa) e **limites** (quanto pode usar). Os cgroups formam uma **árvore** em `/sys/fs/cgroup/`: cada
pasta é um grupo, cada arquivo é um contador ou um limite, e o limite do pai vale para a soma dos filhos.

O systemd traduz suas units nessa árvore. Recorte de `systemd-cgls`:

```text
-.slice                                  # raiz: a máquina inteira
├─user.slice
│ └─user-1000.slice                      # um usuário (UID 1000)
│   ├─user@1000.service                  # o systemd de usuário
│   │ ├─session.slice                    # desktop: compositor, áudio
│   │ ├─app.slice                        # apps e terminais
│   │ │ └─kitty-10852-28.scope
│   │ └─dev.slice                        # criada por você (seção 4)
│   └─session-3.scope                    # o login gráfico
├─init.scope                             # o PID 1
└─system.slice                           # serviços do sistema
  ├─docker.service
  └─docker-c0d7c13e....scope             # um container
```

Regra que explica quase tudo: **todo processo filho herda o cgroup do pai**. Se um terminal está num scope, tudo que
você roda nele também está.

```bash
cat /proc/self/cgroup
# 0::/user.slice/user-1000.slice/user@1000.service/app.slice/kitty-10852-28.scope
```

### Controllers e delegação

Cada recurso tem um **controller** no kernel: `memory`, `cpu`, `io`, `pids`. Um controller só vale num ramo se o pai o
**delegar** aos filhos.

```bash
cat /sys/fs/cgroup/cgroup.controllers
# cpuset cpu io memory hugetlb pids rdma misc
cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/cgroup.controllers
# cpu memory pids
```

O systemd de usuário costuma receber só `cpu memory pids`. Slices de usuário limitam memória e CPU, mas **não** disco:
um `IOWeight=` ali não tem efeito.

## 3. Sistema ou usuário?

| | systemd de sistema | systemd de usuário |
|---|---|---|
| Processo | PID 1, root | `systemd --user`, um por usuário logado |
| Comando | `sudo systemctl ...` | `systemctl --user ...` |
| Onde ficam as units | `/etc/systemd/system/` | `~/.config/systemd/user/` |
| Controllers | Todos, inclusive `io` | Os delegados (em geral `cpu memory pids`) |

Para seus programas interativos (editor, language servers, builds), use slices de **usuário**: não precisa de `sudo`.
Para serviços do sistema, como o Docker, a slice precisa ser de **sistema**.

Depois de criar ou editar um arquivo de unit, rode `systemctl [--user] daemon-reload`.

## 4. Os limites, um por um

`~/.config/systemd/user/dev.slice`:

```ini
[Unit]
Description=Ferramentas de desenvolvimento (builds, language servers, testes)

[Slice]
MemoryHigh=6G
MemoryMax=8G
CPUWeight=50
```

### MemoryHigh: o freio

Acima do valor, o kernel **desacelera** os processos do grupo e **recupera** memória deles (descarta cache, manda
páginas para o swap). Ninguém morre. É um carro que perde potência na subida: continua andando, mais devagar. A pressão
fica no grupo; o resto da máquina não sente.

### MemoryMax: o disjuntor

Limite duro. Se o freio não bastar e o grupo encostar no valor, o OOM killer mata um processo **de dentro do grupo**. O
desktop continua de pé. Deixe uma folga entre `MemoryHigh` e `MemoryMax` para o freio agir antes.

| | MemoryHigh | MemoryMax |
|---|---|---|
| Ao passar | Desacelera e recupera memória | Não deixa passar; OOM no grupo |
| Mata processo? | Não | Sim, só dentro do grupo |
| Contador em `memory.events` | `high` | `max`, `oom`, `oom_kill` |

### CPUWeight: peso, não teto

Peso relativo que só vale quando há **disputa**. O padrão é 100. Com a CPU cheia, um grupo com 50 contra um com 100
recebe ~1/3 da CPU. Com a CPU livre, usa 100%.

Não confunda com **`CPUQuota=`**, um **teto fixo** (`CPUQuota=400%` = no máximo 4 núcleos, mesmo com a máquina ociosa).
O teto desperdiça CPU quando ninguém mais quer usá-la. Prefira o peso.

### Hierarquia pelo nome

O hífen no nome cria a hierarquia: `dev-lsp.slice` é filha de `dev.slice`. Os limites da filha valem dentro dos da mãe:

```text
dev.slice            MemoryMax=8G   (soma de tudo)
├─dev-lsp.slice      MemoryMax=3G   (só language servers)
└─dev-build.slice    MemoryMax=6G   (builds e testes)
```

## 5. Receitas

### Um comando avulso com limite

O `systemd-run --scope` roda o comando **no próprio terminal** (com teclado e tela) e só o adota num scope novo. Com
`-p` você passa limites sem criar arquivo:

```bash
systemd-run --user --scope -p MemoryMax=4G -p CPUWeight=50 -- go test ./...
systemd-run --user --scope --slice=dev-build.slice -- npm run build
```

Sem `--scope`, o systemd criaria um service e rodaria o comando em segundo plano, longe do terminal.

### Language server numa slice própria (exemplo: gopls)

O editor inicia o language server e conversa com ele por stdin/stdout. Um script com o mesmo nome, antes do binário real
no `PATH`, faz o editor iniciar o servidor dentro da slice sem perceber:

`~/.local/bin/lsp-wrap/gopls`:

```bash
#!/usr/bin/env bash
exec systemd-run --user --scope --quiet --slice=dev-lsp.slice -- \
  env GOMEMLIMIT=1GiB "$(go env GOPATH)/bin/gopls" "$@"
```

```bash
chmod +x ~/.local/bin/lsp-wrap/gopls
export PATH="$HOME/.local/bin/lsp-wrap:$PATH"   # no rc do shell, antes do PATH do Go
```

O scope mantém stdin e stdout ligados ao editor. Vale para qualquer servidor (`rust-analyzer`, `tsserver`, `jdtls`).
Vários `gopls` abertos somam no limite da slice: um projeto grande não derruba os outros.

### Containers Docker numa slice

Containers **não** herdam o cgroup de quem rodou `docker run`. Quem cria o container é o `dockerd`, um serviço do
systemd de **sistema**, e o container nasce em `system.slice/docker-<id>.scope`. Por isso uma slice de usuário não pega
containers.

Com o driver de cgroup `systemd` (confira com `docker info --format '{{.CgroupDriver}}'`), o Docker aceita uma slice de
sistema como pai:

`/etc/systemd/system/containers.slice`:

```ini
[Unit]
Description=Containers de desenvolvimento

[Slice]
MemoryHigh=5G
MemoryMax=6G
CPUWeight=50
IOWeight=50
```

```bash
sudo systemctl daemon-reload
docker run --cgroup-parent=containers.slice postgres:17
```

No compose:

```yaml
services:
  db:
    image: postgres:17
    cgroup_parent: containers.slice
    mem_limit: 1g
```

Para valer em todo container, use `"cgroup-parent": "containers.slice"` no `/etc/docker/daemon.json`. Aqui o `IOWeight`
funciona, porque a slice de sistema tem o controller `io`. O `mem_limit` de cada serviço continua útil para que um
container não ocupe a slice inteira.

### Uma aplicação gráfica

Navegadores e IDEs também podem ir para uma slice. Muitos ambientes gráficos já iniciam apps em `app.slice`. Para um app
específico:

```bash
systemd-run --user --scope --slice=dev.slice -- code .
```

## 6. Como observar

```bash
systemctl --user status dev.slice                    # limites e consumo
systemd-cgtop /user.slice/user-$(id -u).slice/user@$(id -u).service/dev.slice
systemd-cgls --user                                  # árvore com processos
cat /proc/<pid>/cgroup                               # em que grupo está um processo
cat /sys/fs/cgroup/user.slice/user-$(id -u).slice/user@$(id -u).service/dev.slice/memory.events
```

| Contador em `memory.events` | Quando sobe | Leitura |
|---|---|---|
| `high` | O grupo passou do `MemoryHigh` e foi freado | Alguns por dia é normal; subindo sempre, o grupo está lento |
| `max` | Encostou no `MemoryMax` | Aperto real |
| `oom` / `oom_kill` | O kernel matou processo no grupo | Algo morreu; reduza a carga ou aumente o limite |

Cada grupo também tem o próprio [PSI](psi-pressure-stall-information.md) em `memory.pressure`, `cpu.pressure` e
`io.pressure`.

## 7. Ajustar e reverter

Testar sem editar arquivo (`--runtime` some no próximo login):

```bash
systemctl --user set-property --runtime dev.slice MemoryHigh=7G MemoryMax=9G
```

Permanente: edite o arquivo da slice e rode `systemctl --user daemon-reload`.

Reverter: apague o arquivo, rode `daemon-reload` e tire o `systemd-run ... --slice=...` dos wrappers. Processos já
abertos continuam no grupo até fecharem: processo não troca de cgroup sozinho.

## Glossário

- **Unit**: qualquer coisa que o systemd gerencia, descrita por um arquivo INI.
- **Service / Scope / Slice**: programa iniciado pelo systemd / processos adotados / pasta com limites.
- **cgroup**: grupo de processos do kernel com contadores e limites.
- **Controller**: módulo do kernel que mede e limita um recurso (`memory`, `cpu`, `io`, `pids`).
- **Delegação**: o pai repassa um controller aos filhos.
- **OOM killer**: rotina do kernel que mata um processo quando falta memória.

## Referências

- [Control Group v2 — documentação do kernel](https://docs.kernel.org/admin-guide/cgroup-v2.html)
- `man systemd.resource-control`, `man systemd.slice`, `man systemd.scope`, `man systemd-run`
- [Docker: cgroup parent](https://docs.docker.com/reference/cli/dockerd/#default-cgroup-parent)
- [PSI: Pressure Stall Information](psi-pressure-stall-information.md)
- [zram: swap comprimido em RAM](zram-swap-comprimido-em-ram.md)
