---
title: Status update, my contributions to open source in March 2025
date: 2025-04-02 10:30:00
categories:
- open-source
---

For a long time, I've wanted to make writing blog posts a regular thing about
what I'm doing in the open-source world. Recently, I got inspired by Rodrigo
Siqueira and Emmanuel Arias. I remember Siqueira doing something
[similar](https://siqueira.tech/) a while back, and I found the idea of a
status update really interesting. I also saw some of Emmanuel's LinkedIn posts
where he shares what he's been working on [each month](https://eamanu.com/),
and it hit me, this is actually a great way to track progress and stay
motivated, and keep grinding.

# What I worked on in March

### OpenVPN

I worked on security fixes and quality assurance for OpenVPN. I fixed
*CVE-2022-0547* and *CVE-2024-5594* in Bullseye, which is currently Debian's
oldstable release, improving security for users. Security support for Bullseye
is now handled by the Debian LTS team, with Freexian coordinating the efforts.

**CVE-2022-0547**: Possible authentication bypass in external plug-ins under
specific conditions.

**CVE-2024-5594**: Improper sanitization of PUSH_REPLY messages, allowing
potential data injection.

For more details, see:
1. [Debian LTS Announcement](https://lists.debian.org/debian-lts-announce/2025/03/msg00005.html)
2. [OpenVPN Update](https://tracker.debian.org/news/1625617/accepted-openvpn-251-3deb11u1-source-into-oldstable-security/)

A big thanks to Sylvain Beucler (@beuc) from the Debian LTS-Team for handling
the upload for me.

### cURL

When fixing security vulnerabilities (CVEs), projects often add tests to ensure
the issue is resolved and everything still works as expected. Curl is not
different, it includes these validation tests as well.

Back in the days, when I was fixing *CVE-2024-9681* for curl in the Debian
bookworm release, Samuel Henrique (@samueloph), one of the maintainers of Curl
in Debian, noticed that the tests introduced to validate the fix [weren't
executing as
expected](https://salsa.debian.org/debian/curl/-/merge_requests/37#note_547058).
This happened because those tests required curl to be built in a specific way
known as "event-based" which only works when Curl is compiled with Debug
enabled [[1]](#ref1).

To address this, I created autopkgtests that build Curl with Debug enabled,
allowing those tests to run as intended. Since we wanted to save resources and
avoid running the entire test suite again just to validate these specific
cases, and Curl's test script (runtests.pl) lacked an option to filter them, I
submitted a patch to the curl upstream to add this functionality. It was
accepted, and that actually marked my first contribution to Curl upstream! :)

Check it out: [[2]](#ref2)

But why is ensuring test execution in older versions is so crucial? When
backporting a fix, it's not just about applying a patch, the person working on
it must ensure that everything still works as expected and that no regressions
occur. This was particularly relevant to Debian's security process, where
backporting CVE fixes required verifying that introduced tests ran properly (if
there were any). Ensuring test execution in bookworm was a key step in
validating the backport.

So, this problem turned into a real rabbit hole, where every solution just led
to more issues (*which was good, tbh*). Overall, it was a nice learning
experience, and I enjoyed the challenge of improving the testing workflow. 

These autopkgtests were added in [[3]](#ref3), making them a useful addition
for CVE validation.

A huge thanks to Samuel Henrique (@samueloph) for handling the upload,
providing feedback, and reviewing everything along the way!

# Reflections

This month was all about balancing different contributions while staying
consistent. Writing about it helped me realize how much actually got done, it's
easy to miss that when you're in the middle of things. Even small wins add up.

That's it for now, dear reader. Thanks for reading, and see you in the next one! o/

# References

<a id="ref1"></a>
[1] "curl Event-based". URL: [https://curl.se/dev/tests-overview.html#event-based](https://curl.se/dev/tests-overview.html#event-based)

<a id="ref2"></a>
[2] "First curl upstream patch". URL: [https://github.com/curl/curl/commit/912efa2d1c33531561f044af72e4451e08682883](https://github.com/curl/curl/commit/912efa2d1c33531561f044af72e4451e08682883) 

<a id="ref3"></a>
[3] "Curl 8.13.0~rc2-2 in Debian". URL: [https://tracker.debian.org/news/1630510/accepted-curl-8130rc2-2-source-into-unstable/](https://tracker.debian.org/news/1630510/accepted-curl-8130rc2-2-source-into-unstable/)
