# Capabilities in Context

This repository provides *capcon.stp*, a [SystemTap] script that produces [strace]-like output for contextualizing capability checks.

## Example: [useradd]

Let’s run SystemTap in target process mode on the useradd command:

```console
$ sudo stap \
    -c 'useradd --create-home --user-group user' \
    -o examples/useradd.txt \
    -G follow_forks=1 \ # (a)
    -G with_context=1 \ # (b)
    capcon.stp
```

… where we asked SystemTap to (a) print a trace for each child process of useradd and (b) decode the file descriptor passed to any syscall that expects a file descriptor as its first argument. Each line of output resembles the following:

```
1687903555843481595 useradd<1296> fchown(7, 1001, 1001) = 0 {CAP_FOWNER:1, CAP_CHOWN:2} [7:/home/user/.bashrc]
```

From left to right, we have:

- the number of nanoseconds since the Unix epoch when the syscall returned;
- the command name;
- the thread ID, in angular brackets;
- the syscall’s name;
- the syscall’s arguments, in parentheses;
- the syscall’s return value;
- the capabilities that were checked during the syscall, in curly braces, and
- the file descriptor taken by the syscall and the corresponding file path, in square brackets.

From this line, we can infer that fchown triggers checks for `CAP_FOWNER` and `CAP_CHOWN` when changing the owner of a file (here, *.bashrc*) that has been copied from */etc/skel* to the new user’s home directory.

## Requirements and compatibility

*capcon.stp* has been tested with SystemTap v4.8 on a x86_64 machine running Linux v6.0.7. It may run on other versions of SystemTap not earlier than v4.1, which introduced the `@this1`, …, `@this8` macros.

*capcon.stp* doesn’t require debuginfo or guru mode.

## Assumptions

*capcon.stp* assumes that, on a per-thread basis, every syscall is atomic and every capability check happens within the context of the last syscall that was called and that hasn’t yet returned.

*capcon.stp* doesn’t make any assumptions about which syscalls do and do not involve capability checks. Probes are compiled for all available syscalls enumerated in SystemTap’s `nd_syscall` tapset.

## Alternative solutions

If you’re interested in only enumerating the capabilities checked by an application and don’t need to provide a reason for those checks, then you probably don’t need *capcon.stp*. Consider one of these alternatives instead:

- [capable], a script for the eBPF-based BCC project
- [libcap-ng]’s pscap utility
- [container_check.stp], another SystemTap script (presently outdated)

## License

*capcon.stp* is free and open source software licensed under [GNU GPL version 3 or any later version][copying].

[capable]: https://github.com/iovisor/bcc/blob/master/tools/capable.py
[container_check.stp]: https://sourceware.org/git/?p=systemtap.git;a=blob;f=testsuite/systemtap.examples/profiling/container_check.stp;h=217c2df90e5da593924f49c2943f1c88850d781b;hb=HEAD
[copying]: ./COPYING
[libcap-ng]: https://people.redhat.com/sgrubb/libcap-ng/
[strace]: https://strace.io/
[SystemTap]: https://sourceware.org/systemtap/
[useradd]: https://github.com/shadow-maint/shadow
