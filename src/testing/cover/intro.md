# Finding Test Coverage

You can use Go’s built-in tool to generate a test report for the package
you’re testing by simply adding the `-cover` flag in the test command.

Using the flag `-coverprofile` will create a local coverage report file.

```bash

    go test -coverprofile=c.out

```

You can also use go tool cover to format the report. For example, the
`-html` flag can be used to covert to html.

```bash

    go tool cover -html=c.out -o=c.html

```
