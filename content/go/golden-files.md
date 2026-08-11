---
type: tecnologia
description: "Used in golang unit testing to store large input/output data. Example the input is the whole content of a file and the output is as large..."
tags:
  - notes/go
---

# Golden Files

## What is?
Used in golang unit testing to store large input/output data.

Example: the input is the whole content of a file and the output is as large or complex as the file.

## How it is done?
The [gofmt](https://github.com/golang/go/tree/master/src/cmd/gofmt) is a good example.

The files are stored inside the [testdata](https://github.com/golang/go/tree/master/src/cmd/gofmt/testdata) directory.

The [input](https://github.com/golang/go/blob/master/src/cmd/gofmt/testdata/import.input) files holds the input for the function and the [golden](https://github.com/golang/go/blob/master/src/cmd/gofmt/testdata/import.golden) files holds the expected response.

The [test file](https://github.com/golang/go/blob/master/src/cmd/gofmt/gofmt_test.go) iterates and compares the files.

```go
// TestRewrite processes testdata/*.input files and compares them to the
// corresponding testdata/*.golden files. The gofmt flags used to process
// a file must be provided via a comment of the form
//
//	//gofmt flags
//
// in the processed file within the first 20 lines, if any.
func TestRewrite(t *testing.T) {
	// determine input files
	match, err := filepath.Glob("testdata/*.input")
	if err != nil {
		t.Fatal(err)
	}

	// add larger examples
	match = append(match, "gofmt.go", "gofmt_test.go")

	for _, in := range match {
		out := in // for files where input and output are identical
		if strings.HasSuffix(in, ".input") {
			out = in[:len(in)-len(".input")] + ".golden"
		}
		runTest(t, in, out)
		if in != out {
			// Check idempotence.
			runTest(t, out, out)
		}
	}
}
```

## Tips
It plays very nice with the new [embed](https://golang.org/pkg/embed/) feature of go 1.16.
## Links:
- [This blog post](https://ieftimov.com/post/testing-in-go-golden-files/) by Ilija Eftimov
