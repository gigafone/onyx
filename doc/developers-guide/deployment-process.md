## Deployment Process

We use a moderately sophisticated deployment process to make sure that any changes we make to Onyx core are healthy for the entire ecosystem. We outline how it works in this chapter.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [The Problem](#the-problem)
- [Voom](#voom)
- [Unison](#unison)
  - [CircleCI Bug Fix](#circleci-bug-fix)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

### The Problem

Onyx core is a reasonably large Clojure project - several thousand lines at the time of writing this. With a solid test suite and a continuous integration server, it's easy enough to make sure that any patches we apply to Onyx keep the system healthy. The larger problem that we deal with is Onyx's vast ecosystem. Onyx core was designed to be relatively tiny. As such, dozens of smaller projects have risen up that depend on Onyx. The problem then becomes - after we make a change to Onyx core, how can be find out if we broke any dependent projects?

### Voom

In order to make sure that all dependent projects can see the latest version of Onyx, we employ a tool known as [lein-voom](https://github.com/LonoCloud/lein-voom). Voom lets you specify a Git repository as a dependency, and uses SHA hashes for version numbers to locally build specific versions of dependencies. This avoids having to release a binary artifact for every Git commit in the dependency. In effect, Voom lets us watch "the edge" of Onyx's commit history.

### Unison

A second tool which we use is called [lein-unison](https://github.com/LonoCloud/lein-unison). Unison lets you specify projects which depend on you in your `project.clj` file. When `lein unison` is invoked, Unison knows how to check out each of the dependent projects, update their versions, and push them back out to their origin. Unison is invoked every time a patch is applied to Onyx core's `master` branch. In our set up, every dependent project has a `compatibility` branch who's Onyx core dependency points to the latest commit. After Unison pushes out these changes, our CI server triggers.

The result of the infrastructure being applied is that every time we push to Onyx core `master`, all dependent projects run these tests on CI with the latest Onyx dependency, including the patch that was applied. This lets us detect downstream breakage very fast.

#### CircleCI Bug Fix

We use CircleCI to run our continuous integration tests each time we push to a repository in the onyx-platform organization. CircleCI currently has a bug regarding multiple SSH keys. In our Unison configuration, we specify specific host names for each repository as a workaround, seen [here](https://github.com/onyx-platform/onyx/blob/4fd89b756ff61522c315647632e8359e0bee9100/project.clj#L55). These hostnames are resolved in a Gist, seen [here](https://github.com/onyx-platform/onyx/blob/4fd89b756ff61522c315647632e8359e0bee9100/circle.yml#L16). We keep them in a Gist and bring them onto the CI machine using the `circle.yml` configurable file via Curl because the amount of text is fairly large and difficult to express in Yaml such that it complies with CircleCI's standards. To add a new project to Unison for automatic updates after changes to core, modify the Gist and subsequently the URL in the `circle.yml` file. A public/private key pair needs to be generated and added to both GitHub and CircleCI. Ensure that the hostnames you used are not copy and pasted!
