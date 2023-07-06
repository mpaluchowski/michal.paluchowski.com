---
author: michal
date: 2017-02-01T13:47:32.000Z
title: Git hard or go home
categories:
  - Technology
tags:
  - git
  - tools
  - version control
---

I am in love with [Git][git-home]. It's the neatness junkie's dream come true. Multiple people can code away safely, independently, then every commit can be written and rewritten at any time to end up with a tidy, coherent product. As long as you know what you're doing, that is.

<!--more-->

With the years I worked out a few strategies for taming Git, making both mine and my collaborators' work easier. Perhaps these'll save you some headaches down the road.

## Understand the model

A prerequisite to understand most of Git's concepts is learning how the data model works. It's so important, that before I let developers do anything with Git, I force them to read through the ["What is Git?" chapter of Pro Git][progit-gitbasics]. The rest they can learn as they go. In essence:

* __all commits are nodes in a directed graphs__, where
    - when multiple commits point to the same parent, they're all branches,
    - when one commit has multiple parents, it's a merge,
* there's __no such thing as "branch" or "tag"__ in itself, they're just _labels_ given to specific commits---friendlier names to refer to than `63949df`, so:
    - deleting a branch or tag merely deletes the label. The commit `63949df` is still there and you can reference it. At least until [Git decides to do garbage collection][progit-maintenance], when all orphaned commits are permanently deleted.
    - a "fast forward" merge moves a label to a different commit, as long as the branch you're merging _into_ didn't have any commits of its own. Since there's no separate merge commit for it, there will be no trace of this merge later in the graph, so make sure that's what you want. Otherwise do `git merge --no-ff`.

Take your time to study the model, the charts and perhaps the commit graph in [some existing repository][github-explore]. Working with branches will suddenly make much more sense for you, and more importantly you'll grasp the concept of __rebasing__ much, much quicker.

## Rebase freely

Rebasing is the wonder broom that keeps the commit graph tidy. It's what you do to get rid of commits saying `Merge branch 'master' of https://github.com/your-name/your-repo.git` which make the [commit graph look like a Guitar Hero script][git-graph-guitar-hero]---completely unreadable.

You can:

* __amend broken commits__, if you:
    - mistyped the last commit's message do `git commit --amend` and edit the commit,
    - forgot to stage some changes, add a file, do

        ```shell
        git add <missing changes>
        git commit --amend -C HEAD
        ```
* __squash obsolete commits__, once you realize a commit _other_ than your last one could use amending, do:

    ```shell
    git add <missing changes>
    git commit --fixup=<hash of commit to amend>
    git rebase -i --autosquash <hash of commit to amend>^
    ```

    Note the [trailing caret `^`][git-ancestry-references] in the last line.

* __rebase instead of merge on pull__, pretty please. This makes sure `master` (or any other branch) is a single, straight line, that's easy to follow. This will also cause fewer conflicts and make the remaining ones easier to resolve. Use either `git fetch && git rebase origin/<your branch>` or `git pull --rebase`.

There's a __caveat__ to rebasing---it __changes history__, discarding some commits and replacing them with other ones. That's not an issue, as long as all of the commits you're replacing are only local to your sandbox, but once you _pushed_ these commits out into the world, you might be [peeing into the pool][git-pretty].

## Branch smartly

Just because branching and merging with Git is easier than with older version control systems, that [doesn't make branching any less dangerous][codinghorror-branching]:

> Start juggling too many [branches] at once, and you're bound to drop a few. In most source control systems, you can create hundreds of branches with no performance issues whatsoever; it's the mental overhead of keeping track of all those branches that you really need to worry about. Your developer's brains can't exactly be upgraded the same way your source control server can (...)

To stay on top of your repository:

* __keep branches short and small__, branch off, make your changes, create a pull request and merge as soon as possible. I used to advocate the [gitflow model][gitflow] in the past, but nowadays I consider its long-lived `develop` and `release` branches an _anti_-pattern, and favor the simpler [GitHub Flow][github-flow]. Fewer branches mean fewer (ideally _just one_) sources of truth.
* __delete remote branches__, once merged, usually right after a pull request is closed. If you ever need to resurrect a branch and add something to it, you can always push it anew. Otherwise it's hard to see which branches are still active.
* __don't `git cherry-pick`__ or commit the same patch of changes to multiple branches. It's confusing and will likely give you headaches during merging. If you stick to a single, long-lived `master` branch with short-lived feature branches, you shouldn't ever need cherry-picking. Otherwise, use rebasing to _move_ a commit from one branch to another.

## Push thoughtfully

Once your changes are out there, on the remote(s), you might as well assume everybody else has already fetched them. GUI clients like [SourceTree][sourcetree] have auto-fetching enabled by default. If you _now_ realize you'd still like to do some amendments---say fix a silly typo or rebase the `master` branch---you're _screwed_. You'll have to ask everybody to `git reset` their local branches and cause a lot of grief.

__Take your time__ before you push anything, and never, ever mark the "Push changes immediately to remote" checkbox in your Git client. It's like _disabling_ the Undo feature. Commit your changes _locally_, leave them baking for an hour or a day, then _review_ them---see if there are no missing or obsolete line changes, files, whether commit messages are correct---and only then run `git push`.

---

Reduced amount and magnitude of merge conflicts, actually useful commit graphs, immediately obvious location of the most current code---all benefits I drew from applying these here practices. Call it Git etiquette or call it a way to improve the ratio of pleasure to pain in using Git. And do share your own tips in the comments, please.

[codinghorror-branching]: https://blog.codinghorror.com/software-branching-and-parallel-universes/
[git-ancestry-references]: https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection
[git-graph-guitar-hero]: https://twitter.com/HenryHoffman/status/694184106440200192
[git-home]: https://git-scm.com/
[git-pretty]: http://justinhileman.info/article/changing-history/
[gitflow]: http://nvie.com/posts/a-successful-git-branching-model/
[github-explore]: https://github.com/explore
[github-flow]: https://guides.github.com/introduction/flow/
[progit-gitbasics]: https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F
[progit-maintenance]: https://git-scm.com/book/en/v2/Git-Internals-Maintenance-and-Data-Recovery
[sourcetree]: https://www.sourcetreeapp.com/

