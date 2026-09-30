---
layout: post
title: Linux Command Line
---

The command line is a text-based way to interact with your computer. You type commands into a terminal, and a shell reads and runs them. It can feel unfamiliar at first, but a small set of commands makes it possible to navigate files, inspect information, and automate repetitive work.

This guide introduces the commands you will use most often. Examples do not include the `$` prompt; type only the command that follows it.

## Understanding paths

Linux organizes files in a tree of directories. The top of that tree is the root directory, written as `/`. Your personal files are usually in your home directory, represented by `~`.

A path can be absolute, starting from `/`, or relative to your current directory. These shortcuts are useful in either kind of path:

- `.` means the current directory.
- `..` means the parent directory.
- `~` means your home directory.

Use `pwd` (print working directory) to see where you are:

```bash
pwd
```

Use `ls` to list the contents of a directory. Add options to show more information or include hidden files (whose names begin with a dot):

```bash
ls
ls -l
ls -a
ls -la
```

To move to another directory, use `cd` followed by its path:

```bash
cd Documents
cd ..
cd ~/Downloads
cd /
cd
```

Running `cd` without a path returns you to your home directory. If a path contains spaces, put it in quotes, for example `cd "Project Files"`.

## Working with files and directories

These commands create, copy, move, and remove files and directories:

```bash
mkdir practice
cd practice
touch notes.txt
cp notes.txt notes-backup.txt
mv notes-backup.txt archive.txt
```

`mkdir` creates a directory, and `touch` creates an empty file if it does not already exist. `cp` copies a file; `mv` moves or renames it. For example, `mv old-name.txt new-name.txt` renames a file in the current directory.

To remove a file, use `rm`. Check the path before running it: removed files may not be recoverable from the terminal.

```bash
rm archive.txt
```

To remove an empty directory, use `rmdir directory-name`. Removing a directory and its contents with `rm -r` is more consequential, so use it only after confirming the target path. Avoid running removal commands with elevated privileges unless you understand exactly what they will delete.

## Reading and finding information

`cat` prints a file's contents. For longer files, `less` lets you scroll through the text; press `q` to quit.

```bash
cat notes.txt
less /etc/os-release
```

Use `head` or `tail` to view the beginning or end of a file. The `-n` option sets how many lines to show:

```bash
head -n 5 notes.txt
tail -n 10 notes.txt
```

`grep` searches text for a word or pattern. This example searches a file without matching letter case:

```bash
grep -i "linux" notes.txt
```

Use `find` to locate files by name. This searches your home directory for files named `notes.txt`:

```bash
find ~ -name "notes.txt"
```

## Commands, options, and help

Many commands accept options that change how they work. Options commonly begin with a hyphen, such as `ls -l`. A command's manual page explains its options and behavior:

```bash
man ls
```

Press `q` to leave a manual page. For a brief summary, try `command --help`, for example `cp --help`. Not every command supports the same help option.

## Combining commands

The pipe character (`|`) sends the output of one command to another. This example lists the current directory and lets `less` display the results one screen at a time:

```bash
ls -la | less
```

You can redirect output into a file with `>`:

```bash
ls -la > directory-list.txt
```

This creates the file or replaces its existing contents. Use `>>` to append output instead:

```bash
echo "Practice complete" >> notes.txt
```

## Permissions and administrator access

The `ls -l` command shows who can read, write, or execute a file. Permissions are divided among the file's owner, its group, and other users. The `chmod` command changes permissions; for example, this adds execute permission for the file's owner:

```bash
chmod u+x script.sh
```

Some system tasks require administrator privileges. `sudo` runs a command with those privileges, usually after asking for your password. Use it only when needed and only with commands you understand. It does not make a command safer; it gives that command more power.

## Practice

Try these commands in your home directory. They create a small practice folder and a file without changing system files:

```bash
mkdir ~/cli-practice
cd ~/cli-practice
touch first.txt
echo "Hello from the command line" > first.txt
cat first.txt
cp first.txt second.txt
ls -l
cd ..
```

The best way to become comfortable is to run a command, check its output, and use `man` or `--help` when you are unsure. Start with paths and listing files; the rest builds naturally from there.