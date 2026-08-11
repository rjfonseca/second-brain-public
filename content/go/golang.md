---
type: tecnologia
description: "Google's programming language for backend and cli tools. The language is called go but it's usually referenced as golang for disambiguation."
tags:
  - notes/go
---

# Golang

Google's programming language for backend and cli tools.

The language is called go but it's usually referenced as golang for disambiguation.

## Origins
Designed at Google in 2007 with focus on:

- static typing and run-time efficiency (like C),
- readability and usability (like Python or JavaScript),[23]
- high-performance networking and multiprocessing.

It was designed by Robert Griesemer, Rob Pike, and Ken Thompson.

The first pre-release of go is `2011-03-16`[^1] but it only  officially released on `2012-03-28`[^2].

The first blog post is on `2010-03-18` https://go.dev/blog/hello-world

- [Go at Google: Language Design in the Service of Software Engineering - The Go...](https://go.dev/talks/2012/splash.article)

## Main features

A language that is simple enough to have it in your head[^3].
Rob Pike said that they though a lot on developer experience[^3].

- Language
  - Simple to read
  - Only one way to do something
  - Commitment to compatibility with major version
- Static type
- Batteries included
- Compilation
  - Fast
  - Cross-compiling
  - Static
- Fast and efficient
- Built-in concurrency primitives: light-weight processes (go-routines), channels, and the select statement.
- Simple yet powerful package management
- Community

- [Rob Pike on Google Go: Concurrency, Type System, Memory Management and GC](https://www.infoq.com/interviews/pike-google-go/)

## Frameworks
Go's standard library is very comprehensive but you can find libraries in the wild.

Because programs are normally simple, it's more common to use toolkits. My favorite two are:

- [gorillatoolkit.org (gorillatoolkit.org)](https://www.gorillatoolkit.org/)
- [gokit.io (gokit.io)](https://gokit.io/)

But there are framework available:

- [Gin (github.com)](https://github.com/gin-gonic/gin)
- [Buffalo — Go Web Development Eco-system](https://gobuffalo.io/en/)
- [beego.me (beego.me)](https://beego.me/)

## Tests
Go has it's own testing framework.

Just create a file with `_test.go` suffix.

The test itself are go functions with `Test` prefix. See the example

```go
package morestrings

import "testing"

func TestReverseRunes(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := ReverseRunes(c.in)
		if got != c.want {
			t.Errorf("ReverseRunes(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

We use testify[^4] for asserting results and mocking.

Coverage:
```shell
go test -race -coverprofile=.cover . && go tool cover -html .cover
```

## How deployment works
Compile for the target platform and run it!

You can also package the binary in a scratch docker image.
## Coming from other languages
Go is not object oriented. But have amazing interface handling.

Go is not functional. But functions are first class.

Go has it's own way of handling problems. Some copy&paste is ok.

Errors are part of your domain.

Read existing code!
## Performance
Go's go-routine are very light (4kb) and run on the go's runtime so binaries can scale easily, without switching threads.

Go's memory handling makes use of memory be very efficient.

Go binaries are small.

## How to start
### The official site
The official golang site
- [golang.org (golang.org)](https://golang.org/)
- https://go.dev/ (new site)
- [Tutorial (golang.org)](https://golang.org/doc/tutorial/)
- [Learn to become a Go developer](https://roadmap.sh/golang)
- [Effective Go - The Go Programming Language](https://go.dev/doc/effective_go)
### Create your gopher
- [40E0424908840E40A157D473Bbf1359A7C786Ad6 (gopherize.me)](https://gopherize.me/gopher/40e0424908840e40a157d473bbf1359a7c786ad6)
### Familiarize yourself with standard packages
- [Pkg (golang.org)](https://golang.org/pkg/)
### Style guides
- [GitHub - smallnest/go-best-practices: resources for go best practices](https://github.com/smallnest/go-best-practices)
- [GitHub - bahlo/go-styleguide: 🏆 Opinionated Styleguide for the Go language](https://github.com/bahlo/go-styleguide)
- [The one-and-only, must-have, eternal Go project layout](https://appliedgo.com/blog/go-project-layout)
## Other nice posts and projects
- [7 GitHub projects to make you a better Go Developer💥 - DEV Community](https://dev.to/ankit01oss/7-github-projects-to-make-you-a-better-go-developer-2nmh)
- [dark corners of go. go is an easy to learn language with… | by Arash Cordi | ...](https://arash-cordi.medium.com/dark-corners-of-go-4e29aafcd4b9)
- [GitHub - mxschmitt/golang-tdd-example: Test driven development in Golang base...](https://github.com/mxschmitt/golang-tdd-example)
- [GitHub - quii/learn-go-with-tests: Learn Go with test-driven development](https://github.com/quii/learn-go-with-tests)
- [A curated list of awesome Go frameworks, libraries and software - Awesome Go ...](https://awesome-go.com/)
- [Common pitfalls in Go benchmarking - Eli Bendersky's website](https://eli.thegreenplace.net/2023/common-pitfalls-in-go-benchmarking/)

## Tips and Tricks
### Difference in float formatting
The function `strconv.formatFloat` prints the number differently than the typical `fmt.Printf`:
```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	v := 12.3456
	fmt.Printf("%f\n", v)
	fmt.Println(strconv.FormatFloat(v, 'f', -1, 64))
}
```

```text
12.345600
12.3456
```
## Some articles I like
- [go test and parallelism](https://bryce.is/writing/code/go-test-and-parallelism.html)
## Footnotes
[^4] https://github.com/stretchr/testify

[^3] https://changelog.com/podcast/100
[^2] https://golang.org/doc/devel/release

[^1] https://golang.org/doc/devel/pre_go1
