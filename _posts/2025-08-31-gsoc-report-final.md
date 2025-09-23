---
title: GSoC 2025 Enhancing Salsa CI – Final Report
date: 2025-08-31 10:34:01
categories:
- gsoc
- debian
- salsa-ci
project_image: /assets/images/gsoc25_final.png
description: "Final report for GSoC 2025 project focused on enhancing Salsa CI
infrastructure, improving Debian package testing and continuous integration
workflows."
---

What happens when you spend a whole summer tinkering with Salsa CI, one of
Debian's CI systems? You end up rewriting old tools, breaking a few pipelines
(sorry), and hopefully leaving things a little smoother for thousands of
developers.

This summer I had the opportunity to participate in **Google Summer of Code
2025** with [Debian](https://www.debian.org/), working on a medium-sized
project to enhance Salsa CI. My work started on May 8 and the program
officially concludes on September 1. It really doesn't feel like 16 weeks have
passed, but in this period I have learned a lot and grown as a developer.

During this time I also attended [DebConf25](https://debconf25.debconf.org/) in
Brest, France, where I finally met my mentor Otto Kekäläinen in person after
weeks of video calls and we spent time chatting about Salsa CI, Debian
packaging, and just enjoying the chance to connect face to face.

Most of those conversations circled back to **Salsa CI**, the CI system I had
been working on all summer. For those unfamiliar, [Salsa
CI](https://wiki.debian.org/SalsaCI) is Debian's GitLab-based continuous
integration system, running pipelines on
[salsa.debian.org](https://salsa.debian.org) to automatically build, test, and
check packages. Its goal is to help maintainers catch issues earlier, ensure
policy compliance, and improve package quality before uploads reach the Debian
archive.

This post summarizes the main contributions I made during GSoC.

> My first battle was with ratt.

## Ratt (Rebuild All The Things) improvements

One of my main GSoC goals was to implement a job in Salsa CI to automatically
rebuild reverse build-dependencies, packages that build-depend on the one being
built. For that, I relied on [ratt](https://github.com/Debian/ratt/), the tool
that takes a freshly built package and rebuilds everything depending on it.

While working on this integration, I noticed several limitations/bugs in ratt
that made it harder to use in CI pipelines. To address this, I contributed a
series of improvements that expanded its functionality and made it more
reliable for Debian contributors. These enhancements will also be directly useful
for Salsa CI, since they power the new reverse build-dependencies job.

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

Seeing ratt evolve over the summer from a CLI-only helper to a CI-friendly tool
was one of the most rewarding parts of my work.

For a detailed look at these changes, I wrote a specific post focused on **ratt
improvements**, with examples and commit links, which you can find
[here](https://aquilamacedo.xyz/gsoc/debian/salsa-ci/2025/08/30/ratt-improvements/).

Many thanks to Michael Stapelberg (@stapelberg) for thoroughly reviewing
everything along the way.

## Salsa CI improvements

### Build reverse dependencies job

  One of the most important additions, is a new job that can automatically rebuild
  all reverse build dependencies of a package. This feature is powered by ratt,
  with the improvements I introduced earlier in this post.

  * **Issue**: [rebuild-all-reverse-dependencies
  job (#178)](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/178)
  * **Merge request**: [Add reverse dependency build job using
  ratt (!613)](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/613)
  * **Salsa CI Docs**: [Enable build reverse dependencies checking
  ](https://salsa.debian.org/salsa-ci-team/pipeline#build-reverse-dependencies)


  ![Build reverse dependencies job in action in ruby-graphql](/assets/images/ruby-graphql.png)
  <sub>This image shows the job running in Salsa CI, rebuilding reverse build dependencies of `ruby-graphql`.</sub>

  This job allows maintainers to:

  * Catch regressions earlier in Salsa CI, before uploading to the archive.

  * Make [transitions](https://wiki.debian.org/Teams/ReleaseTeam/Transitions)
  smoother, since maintainers can proactively rebuild reverse dependencies and
  check for failures.

   The job is now functional after the merge of
   [!613](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/613).
   Maintainers can enable build reverse dependencies jobs on demand to rebuild
   packages depending on theirs. Usage is limited to avoid overloading [Salsa's](https://wiki.debian.org/Salsa)
   infrastructure, with details explained in the [Pipeline
   documentation](https://salsa.debian.org/salsa-ci-team/pipeline#build-reverse-dependencies).

  This merge request was reviewed in person during DebConf25 by Bastien
  Roucariès, who suggested creating a JSON output option to make CI integration
  easier, an idea that later became part of ratt's improvements. Santiago Ruano
  Rincón and Otto Kekäläinen also provided valuable reviews and feedback during
  the process, helping shape the final result.

### Autopkgtest on more architectures

  Extended autopkgtest coverage in Salsa CI to `i386`, `arm64`, `armel`, and
  `armhf`, improving package testing across multiple architectures.

  * [Enable autopkgtest on
  i386](https://salsa.debian.org/salsa-ci-team/pipeline#enable-autopkgtest-on-i386)

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

  * **Merge request**: [Add debdiff test job to compare with archive source
  package](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/605) (*waiting for review*)

### Faketime job

  Introduces two opt-in jobs that inject
  [**libfaketime**](https://github.com/wolfcw/libfaketime) during builds and
  autopkgtests, helping check if packages are **future-proof** against
  the [Year 2038 problem](https://en.wikipedia.org/wiki/Year_2038_problem).

  * **Issue**: [Add optional faketime jobs to run build and autopkgtest on
  arbitrary future
  date](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/411)
  * **Merge request**: [Add faketime support for builds and autopkgtests
  ](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/599)
  (*close to being merged, I'm finalizing the last details*)

### Document how to run Salsa CI locally gitlab-ci-local

  I'm also currently looking into the possibility of running Salsa CI pipelines
  locally with gitlab-runner exec. At the moment I'm waiting for some upstream
  bugs to be fixed before moving forward. Once those bugs are fixed, I’ll
  document the steps to run the pipeline locally.

  * **Issue**: [Document howto run Salsa CI locally (in 2025 with
  gitlab-ci-local)](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/169)


### Bugfixes

| Description                                                                                                        | Issue                                                                | Merge request                                                                |
| ------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Temporarily disable reprotest until [#1108550](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1108550) is fixed | [#455](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/455) | [!619](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/619) |
| Fix use of deb-check-file-conflicts on unstable/experimental/trixie/questing                                       | [#445](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/445) | [!609](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/609) |
| Fix `apt-get` usage in wrap-and-sort job                                                                           | [#454](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/454) | [!620](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/620) |
| Add needs for arch-specific autopkgtest jobs and fix disable rules                                                 | [#476](https://salsa.debian.org/salsa-ci-team/pipeline/-/issues/476) | [!642](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/642) |


### Reviews

Besides my own contributions, I also helped review merge requests from others
in the team.

One example is the [Switch to sbuild (and
unshare)](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/569/)
work by Santiago Ruano Rincón, which has since been merged and became a key
step forward for improving our build infrastructure.

I also reviewed the following MRs:


| Title | Merge Request |
|-------|---------------|
| Disable arch-specific autopkgtest if corresponding builds are disabled | [!643](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/643) |
| ubuntu: disable the autopkgtest i386 job | [!640](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/640) |
| Support piuparts and blhc arguments containing spaces or quotes | [!636](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/636) |
| Revert "Workaround reprotest bug #1108550 by disabling some variations" | [!625](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/625) |
| missing-breaks: Use SALSA_CI_EXTRA_REPOSITORY | [!618](https://salsa.debian.org/salsa-ci-team/pipeline/-/merge_requests/618) |

## Packaging contributions

Besides Salsa CI improvements, I also contributed to Debian packaging during
the GSoC period. These uploads not only fixed issues but also gave me the
perspective of a maintainer running Salsa CI pipelines in practice.

### Package **openvr**

  Closed [bug #1067724](https://bugs.debian.org/1067724). Required a small
  transition (easy since the package has no reverse-dependencies). OpenVR is a
  virtual reality API by Valve, used by the package
  [**gamescope**](https://tracker.debian.org/pkg/gamescope), which had dropped
  the **Build-Depends** due to incompatibility with the old version. With
  this update, gamescope will include the build dependency again in its next
  release.

  Thanks to Dylan Aïssi for handling the upload, Sergio Durigan Junior for the
  review, and to Rodrigo Siqueira and Melissa Wen for suggesting this bug for me
  to tackle.

  * [Upload accepted on Debian
  tracker](https://tracker.debian.org/news/1660410/accepted-openvr-2121ds1-1exp1-source-amd64-into-experimental/)

### Package **ratt**

  Uploaded the new upstream version first to experimental, then to unstable. I
  am now co-maintaining the package together with Otto Kekäläinen, and I would
  like to thank him for reviewing and handling the uploads for me.

  * [Experimental upload Debian
  tracker](https://tracker.debian.org/news/1650881/accepted-ratt-00git20250702ac6eb62-1-source-into-experimental/)
  * [Unstable upload Debian
  tracker](https://tracker.debian.org/news/1654081/accepted-ratt-00git20250812e11cf12-1-source-into-unstable/)

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

When I first contributed, Salsa CI felt like just a big YAML file. Now I see it
as an essential part of Debian's workflow, and I had the chance to leave my
fingerprints on it, making it stronger for contributors, users, and the many
distributions based on Debian.

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

GSoC 2025 may be ending, but my work on Salsa CI and Debian packaging is just
getting started.

I plan to:

* Keep contributing to the Debian ecosystem.
* Create a job in Salsa CI to run autopkgtests of reverse dependencies
  extending coverage beyond build-time testing.

As Seneca reminds us:

> *"Every new beginning comes from some other beginning's end."*

## Acknowledgments

I'm really grateful to my mentor Otto Kekäläinen, whose weekly meetings,
guidance, and insights were essential throughout the project. Thanks also to
Santiago Ruano Rincón for his reviews and insights, as well as to the Salsa CI
team for their feedback. I would also like to thank my fellow GSoC participant
Aayush Raj, who was also working on Salsa CI for Debian this summer, for the
collaboration and exchange of ideas. None of this work would have been possible
without their support and collaboration.

Finally, special thanks to Debian and Google for making this program possible.

This summer was just the beginning. I'll keep hacking on Salsa CI so Debian
contributors can rely on stronger quality checks, end users get packages tested
from every angle before they reach the archive, and the many distributions
based on Debian benefit from the same improvements.
