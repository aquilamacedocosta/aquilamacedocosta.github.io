---
title: GSoC 2025 Enhancing Salsa CI – Final Report
date: 2025-08-31 10:34:01
categories:
- gsoc
- debian
- salsa-ci
---

This summer I had the opportunity to participate in **Google Summer of Code
2025** with [Debian](https://www.debian.org/), working on a medium sized
project to enhance Salsa CI. My work started on May 8 and the program
officially concludes on September 1. It really doesn't feel like 16 weeks have
passed, but in this period I have learned a lot and grown as a developer.

During this time I also attended **DebConf25** in Brest, France, where I met my
mentor Otto Kekäläinen in person, along with members of the Salsa CI team. Some
people I already knew, and it's always nice to have those conversations
face-to-face.

[Salsa CI](https://salsa.debian.org/salsa-ci-team/pipeline/), is Debian's
continuous integration system, running pipelines on salsa.debian.org to
automatically build, test, and check packages. Its goal is to help maintainers
catch issues earlier, ensure policy compliance, and improve package quality
before uploads reach the Debian archive.

This post summarizes the main contributions I made during GSoC.

## Ratt (Rebuild All The Things) improvements

One of my main GSoC goals was to implement a job in Salsa CI to automatically
rebuild reverse build-dependencies. For that, I relied on ratt, the tool that
takes a freshly built package and rebuilds everything depending on it.

While working on this integration, I noticed several limitations/bugs in ratt
that made it harder to use in CI pipelines. To address this, I contributed a
series of improvements that expanded its functionality and made it more
reliable for Debian contributors. These enhancements will also be directly useful
for **Salsa CI**, since they power the new reverse build-dependencies job.

| Pull Request                                                                                                         | Nº of commits |
| -------------------------------------------------------------------------------------------------------------------- | ------------- |
| [Use apt-helper cat-file to handle compressed APT index files for dose-ceve](https://github.com/Debian/ratt/pull/25) | 1             |
| [Add -chdist option and refactor APT index retrieval logic](https://github.com/Debian/ratt/pull/27)                  | 2             |
| [Add option to filter out reverse dependencies tagged as FTBFS by udd.d.o](https://github.com/Debian/ratt/pull/29)   | 1             |
| [Add manpage for ratt and support for limiting reverse dependency depth](https://github.com/Debian/ratt/pull/30)     | 2             |
| [Add json output support for dry-run mode](https://github.com/Debian/ratt/pull/31)                                   | 1             |
| [Add -sbuild-keep-build-log option](https://github.com/Debian/ratt/pull/32)                                          | 1             |
| [Add support for experimental, stable, and oldstable](https://github.com/Debian/ratt/pull/33)                        | 4             |
| [Add reverse\_dep\_count to JSON output and fix typo](https://github.com/Debian/ratt/pull/35)                        | 2             |

For a detailed look at these changes, I wrote a specific blogpost focused on
**ratt improvements**, with examples and commit links, which you can find
[here](https://aquilamacedo.github.io/gsoc/debian/salsa-ci/2025/08/30/ratt-improvements/).

## Salsa CI improvements

### Reverse build-dependencies job

  One of the most important additions is a new job that can automatically rebuild
  all reverse build-dependencies of a package. This feature is powered by `ratt`,
  with the improvements I introduced earlier in the project.

  * **Issue**: [rebuild-all-reverse-dependencies
  job](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/178)
  * **Merge Request**: [Add reverse dependency build job using
  ratt (!613)](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/613)

  This job allows maintainers to:

  * Catch regressions earlier in Salsa CI, before uploading to the archive.

  * Make transitions smoother, since maintainers can proactively rebuild
  reverse dependencies and check for failures.

  The job is already functional, but the MR remains open while we discuss the
  safest way to enable it. Since it can spawn a child pipeline with potentially
  hundreds of builds, we need to carefully balance its usefulness with the risk
  of overloading Salsa's infrastructure.

### Autopkgtest on more architectures

  Extended autopkgtest coverage in Salsa CI to `i386`, `arm64`, `armel`, and
  `armhf`, improving package testing across multiple architectures.

  * [Enable autopkgtest on
  ARM](https://salsa.debian.org/salsa-ci-team/pipeline#enable-autopkgtest-on-arm)

### Licenserecon job

  Added a job that runs
  [licenserecon](https://salsa.debian.org/debian/licenserecon), checking for
  mismatches between `debian/copyright` and upstream source licenses.
 
  * [Enable licenserecon
  job](https://salsa.debian.org/salsa-ci-team/pipeline#enable-licenserecon-job)

### Debdiff job

  The debdiff job automatically compares the package source in the Salsa
  repository with the version in the Debian archive, generating a debdiff to
  highlight changes between them.

  * **Issue**: [Add a debdiff job](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/397)

  * **Merge Request**: [Add debdiff test job to compare with archive source
  package](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/605) (*waiting for review*)

### Faketime job

  Introduces two opt-in jobs that inject
  [**libfaketime**](https://github.com/wolfcw/libfaketime) during builds and
  autopkgtests, helping check if packages are **future-proof** (e.g. against
  the Year 2038 problem).

  * **Issue**: [Add optional faketime jobs to run build and autopkgtest on
  arbitrary future
  date](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/411)
  * **Merge Request**: [Add debdiff test job to compare with archive source
  package](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/599) (*close
  to being merged, I'm finalizing the last details*)

### Bugfixes

| Description                                                                                                        | Issue                                                                | Merge Request                                                                |
| ------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Temporarily disable reprotest until [#1108550](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1108550) is fixed | [#455](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/455) | [!619](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/619) |
| Fix use of deb-check-file-conflicts on unstable/experimental/trixie/questing                                       | [#445](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/445) | [!609](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/609) |
| Fix `apt-get` usage in wrap-and-sort job                                                                           | [#454](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/454) | [!620](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/620) |
| Add needs for arch-specific autopkgtest jobs and fix disable rules                                                 | [#476](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/476) | [!642](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/642) |


## Packaging contributions

Besides Salsa CI improvements, I also contributed to Debian packaging during
the GSoC period. These uploads not only fixed issues but also gave me the
perspective of a maintainer running Salsa CI pipelines in practice.


### Package **openvr**

  Closed [bug #1067724](https://bugs.debian.org/1067724). Required a small
  transition (easy since the package has no reverse-dependencies). OpenVR is a
  virtual reality API by Valve, used by the package
  [**gamescope**](https://tracker.debian.org/pkg/gamescope), which had dropped
  the **build-dependency** due to incompatibility with the old version. With
  this update, gamescope will include the build-dependency again in its next
  release. Thanks to Dylan Aïssi(@daissi) for handling the upload for me.

  * [Upload accepted on Debian
  tracker](https://tracker.debian.org/news/1660410/accepted-openvr-2121ds1-1exp1-source-amd64-into-experimental/)

### Package **ratt**

  Uploaded the new upstream version first to experimental, then to unstable. I
  am now co-maintaining the package together with **Otto Kekäläinen**, and I
  would like to thank him for reviewing and handling the uploads for me.

  * [Experimental upload Debian
  tracker](https://tracker.debian.org/news/1650881/accepted-ratt-00git20250702ac6eb62-1-source-into-experimental/)
  * [Unstable upload Debian
  trcker](https://tracker.debian.org/news/1654081/accepted-ratt-00git20250812e11cf12-1-source-into-unstable/)

### Package **curl** 

 Corrected inconsistencies in `debian/copyright`. Thanks to Samuel Henrique
 for handling this upload.

  * [Upload accepted on Debian
  tracker](https://tracker.debian.org/news/1660098/accepted-curl-8160rc2-1-source-into-unstable/)

### Package **waymore** 
    
 New upstream version + improvements, uploaded `6.1-1~exp1` to experimental.
 Thanks to Samuel Henrique for handling this upload.

  * [Upload accepted on Debian
  tracker](https://tracker.debian.org/news/1652226/accepted-waymore-61-1exp1-source-into-experimental/)

By working on these packages and running their Salsa CI pipelines myself, I
experienced the workflow from a maintainer's point of view. This gave me
valuable insight into the practical needs of maintainers and confirmed that the
CI improvements were solving real problems in day-to-day packaging work.

## Reflections

Looking back at this cycle of work, what stands out most are not only the
technical results but also the lessons learned along the way. In the Debian
ecosystem, progress does not come from isolated effort, it comes from
communication, reviews, and compromise. Learning to navigate different
perspectives, accept feedback, and refine ideas based on community input has
been as important as writing the code itself.

I realized that even small changes, when multiplied across Debian, make a big
difference. Improvements in Salsa CI or packaging may look minor alone, but
together they strengthen workflows and help maintainers, a reminder of the
value of collaboration and steady progress, and something I'll carry forward.

## Next steps

Although GSoC 2025 is ending, my contributions won't stop here. I plan to:

* Keep contributing to **Salsa CI** and **Debian packaging**.
* Work on adding **reverse dependency support for autopkgtest**, extending coverage beyond build-time testing.

As Seneca once said:

> *"Every new beginning comes from some other beginning's end."*

## Acknowledgments

I'm really grateful to my mentor Otto Kekäläinen, whose weekly meetings,
guidance, and insights were essential throughout the project. Thanks also to
Santiago Ruano Rincón for his reviews and insights, as well as to the Salsa CI
team and Debian developers for their feedback. I would also like to thank my
fellow GSoC participant Aayush Raj, who was also working on Salsa CI for Debian
this summer, for the collaboration and exchange of ideas.

Finally, special thanks to Debian and Google for making this program possible.
