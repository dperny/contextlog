# `contextlog` - Logging With Contexts

`contextlog` is a fork of `github.com/docker/swarmkit/log`. It's been broken out
into its own codebase here so that users don't have to vendor (the part of)
swarmkit to use this functionality

The purpose of `contextlog` is to attach a logrus logger to a context, so that
fields attached to the logger can be passed through the call stack.

## Usage

```go
import (
    "context"

    // you're going to be using the package name pretty often, so I recommend
    // exporting it as "log"
    log "github.com/dperny/contextlog"
)

func SomeSubFunc(ctx context.Context) {
    // inherits the foo=baz from SomeFunc
    // prints with foo=baz and iteration=i (for whatever iteration)
    log.Info("Hello, SomeSubFunc!")
}

func SomeFunc(ctx context.Context) {
    // shadows the ctx variable with a new one
    ctx := log.WithField(ctx, "foo", "baz")

    // prints with foo=baz
    log.Info("Hello, SomeFunc!")
    for (i := 0; i < 3; i++) {
        ctx := log.WithField(ctx, "iteration", i)
        SomeSubFunc(ctx)
    }
}

func main() {
    // create a new logger attached to ctx, with a parent context.Background()
    ctx := log.WithField(context.Background(), "foo", "bar")

    // Prints with foo=bar
    log.G(ctx).Info("Hello, world!")

    // call SomeFunc with this new context
    SomeFunc(ctx)

    // Still prints with foo=bar, the original logger has not been modified
    log.G(ctx).Info("Goodbye!")
}
```
