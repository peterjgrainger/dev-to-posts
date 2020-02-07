---
published: false
title: "Monorepo: How to keep your history in order"
cover_image: "https://raw.githubusercontent.com/peterjgrainger/dev-to-posts/master/blog-posts/monorepo-keeping-your-history-in-order/assets/cover.jpg"
description: "How to move lots of separate repositories into one repo while keeping everything in order"
tags: monorepo, git, GitHub
series:
canonical_url: "grainger.xyz/monorepo-keep-history"
---

Can you really rely on history to be accurate? History is written by the winners not the losers. I'm a massive fan of the [Vikings](https://en.wikipedia.org/wiki/Vikings_(2013_TV_series)). I love historical TV series but the more I looked into the actual facts about the time the more I realised much of the series was based around rumours and myths. There is a distinct lack of paper and pens in the fiords of Norway in that time period. Even if they had the equipment, I doubt they would care that much to write down every detail. So who writes the history then? The conquering nations who do care about writing things down, that's who!

![Viking](https://raw.githubusercontent.com/peterjgrainger/dev-to-posts/master/blog-posts/monorepo-keeping-your-history-in-order/assets/viking.jpg)

So like those conquers of the past we are going to re-write history--Git history :)

In this post I'll explain how I converted multiple Git repositories into one repository by gluing together two different solutions.

## My project

My current setup consisted of multiple repositories with a few characteristics that caused a few headaches in this process:

- Most of the repositories had 2000+ commits in each
- Repositories were reliant on each other and had branches with the same name that related to each other, e.g. `release-123`
- There were about 20 different repositories
- The history was important
- There were lots of branches, most stale.

## Reusing Others Hard Work

The first thing I did when approaching this task was to re-use as much from others who have attempted this before.

There are some great tools to convert the structure of your repos from multi to monorepos. I'll discuss [tomono](https://github.com/hraban/tomono) and [ShopSys](https://www.shopsys.com/how-to-merge-15-repositories-to-1-monorepo-keep-their-git-history-and-add-project-base-as-well-6e124f3a0ab3/). Tomono is simpler but shopsys has fuller features, both didn't really work for me.

## Tomono 

The tomono script is first. I noticed late on that the history wasn't actually being re-written (by design) and was relies on the built in mechanisms of git merge. I would probably agree with the maintainer that this is a [bug in git](https://github.com/hraban/tomono/issues/6) not the scripts but it doesn't help me right now.

## Shopsys

Next to the shopsys tools. These scripts *do* re-write history but they only merge the master branch. As my setup is weird and I need to merge *any* branch that has the same name. I can't use these scripts out of the box either.

## Creating a Monster

Like Frankenstein my requirements were for something that didn't exist. I've got a good start but need to take the best parts from each script and combine them to create a monster script. I'll warn you now--like Frankenstein's monster this script is not the prettiest or most efficient but it gets the job done. 

![Frankenstein (1922 book)](https://commons.wikimedia.org/wiki/Category:Frankenstein_(1922_book)#/media/File:Frankenstein,_pg_7.jpg)

## It's alive

The magic from the shopsys script comes from the script [`rewrite-history-into`](https://github.com/shopsys/monorepo-tools/blob/master/rewrite_history_into.sh) which uses `git filter-branch` to rewrite the Git history to move everything into a subfolder.

I call the shopsys script from a [modified tomono script](https://github.com/arup-group/tomono/blob/c9e7dfede85a0d010eeca02db068f326e0689a9d/tomono.sh#L163) to re-write the history for each branch. I've shown the interesting parts below:


```bash
# tomono.sh
git checkout -q $name/$branch # checkout the remote branch which will be a subfolder
../monorepo-tools/rewrite_history_into.sh $name # re-write the history
git switch -q -c temp-munge-branch # make a temp branch to store the changes
git checkout -q $branch # Go back to the base repo
git reset -q --hard # get rid of any changes
git merge -q --no-commit temp-munge-branch --allow-unrelated-histories # Do the merge!
```

## Now for the downside

This script is *painfully* slow. It takes *hours* to run. I'm sure there are optimisations I can make but as I'm only running this once i don't really care.

My script also includes some logic around which branches to move over. You might want to clean up your branches before the move but I'm lazy and just avoided transferring those branches.

## Expectation vs Reality

Current scripts are good but can't account for every scenario. If you are in the same scenario as me I hope this helps. There is a high chance you have a slightly different use case so be sure to allow enough time for your transition so you don't rush it and make a mistake!

# Found a typo?

If you've found a typo, a sentence that could be improved or anything else that should be updated on this blog post, you can access it through a git repository and make a pull request. Instead of posting a comment, please go directly to <https://github.com/peterjgrainger/master/blog-posts/monorepo-keeping-your-history-in-order/monorepo-keeping-your-history-in-order.md> and open a new pull request with your changes.
