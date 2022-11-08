---
layout: post
title: "M2 cleaner"
categories: [github, maven]
---

Once upon a time in a project ... I ran out of disk space.

And it happened cyclically. Very large project, lots of dependent modules. All this was quickly filling the disk space with new versions and snapshots of artifacts in the Maven repository.

Solution: script to automatically remove unnecessary artifacts in the .m2 folder.

Here is [my Github repository with the M2 Cleaner](https://github.com/wgawel/m2_cleaner).

How it's working?

0. Configure the script (project name and optional list of unnecessary artifacts).
1. Run the script *m2_cleaner.sh*.
2. Enter the path to .m2 (or confirm the default).
3. Enter the version of the project from which the artifacts should not be removed (in the format "x.x.x.x", eg "3.9.9.0" or shorter "3.9.9").
4. Enter information whether you want to remove all artifacts from the additional list defined in point 0.
5. Done! You have disk space again.

A small thing, but useful, and therefore became popular in the project I was working on.

Note: Script written to run on Cmder in Windows (or other bash command line).

[![Maven M2 Cleanser screenshot](/assets/image/screenshots/m2_cleaner.png)](/assets/image/screenshots/m2_cleaner.png){:.glightbox}