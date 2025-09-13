---
title: Improving ratt with new features and contributions
date: 2025-08-30 10:34:01
categories:
- gsoc
- debian
- salsa-ci
project_image: /assets/images/ratt.png
description: "Contributing to ratt (Debian's testing tool) with new features
and improvements, enhancing the Debian package testing infrastructure and CI/CD
workflows."
---

When working with Debian packages, ensuring that changes don't break other
software is essential. That's exactly where
[ratt](https://github.com/Debian/ratt/) ("Rebuild All The Things") comes in: it
takes a freshly built `.changes` file, finds all reverse build-dependencies,
and rebuilds them using the `.deb` files included.

Recently, as part of the GSoC 2025 program with [Salsa
CI](https://salsa.debian.org/salsa-ci-team/pipeline/), one of my goals was to
create a pipeline job that rebuilds reverse build-depends. For this, I chose
ratt, and along the way I contributed several improvements to the tool.

In this post, I'll go through the main enhancements I introduced to `ratt` and
how they make the tool more useful in practice.

## Direct reverse-dependencies (`-direct-rdeps`)

One of the most impactful improvements I worked on is the new `-direct-rdeps`
option.

Under the hood, ratt relies on `dose-ceve` to parse package metadata and
determine reverse dependencies. By default, it looks not only at direct reverse
build-dependencies but also at **all transitive ones**. While this ensures complete
coverage, it can be unnecessarily heavy: in many cases, only the immediate
reverse dependencies are actually affected by a change, and rebuilding
everything can be a waste of time and resources.

To address this, I introduced the `-direct-rdeps` option, which limits the
scope to direct reverse build-dependencies only. This significantly reduces the
rebuild set while still giving meaningful feedback for most changes.

For example, consider the package curl and its binaries:

| Option                              | Reverse Dependencies |
| ----------------------------------- | -------------------- |
| With `-direct-rdeps`                | 475                  |
| Without (all, including transitive) | 12,143               |

This simple option can make a huge difference, instead of spending hours
rebuilding thousands of packages, you can now focus on the few hundred that are
most likely to be affected.

#### Related commits: 

* [Add support for limiting reverse dependency depth
](https://github.com/Debian/ratt/commit/b4087ea0f60b2816e3e264e3728303edd0a1b88f)

## Adding support for more Debian releases

Previously, ratt worked reliably only with **unstable**. Now, the tool also
supports **experimental**, **stable**, and **oldstable**. This means developers
can validate packages across different archive suites, not just the latest
development branch.

Being able to test against *experimental* is especially valuable, since it lets
maintainers ensure a package is in good shape before moving it into *unstable*.

Similarly, checking *stable* and *oldstable* is important because these
releases are actively maintained with **security patches** and **updates**.
Being able to catch regressions in widely deployed environments makes ratt a
much more flexible and reliable tool for package testing.

#### Related commits:

* [Handle Distribution=experimental by feeding sid+rc-buggy to
dose-ceve](https://github.com/Debian/ratt/commit/ed31253fa5c34c553d2d11a11d000359e3549daa)

* [Generate correct sbuild command for
experimental](https://github.com/Debian/ratt/commit/38ffdd3bdac503c1c4db36236615c12f053069de)

* [Add stable/oldstable pockets to
dose-ceve](https://github.com/Debian/ratt/commit/bec96e836097e107bd7ba19e8d522d38b69bbbe0)

* [Support -updates/-security in sbuild for
stable/oldstable](https://github.com/Debian/ratt/commit/c74a5ea284d8e1320153aa65e2ce24300857b770)

## Support for chdist environments

Another improvement is support for using `chdist` environments directly. chdist
is a Debian tool that creates lightweight APT environments for different
releases, without needing to change the host system.

Before this, running ratt on a different release was really inconvenient and
required extra setup. Now, with chdist support, the APT trees provided by
chdist are more than enough.

This makes ratt more efficient and convenient, letting developers quickly test
against different Debian releases with minimal effort.

#### Related commits: 

* [Add support for using chdist to fetch apt index files
](https://github.com/Debian/ratt/commit/a314a5269cbbe446e3854d3900321e886d16af2e)


## Skipping known FTBFS packages

Another useful addition is the `-skip_ftbfs` option.

This option makes ratt query [udd.debian.org](udd.debian.org) (Ultimate Debian
Database) to check whether a reverse-dependency package is already marked as
**FTBFS** (Fails To Build From Source). If so, ratt simply skips rebuilding it.

This saves time and resources by avoiding rebuild attempts for packages that
are already known to fail.

#### Related commits: 
* [Add option to filter out reverse dependencies tagged as FTBFS by
udd.d.o](https://github.com/Debian/ratt/commit/ac6eb62d5df00421bfdbc31e24dc1ae27ff8db42)

## JSON output for dry-run mode

Another improvement is the addition of `-json` support for `-dry_run` mode

The `-dry_run` flag normally just prints the
[sbuild](https://wiki.debian.org/sbuild) command lines without actually
building anything. With the new `-json` option, these results are now available
in a structured **JSON** format, including the **package** name, **version**, and the
exact **command** that would be executed.

Example of the output:


```json
{
  "dry_run_builds": [
    {
      "package": "foo",
      "version": "x.x-x",
      "sbuild_command": "sbuild..."
    },
    {
      "package": "bar",
      "version": "x.x-x",
      "sbuild_command": "sbuild..."
    }
  ]
}
```

This is especially useful in the Salsa CI context, where pipelines often need
machine readable output. Instead of parsing logs or relying on ad-hoc text
processing, CI jobs can now consume structured data directly. This makes it
easier to filter, analyze, and handle results programmatically.

Having this option, ratt integrates more smoothly with automation and helps
pipelines make better use of resources.

#### Related commits: 
* [Add -json support for dry-run mode](https://github.com/Debian/ratt/commit/f6ccd0a4bc83a5e6ae2f3fcb7ae4db1751becd7b)

## Conclusion

I would like to thank *Michael Stapelberg (@stapelberg)* for reviewing all my
contributions throughout this process and for his feedback. His guidance was
essential in shaping these improvements and getting them merged into `ratt`.

Now, I'm also a **co-maintainer of the `ratt` package in Debian**, which you
can follow on the [Debian Package
Tracker](https://tracker.debian.org/pkg/ratt). Working closely with this tool
has been a great way to understand the process of reverse-dependencies in
Debian and how important they are for ensuring package quality.

I plan to keep contributing to `ratt` by fixing bugs, refining features, and
adding new ones to make it more robust for Debian contributors.

Maintaining `ratt` has been both a learning experience and a rewarding way to
give back to Debian, and I look forward to seeing how the tool continues to
evolve.
