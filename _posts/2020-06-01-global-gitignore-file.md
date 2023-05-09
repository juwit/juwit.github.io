---
title: Configure a Global Gitignore 🙈
created: '2020-04-16'
modified: '2022-06-17'
language: en
tags: 
  - DevOps
  - Git
---

This short articles shows how to setup a global `.gitignore` file, to exclude files or directories for all your git repositories.

This is very useful for editor files or `.env` file, and prevents accidental commits.
I also added common directories for Java and NodeJS related developments (`target/` and `node_modules`), and IntelliJ IDEA files (`*.iml` and `.idea/`)

Thus said, you should also always setup a `.gitignore` file in your projets, as the global file only work for you, and will not be shared with the code of your project.

## create the .gitignore file

On Linux systems, the default location for a global `.gitignore` file is `~/.config/git/ignore`.

Create this file if it doesn't exist, and put your content in it: 

```shell
# create the directory if it doesn't exists
$ mkdir -p ~/.config/git

# create the global ignore file
$ cat <<EXCL >> ~/.config/git/ignore
# global gitignore file

# idea settings
.idea/
*.iml

# java
target/

# direnv
.envrc
.env
EXCL
```

And you are done!