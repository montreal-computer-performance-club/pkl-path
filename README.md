# Pkl Path

This library provides a `Path` type for Pkl, allowing strongly typed
manipulation of paths.

## Usage

A path is stored in a canonical lexical form, such that path equality can be
checked by directly comparing them. Repeated and trailing separators are
respectively collapsed and removed, except for the root path `/`.

Paths are created with the factory function `unix`.

```pkl
import "Path.pkl"

config = Path.unix("/srv/app/appsettings.json")

config == Path.unix("/srv/app/.//appsettings.json")
config.path == "/srv/app/appsettings.json"
config.isAbsolute
config.parts == List("/", "srv", "app", "appsettings.json")
```

Relative paths are also representable.

```pkl
state = Path.unix("../state/")

state.path == "../state"
state.isRelative
state.parts == List("..", "state")
```

An empty path represents the current directory, like `.`.

```pkl
current = Path.unix("")

current == Path.unix(".")
current.path == "."
current.parts == List(".")
```

When a trailing separator matters for a command-line tool or wire format, render
it explicitly:

```pkl
target = Path.unix("target")

command = "rsync source/ \(target.withTrailingSlash())
```

## Limitations

Because Pkl is a configuration-as-code language, it provides only a subset of
the usual features for such libraries. `Path` exposes an API for manipulating
path names, but does not interact with an actual filesystem. The reason is that
most paths in a running Pkl program simply do not represent paths on the system
running it (and also it's not possible to interact with the OS like this in pure
Pkl).
