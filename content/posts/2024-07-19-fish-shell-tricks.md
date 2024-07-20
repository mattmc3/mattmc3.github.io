+++
title = 'Fish Shell Tricks'
date = 2024-07-19T09:00:00-04:00
categories = 'tutorial'
draft = true
tags = ['shell', 'Fish']
+++

From [https://itnext.io/the-zsh-shell-tricks-i-wish-id-known-earlier-ae99e91c53c2](https://itnext.io/the-zsh-shell-tricks-i-wish-id-known-earlier-ae99e91c53c2)

## Introduction

Navigating the command line efficiently can significantly enhance your productivity,
especially when using powerful shells like Zsh. Whether you’re a seasoned developer or a
curious newbie, mastering Zsh’s features can transform your workflow. In this guide,
I’ll show you some essential Zsh tips and tricks, from cursor navigation to custom
commands, that will streamline your terminal experience. Those are in no particular
order, however I tried to group them into logical units. Each chapter shows different
practical applications.

## Why Zsh?

Zsh, or Z shell, is a Unix shell that can be used as an interactive login shell and as a
command interpreter for shell scripting. Zsh is known for its rich feature set, which
includes powerful command-line editing, built-in spell checking, and programmable
command completion. It’s the default shell for macOS and is available on various
GNU-Linux operating systems. It will also work on Windows with Windows Subsystem for
Linux.

## Who is This Blog For?

- **Developers:** If you’re writing code, automating tasks, or managing servers, knowing
  Zsh can help you work more efficiently.
- **System Administrators:** Zsh’s scripting capabilities and robust feature set can
  make managing systems and automating tasks easier.
- **DevOps Engineers:** For those involved in continuous integration and deployment,
  mastering Zsh can streamline operations and enhance productivity.
- **Students and Beginners:** If you’re new to the command line, learning Zsh can
  provide a solid foundation and make your learning curve smoother.
- **Tech Enthusiasts:** If you love exploring new tools and optimizing your workflow,
  this guide will introduce you to some cool features of Zsh.

By the end of this blog, you’ll be equipped with practical tips and tricks to boost your
command-line efficiency using Zsh.

## Cursor Navigation and Line Editing

Efficient cursor movement and line editing are crucial for command-line productivity.
Zsh provides powerful shortcuts to enhance these operations, making it easier to
navigate and edit commands quickly.

### Cursor Movement

Moving the cursor efficiently can save you a lot of time, especially when dealing with
long commands.

- `Ctrl + A` - Move the cursor to the beginning of the line.
- `Ctrl + E` - Move the cursor to the end of the line.

These shortcuts allow you to quickly jump to the start or end of the command line, which
is particularly useful when you need to edit the beginning or the end of a command.

### Text Deletion

Deleting text efficiently helps you correct mistakes and modify commands without
retyping them entirely.

- `Ctrl + U` - Delete from the cursor to the start of the line.
- `Ctrl + K` - Cut from the cursor to the end of the line.
- `Ctrl + W` - Cut from the cursor to the start of the preceding word.
- `Alt + D` - Remove from the cursor to the end of the next word.

These shortcuts enable you to quickly remove large chunks of text, whether it’s clearing
the entire line or just deleting the last word you typed.

### Text Manipulation

Manipulating text directly from the command line can enhance your efficiency, allowing
you to make quick changes and corrections.

- `Ctrl + Y` - Paste (yank) previously cut text.
- `Ctrl + Shift + _` - Undo the last keystroke in the command.
- `Ctrl + XT` - Swap the current word with the previous word.
- `Ctrl + Q` - Move the current command to the buffer, clear the line for a new command.

Using these shortcuts, you can easily move text around, undo mistakes, and restore
previously deleted text, making your command-line experience much smoother.

### Command Editing

Sometimes, you need more advanced editing capabilities to fine-tune your commands.

- `Ctrl + XE` - Edit the current command in the default editor.

## Globbing

Zsh provides advanced globbing features for flexible file matching, which is a powerful
way to handle files and directories. Globbing allows you to specify patterns that match
multiple filenames, enabling you to work with groups of files, directories and more very
efficiently.

### Standard Globbing

Standard globbing uses wildcards to match filenames based on simple patterns.

- `*.txt` - Match all .txt files in the current directory.
- `file?.txt` - Match files like file1.txt, file2.txt, etc., in the current directory.

These patterns help you quickly select and operate on groups of files without typing
each filename individually.

### Recursive Globbing

Recursive globbing extends the standard patterns to search through directories and
subdirectories.

- `**/*.txt` - Match all `.txt` files recursively within directories.

This powerful feature allows you to search for files across an entire directory tree,
making it easy to find and manipulate files regardless of their location.

### Extended Globbing

Zsh’s extended globbing features provide even more advanced pattern matching
capabilities.

Enable extended globbing with the following command:

```zsh
setopt EXTENDED_GLOB
```

Once enabled, you can use additional pattern matching features:

- `ls *(.)` - List only regular files.
- `ls *(/)` - List only directories.
- `ls *(-/)` - List only directories and symbolic links to directories.

Extended globbing lets you refine your file searches with greater precision, making it
easier to filter and list files based on specific criteria.

### Examples of Globbing in Action

Here are some practical examples of using globbing to manage files:

- `mv *.txt backup/` - Move all .txt files to the backup directory.
- `rm **/*.log` - Remove all .log files in the current directory and all subdirectories.
- `cp *.{jpg,png} images/` - Copy all .jpg and .png files to the images directory.

By leveraging the power of globbing, you can streamline your file management tasks,
making it easier to work with large sets of files and directories.
