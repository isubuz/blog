+++
date = "2015-12-02"
draft = false
title = "Custom level based Go logger"

+++

The standard logger in the Go library does not support level based logging. As with other things in Go, instead of using another 3rd party library, I wanted to try if I could write a *simple* wrapper on top of the `log` library to achieve what I want. It turned out to be fairly easy.

I wanted the log message to have the following format -

```
<date> <time> <level> <file:lineno:func> <message>

2015/12/02 18:05:07 INFO  [app.go:66:ServeHTTPC] Received request. method=GET url=/api/builds/133
```

The function context in the log message is based on the detailed error context which I discuss in a previous blog post. The function name can be generated automatically instead of passing it explicitly in the error message. 

```go
# log.go

package log

import (
  "fmt"
  "log"
  "os"
  "runtime"
  "strings"
)

var (
  debug = log.New(os.Stdout, "", log.Ldate|log.Ltime)
)

func shortFile(f string) string {
  v := strings.Split(f, string(os.PathSeparator))
  return v[len(v)-1]
}

func shortFunc(f string) string {
  v := strings.Split(f, ".")
  return v[len(v)-1]
}

func print(log *log.Logger, msg string, level string) {
  pc, file, line, _ := runtime.Caller(2)
  f := runtime.FuncForPC(pc)
  log.Output(3, fmt.Sprintf("%-5s [%s] %s", level, fmt.Sprintf("%s:%d:%s", shortFile(file), line, shortFunc(f.Name())), msg))
}

// Debug wraps the call to `log.Println()` on the debug logger.
func Debug(msg interface{}) {
  print(debug, fmt.Sprintf("%s", msg), "DEBUG")
}

// Debugf wraps the call to `log.Printf()` on the debug logger.
func Debugf(format string, args ...interface{}) {
  print(debug, fmt.Sprintf(format, args...), "DEBUG")
}
```

The calls to the standard logger is wrapped with a `debug` logger. We can create more logger objects for INFO, WARNING and ERROR messages. 

The key point is the call to `log.Output` and the `callDepth` passed to it. The hardcoded value of `3` ensures that the details of the caller of `Debug()` is printed.

To use this in another module -

```sh
import log "/path/to/log"

func foo() {
  log.Debug("Hello, world")
}
```

If the above pattern proves to be generic and useful, I will be creating a different project for this.

- isubuz