+++
date = "2015-12-01"
draft = false
title = "Error logging idioms in Go"

+++

Like most Go beginners, I was put off by the error handling idioms in Go when I started. Since Go does not support exception handling, errors are handled explicitly and inline. The following error handling idiom is common in a Go codebase -

```go
if _, err := method(); err != nil {
  // log the error and return
}
```

Now that I have embraced this idiom (and believe that this makes my code more robust as I have to think about all the error conditions up front), the next pattern to understand is the error logging responsibilities i.e. who does the logging. E.g. 

```go
func Foo(id int) {
  if err := Bar(id); err != nil {
    // log error here??
  }
}

func Bar(id int) error {
  if err := openBars(); err != nil {
    // log error here??
    return err
  }
  return nil
}
```

So there are possibly 3 options as to how the logging can be done -

- In both `Foo()` and `Bar()`. This is not desired as it will produce duplicate log messages.
- In `Foo()`. This makes debugging harder as the error context is lost i.e. where exactly in the source, the error occurred.
- In `Bar()`. However if we are writing a library, the user of the library should decide how to perform the logging. An external logger library can also be used. So we need to return the appropriate error only.

So it seems like the best place is in `Foo()` but the error returned from `Bar()` should have the appropriate context.

I have been using the following idiom in my Go projects.

```go
func Foo(id int) {
  if err := Bar(id); err != nil {
    err = fmt.Errorf("Foo: cannot fetch list of bars. id=%d error=%s", id, err)
    // Now log the error is desired.
  }
}

// Bar returns the list of bars.
func Bar(id int) error {
  if err := openBars(); err != nil {
    return fmt.Errorf("Bar: cannot fetch open bars. error=%s", err)
  }
  return nil
}
```

The error returned from `Bar()` has the context i.e. the function name and the cause of the failure.

This seems like a good idiom to begin with and as I use more of this the coverage and usefulness of this will be clearer.

- isubuz