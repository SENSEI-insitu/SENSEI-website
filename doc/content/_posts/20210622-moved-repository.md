---
title: SENSEI-insitu moved to GitHub
date: 2021-06-22 15:35
markdown:
  gfm: false
---

# For Developers

With the move to GitHub we need your help to move relevent issues from the archived GitLab instance to the new GitHub instance. Provided is a checklist of things to do to make sure this is all done.

- Make an account on [GitHub][]
- (optional) Star the new [SENSEI-insitu/SENSEI][sensei-repo] repository
- (optional) Make a development fork 
- [Add a new remote](#Adding-a-remote) for the [GitHub repository][sensei-repo] (or your fork)
- [Push branches](#Push-branches) that are currently being developed to github
- Re-open active issues and MRs from GitLab in the new repository's [issues][] and [pull requests][]
- [Update](#Update origin) `origin` to point at the new repository


### Adding a remote

```bash
$ git remote add github https://github.com/SENSEI-insitu/SENSEI.git
```

or if you have a fork

```bash
$ git remote add github https://github.com/my-github-id/SENSEI.git
```

### Push branches

You will have to run this for each development branch

```bash
$ git push github my-feature-branch
```

### Update origin

```bash
$ git remote set-url origin https://github.com/SENSEI-insitu/SENSEI.git
```

and optionally, if you have a fork

```bash
$ git remote set-url --push origin https://github.com/my-github-id/SENSEI.git
```

If there are any questions about the transitions to GitHub, please contact [Ryan Krattiger][]


[GitHub]: https://github.com/
[sensei-repo]: https://github.com/SENSEI-insitu/SENSEI
[issues]: https://github.com/SENSEI-insitu/SENSEI/issues
[pull requests]: https://github.com/SENSEI-insitu/SENSEI/pulls
[Ryan Krattiger]: mailto:ryan.krattiger@kitware.com
