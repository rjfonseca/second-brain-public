---
type: tecnologia
description: "Anotações técnicas e referências sobre Google Cloud Go Release Format."
tags:
  - notes/go
---

# Google Cloud Go Release Format

Google has various packages in the same repository but creates releases named `<package>/<version>`, like [this](https://github.com/googleapis/google-cloud-go/releases).

They also use the file `internal/version/version.go` to store the internal version, but it is in the format `YYYYMMDD`.

```go
// Repo is the current version of the client libraries in this
// repo. It should be a date in YYYYMMDD format.
const Repo = "20200928"
```

So in [this example](https://github.com/googleapis/google-cloud-go/commit/9267b54da5ffd9499ddc1831d2a06b29e91373b5#diff-1041ef75e001bd17af8a1ec7a759803e) we have release `pubsub/v1.8.0` and Repo `20200928`. I'm not sure if `Repo` is important or it's just a nice-to-have for logging.
