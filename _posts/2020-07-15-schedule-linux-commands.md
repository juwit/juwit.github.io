---
title: Schedule a linux command to run later with `at`
created: '2020-07-15'
modified: '2022-06-17'
language: en
canonical_url: https://codeka.io/2020/07/15/schedule-linux-commands/
tags:
  - DevOps
---

As I prepare and run a lot of scripts, sometimes I need to run a script at a precise time of the day.

When a script must be only run once, `cron` is not a viable solution.
So I discovered the `at` scheduler

You need to install it first, using `apt` as usual.

```shell
$ sudo apt install at
```

## Schedule a command to run

1. use the command `at` with a time / date
2. input the commands to run in the prompt
3. type CTRL+D to exit (^D)

```shell
$ at 9AM       
warning: commands will be executed using /bin/sh
at> cd workspaces/github/dotfiles
at> git pull
at> <EOT>
job 1 at Sat Apr 16 09:00:00 2022
```

This example will pull a repository contents at 9 AM tomorrow !

`at` supports a lot of time specifications.

Here is an extract of its man page:

> At  allows  fairly  complex time specifications, extending the POSIX.2 standard.  It accepts times of the form
> HH:MM to run a job at a specific time of day.  (If that time is already past, the next day is  assumed.)   You
> may  also  specify  midnight, noon, or teatime (4pm) and you can have a time-of-day suffixed with AM or PM for
> running in the morning or the evening.

The commands are executed with the logged-in user account, using a `/bin/sh` shell.
It will use the available env-vars of the shell when the command `at` is executed, and will `cd` into the current directory before running your scruipt.

## view scheduled commands

```shell
$ atq
1	Sat Apr 16 09:00:00 2022 a jwittouck
```

## view the details of a job

```shell
$ at -c 1


cd /home/jwittouck || {
	 echo 'Execution directory inaccessible' >&2
	 exit 1
}
cd workspaces/github/dotfiles
git pull

```

## delete a job

```shell
$ atrm 1
```
