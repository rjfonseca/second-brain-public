---
type: tecnologia
description: "A debugger for golang projects We need to publish a new port for the debug server and start the debugger in a headless state."
tags:
  - notes/go
---

# Delve

## What is?
A debugger for golang projects

## Using on docker-compose

We need to publish a new port for the debug server and start the debugger in a headless state.

```yaml
version: '3.7'
services:
  myservice:
    ports:
      - 9001:9001
    entrypoint:
      - dlv
      - debug
      - ./cmd/api
      - --build-flags
      - "-gcflags='all=-N -l' -tags=musl -mod=vendor"
      - --headless
      - --listen
      - ":9001"
      - --api-version
      - "2"
      - --accept-multiclient
```

Then we can connect by cli using:

```shell
dlv connect :9001
```

Or connect using an IDE, by executing `dap-debug` in emacs for example.

## Using the cli

The official guide is in [delve/README.md at master · go-delve/delve · GitHub](https://github.com/go-delve/delve/blob/master/Documentation/cli/README.md).

But the cli is self explanatory. just enter the command `help` to learn how to use it.

### Troubleshoot
#### File not found when using docker

This problem occurs because the debug server generates an absolute file path for the source but it refers to the file inside the container, and not on the host, where the cli is running.

You need to configure delve cli to substitute the path to fix the absolute file name from the container to the host.

Change `substitute-path` config in the config file ~`/.config/dlv/config.yml`:

```text
# Define sources path substitution rules. Can be used to rewrite a source path stored
# in program's debug information, if the sources were moved to a different place
# between compilation and debugging.
# Note that substitution rules will not be used for paths passed to "break" and "trace"
# commands.
substitute-path:
  # - {from: path, to: path}
  - { from: /go, to: /home/yourusername/go }
```

## Resources
- [Delve (github.com)](https://github.com/go-delve/delve)
- [Using The Go Delve Debugger From The Command Line (jamessturtevant.com)](https://www.jamessturtevant.com/posts/Using-the-Go-Delve-Debugger-from-the-command-line/)
- [Go Docker Delve Remote Debug.Html (golangforall.com)](https://golangforall.com/en/post/go-docker-delve-remote-debug.html)
