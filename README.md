# Pkl Path

This library provides a `Path` type for Pkl, allowing strongly typed operations,
such as syntactical normalization, splitting, and joining.

## Usage

Paths are created with the factory function `unix`.

```pkl
import "Path.pkl"

config = Path.unix("/opt/app/appsettings.json")

config == Path.unix("/opt/app/.//appsettings.json")
config.path == "/opt/app/appsettings.json"
config.isAbsolute
```

`parts` is a list of typed `Component` values — `Anchor` for the root marker,
`Normal` for a regular segment, `ParentDir` for `..`, and `CurrentDir` for `.`.
Each component has a `toString()` method returning its rendered form.

```
config.parts == List(
  new Path.Anchor { value = "/" },
  new Path.Normal { value = "opt" },
  new Path.Normal { value = "app" },
  new Path.Normal { value = "appsettings.json" }
)
```

Relative paths are also representable.

```pkl
state = Path.unix("../state/")

state.path == "../state"
state.isRelative
state.parts == List(new Path.ParentDir {}, new Path.Normal { value = "state" })
```

An empty path represents the current directory, like `.`.

```pkl
current = Path.unix("")

current == Path.unix(".")
current.path == "."
current.parts == List(new Path.CurrentDir {})
```

When a trailing separator matters for a command-line tool or wire format, render
it explicitly:

```pkl
target = Path.unix("target")
command = "rsync source/ \(target.withTrailingSlash())"
```

A path's segments can be inspected without parsing strings yourself.

```pkl
config = Path.unix("/usr/share/config.tar.gz")

config.name == "config.tar.gz"
config.stem == "config.tar"
config.suffix == ".gz"
config.suffixes == List(".tar", ".gz")
config.parent() == Path.unix("/usr/share")
config.parents() == List(Path.unix("/usr/share"), Path.unix("/usr"), Path.unix("/"))
```

Two paths can be joined. A relative path is appended to the base; an absolute
path replaces it.

```pkl
base = Path.unix("/opt/app")

base.join(Path.unix("config.pkl")) == Path.unix("/opt/app/config.pkl")
base.join(Path.unix("/etc/override.pkl")) == Path.unix("/etc/override.pkl")
```

`startsWith` tests whether a path is rooted in another, comparing whole
components rather than character prefixes. `/srv/appfoo` does not start with
`/srv/app`, because `appfoo` and `app` are different segments.

```pkl
Path.unix("/srv/app/config").startsWith(Path.unix("/srv/app"))
!Path.unix("/srv/appfoo").startsWith(Path.unix("/srv/app"))
!Path.unix("foo").startsWith(Path.unix("/foo"))
```

`Path` is purely lexical: `..` segments are preserved rather than resolved
against the preceding segment. A Pkl program rendering configuration usually
describes paths on a remote machine, a container, or a different operating
system — there is no filesystem accessible to resolve `..` against. Even on
a local filesystem, a directory in the path could be a symbolic link, so
`foo/..` is not necessarily `.`. Lexical preservation is the only sound
behavior under both conditions.

```pkl
nested = Path.unix("/srv/app").join(Path.unix("../etc"))

nested.path == "/srv/app/../etc"
nested.parent() == Path.unix("/srv/app/..")
```

## Normalization and limitations

A path is stored in a canonical lexical form, such that path equality can be
checked by directly comparing them. Repeated and trailing separators are
respectively collapsed and removed, except for the root path `/`.

Because Pkl is a configuration-as-code language, it provides only a subset of
the usual features for such libraries. `Path` exposes an API for manipulating
path names, but does not interact with an actual filesystem. The reason is that
most paths in a running Pkl program simply do not represent paths on the system
running it (and also it's not possible to interact with the OS like this in pure
Pkl).

Because of this, normalization is not performed for `..`. The reason is that it
does more than "eat away" the previous directory name. Resolving a path means
visiting each segment in a sequence, and `..` is no different. You will go "up",
but not necessarily back to where you were if the last segment was not a
directory, but a symbolic link to a directory.

While some languages like Bash and [Go](https://pkg.go.dev/path/filepath#Clean)
do collapse `..` with the previous parent, we take the more cautious approach
of others such as Python's `pathlib` and Rust's standard library.
