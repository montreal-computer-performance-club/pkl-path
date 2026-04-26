# Pkl Path

This library provides a `Path` type for Pkl, allowing strongly typed
manipulation of paths.

## Normalization Policy

`pkl-path` does not automatically normalize `.` or `..` segments.

For example, `/srv/app/./config.pkl` preserves the explicit `./` segment. That
segment can carry useful intent in configuration, such as marking a pivot point
for a later `read*` operation or for another tool that interprets the path.

The library also does not automatically collapse `..` segments. Lexically,
`/a/b/../c` may look equivalent to `/a/c`, but that equivalence can be false on
real filesystems because of symlinks, mount points, bind mounts, chroot-like
contexts, or remote path semantics. Since Pkl configuration often targets
systems other than the machine evaluating the config, this library preserves the
path the user wrote unless an explicit lexical transformation is requested.

## Limitations

Because Pkl is a configuration-as-code language, it provides only a subset of
the usual features for such libraries. `Path` exposes an API for manipulating
path names, but does not interact with an actual filesystem. The reason is that
most paths in a running Pkl program simply do not represent paths on the system
running it (and also it's not possible to interact with the OS like this in pure
Pkl).
