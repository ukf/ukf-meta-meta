# ukf-meta-meta

Tools to generate and maintain the
[public `ukf-meta` repository](https://github.com/ukf/ukf-meta) as
a partial clone of the UK federation's private metadata repository.

**Note: you need to have access to the private version of the
UK federation metadata repository to use these tools.**

## Operation

These tools maintain a connection between the UK federation's
[Subversion](http://subversion.apache.org) metadata repository, which is
private because it contains personal information, and a
[public version](https://github.com/ukf/ukf-meta) of that repository hosted
on [Github](https://github.com/).

The connection is maintained through a local Git repository which uses
`git svn` to read the private Subversion repository, and `git filter-branch`
to sanitise the data for public release.

### Initialization

To create and configure the local Git `ukf-meta` repository:

    $ ./initialize

The configuration includes:

* Setting the `authors` file to map Subversion committer IDs to the
appropriate values for Git. If a Subversion committer ID is not
recognised, the `authors` file needs to be updated.

* Setting a regular expression controlling files to be ignored by
`git svn` operations such as `fetch` and `rebase`. This removes
all the personal information from the repository, and excludes
frequently-changed files such as the signed metadata in order to
reduce the overall size.

### First Fetch and Filter

Now, perform an initial import from the private repository:

    $ ./first_fetch

This operation is timed, and logged into `fetch.log`. It will take an
hour or so, and will result in a local repository of around 140MB at the
time of writing.

You will see from the log that by far the majority of the commits are empty,
because the changes they represented have been filtered out of the content trees by
the regular expression configured into the local repository.

The `first_fetch` script also generates a `files.log` listing all the
files created within the repository in order. You should review this to
make sure that nothing undesirable has made it through.

At this point, we have a single branch called `master`, and it is checked
out as `HEAD`. To set up our initial filtered `public` branch:

    $ ./first_filter

The filtered `public` branch omits the `git svn` metadata from the commit messages, and
omits all empty commits (those which didn't affect the filtered content).

After `first_filter` has run, which will take five to ten minutes, you should have `master`
and `public` branches which have different histories but represent the same content trees.
Verify this as follows:

    $ ./compare
    Commit comments should be the same, hashes different:
      master 57b4d94 Delete embedded.pem after using it.
    * public fd71f3b Delete embedded.pem after using it.

    Tree values should be the same:
       master:  tree 6867e1000b02b14305ba604e141939c3beb72215
       public:  tree 6867e1000b02b14305ba604e141939c3beb72215

    Number of revisions should be different:
       master:  11744
       public:  1081

If you get results roughly like the above (obviously the specific values
will be different) then your branches have been set up correctly.

### First Publication

You now have a working repository which is suitable for publication. It's time to add
an appropriate git remote and push to a new tracking branch. The standard setup is as follows:

    $ git remote add github git@github.com:ukf/ukf-meta.git
    $ git push --verbose --set-upstream github public:master

If the `git push` is rejected, it may be because the remote repository already contains
a `master` branch which is unrelated to the new one we're trying to create from our
`public` branch. This might happen if you regenerated the working repository after changing
the exclusion regular expression. In that case, you can reset things by adding the `--force` option
to `git push`. Be aware that if you do this in other circumstances (for example, if you've
pointed the remote URL at the wrong place), the remote repository
will lose data.

### Subsequent Updates

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
