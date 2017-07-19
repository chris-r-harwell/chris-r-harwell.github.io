---
layout: post
title: Data science tech support episode 2: making a clean pull request using a branch
tags: git pull
---

The GitHub and the git source control system can be intimidating sometimes.  A class I recently took insisted that students pull their repo, add a unique file and create a pull request to turn in assignments.  The setup was such that the they forked the primary repo on GitHub and then checked that out on laptops.  So, three versions:
1. upstream (the class copy)
2. origin (their GitHub copy, forked from above)
3. local (their laptop copy, created with git clone ...)

This is a good opportunity to get used to the work flow which many software projects using git take advantage of, but it can also be a source of frustration.

If your changes are completely independent and a single file, as they were in this case, it may actually make more sense to just create a specific branch each time you want to turn something in. I know this may sound more complex, but it minimized the chance that your pull request will be rejected because of a non-linear history or other interesting artifacts which crop up from time to time. This can be especially true if you are feeling a bit lost as to how you got into this state.


1. This assumes you've already forked and cloned. Change into the checkout directory.
```
cd ~/git/nyc17_ds12
```

2. If you do not already have an upstream _remote_ added (and these students do), add it with
```
git remote add upstream  git@github.com:thisismetis/nyc17_ds12.git
```

3. Copy the file(s) you want to add somewhere you can get to it. (Use your actual file name rather than literally "my-file-to-turn-in".)
```
cp my-file-to-turn-in ~/
```

3. _fetch_ the updates for the origin and upstream remotes from GitHub to your local clone.
```
git remote update
```

4. Create a new branch (just for this) and check it out. Here we called it clean-pull, 
but it could be anything not already listed in ```git branch -v```.

```
git checkout -b clean-pull upstream/master
```

5. Copy that file back, add and commit.
```
cp ~/my-file-to-turn-in ~/
git add my-file-to-turn-in
git commit -m 'adding that file i want to turn in' my-file-to-turn-in
```

6. Push it to origin.
```
git push origin clean-pull
```

7. Visit your GitHub page for this repo and check the top where it says,
'Your recently pushed branches:' and click on the green "Compare & pull request" button
to make the pull request.

