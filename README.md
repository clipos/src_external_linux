CLIP OS Kernel
==============

This repository holds the source code for the [CLIP OS](https://clip-os.org) kernel.

Overview
--------

The CLIP OS kernel is based on Linux. It also integrates:
- existing hardening patches that are not upstream yet and that we consider
  relevant to our security model;
- developments made for previous CLIP versions that we have not upstreamed yet,
  or that cannot be upstreamed;
- entirely new functionalities that have not been upstreamed yet.

Documentation
-------------

The documentation is available in the `products/clip/doc` folder.

Git Workflow
------------

The `master` branch is the main CLIP OS kernel branch. It is the **repo**
branch and it should only contain merges of other branches, which can be:
- `upstream/<name>` branches: those are regularly resynchronized with branches
  from corresponding upstream projects and thus only contain upstream work.
  They must be declared as **repo** metadata in `manifest/sources.xml` so that
  people can retrieve all of them using the **pull-upstream** recipe. Pushes to
  such branches obviously bypass Gerrit code reviews.
- `feature/<name>` branches: those branches are either forked from `upstream/`
  branches or from `master`, and contain added work. They are merged into
  `master`. Any work being merged into `feature/` branches must have previously
  been validated.
  If such a branch contains work that should ultimately be upstreamed, it is
  rebased onto every new major kernel version to create a new `feature/` branch
  suffixed with this version, that will also be merged into `master`. If, on
  the contrary, a `feature/` branch contains work that is not intended for
  upstream, the new major kernel version (typically, tag v4.x) is merged into
  it, and the branch is then merged into `master`.

The `master` branch is a rolling release branch and must always be in a clean
state. Stable branches may be forked from it at some point and supported for
some time.

To keep all our history in the `master` branch even when updating to a new
major Linux kernel version (e.g., going from 4.17 to 4.18), we do the
following:
```bash
# For feature branches that are rebased onto the new major kernel version
$ git checkout -b feature/<branch>-4.18 feature/<branch>-4.17
$ git rebase v4.18
$ git merge --strategy=ours --edit feature/<branch>-4.17

# For feature branches that are not rebased
$ git checkout feature/<branch>
$ git merge --edit v4.18

# Merge (octopus) all feature branches into a temporary branch
$ git checkout -b master-tmp v4.18
$ git merge --edit feature/<branch1> feature/<branch2> ...

# "Import" this merge into the master branch
$ git checkout master
$ git merge --strategy=ours --no-commit feature/<branch1> feature/<branch2> ...
$ git read-tree -um master-tmp
$ git commit

# Delete temporary branch
$ git branch -D master-tmp
```

This results in a `master` branch updated to the 4.18 kernel and that can be forward-pushed.

Integration in CLIP OS
----------------------

The kernel is incorporated into CLIP OS thanks to the **clipos-kernel** ebuilds
located in `src/portage/clipos/sys-kernel/clipos-kernel`. The **cros-workon**
eclass from Chromium OS is used to handle fetching sources from a Git
repository. Different kernel versions can be managed by the use of symbolic
links; see the **clipos-portage** tool located in `src/platform/clipos-portage` for
more information on that.

The kernel is then built as part of the **efiboot** recipe (see the toolkit
documentation for more information) and can be tested using the **qemu**
recipe to fire a new CLIP OS virtual machine.

How to Contribute
-----------------

You can contribute to this project in various ways, including by:
- doing code reviews on [Gerrit](FIXME);
- picking [GitHub](FIXME) issues and working on them;
- testing and giving feedback, suggesting new ideas and, more generally,
  joining our discussions on [Gitter](FIXME), [Discourse](FIXME), etc.
