# ukf-meta-meta

Tools to generate and maintain the
[public `ukf-meta` repository](https://github.com/ukf/ukf-meta) as
a partial clone of the UK federation's private metadata repository.

**Note: you need to have access to the private version of the
UK federation metadata repository to use these tools.**

## Operation

These tools maintain a connection between the UK federation's
metadata tooling repository, which is
private because it contains historical personal information, and a
[public version](https://github.com/ukf/ukf-meta) of that repository hosted
on [Github](https://github.com/).

### Working Repository

The connection between the UK federation and public versions of the repository
is maintained through a local Git repository within which
`git filter-branch` is used to sanitise the data for public release.

The working repository is a clone of the UK federation's
version of the repository, and its `origin` remote points there. The `github`
remote is used to refer to the public version of the repository on GitHub.

The following branches are used:

* `master` tracks the `master` branch in `origin`
* `public` tracks the `master` branch in `github`
* `update` acts as the bridge between the two

### Initialization

To create and configure the local Git `ukf-meta` repository:

    $ ./initialize

The configuration includes:

* Setting the `push.default` configuration variable to `default` to
simplify the update operation.

### Acquire Public Copy

This document assumes that you're not creating the public version of the
repository for the first time.

You should pull in the public repository's `master` branch as well
as follows:

    $ ./pull_public

This will set up the `public` branch to track the `github/master` branch.

If you look at `gitk --all --date-order` you should see that only the first
couple of commits are the same before the branches diverge as the `master`
branch includes files like `xml/sdss-sites-11-unsigned.xml` which are filtered
out of the `public` history.

### Update Process

To incorporate new material from the private repository and merge it into the public
repository, execute:

    $ ./update

This script performs the following steps:

* Switch back to the `master` branch.
* Use `git pull` to fetch the new material.
* Create a new `update` branch, and filter it.
* Merge that in to the `public` branch using a fast-forward only merge.

If this process fails, one good way of debugging it is to say:

    $ (cd ukf-meta; gitk --all --date-order)

This can help you locate things like the `update` and `public` branches
diverging because of a change to the files being filtered out. Another trick
for similar issues is to explicitly look for the place where they diverge:

    $ (cd ukf-meta; git merge-base public update)

Because the update process involves re-filtering every commit,
`./update` can take up to twenty minutes.

You should use the `compare` script to verify that everything has worked
before pushing the results to the public repository:

    $ ./compare
    Commit comments should be the same, hashes different:
      master fcaa044 Add a .gitignore equivalent to existing svn ignores.
    * public ba673cf [ahead 1] Add a .gitignore equivalent to existing svn ignores.

    Number of revisions should be different:
       master:  11745
       public:  1082

Note the "`[ahead 1]`" above indicating that there is something to push. Bring the
public repository up to date as follows:

    $ (cd ukf-meta; git push)

If you run `compare` again, the "`[ahead 1]`" should be gone. Success!

### Save and Restore

Two scripts are provided to help with development of the other scripts. They maintain
a saved copy of the `ukf-meta` repository as `ukf-meta-save`.

* `save` removes any saved repository and copies `ukf-meta` to `ukf-meta-save`.

* `restore` replaces `ukf-meta` with `ukf-meta-save`, if the latter exists.

## Copyright and License

The entire package is Copyright (C) 2013, University of Edinburgh.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
