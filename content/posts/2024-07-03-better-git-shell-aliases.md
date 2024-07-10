+++
title = 'Better git shell aliases'
date = 2024-07-03T09:00:00-04:00
categories = 'tutorial'
tags = 'git'
+++

# Better git shell aliases: using an external shell script

10 years ago, I read [this blog post][github-flow-aliases] on GitHub
Flow git aliases by Phil Haack. From it, I learned a few really clever
tricks. Even though I never much cared for using 'GitHub Flow' as a
git workflow, I used some of those tricks for my own git aliases. One
of those being this basic pattern:

```properties
[alias]
  foo = "!f() { echo \"foobar: $@\"; }; f"
```

This lovely little mess of an alias embeds a one-line shell function
tersely named "`f`" directly into a git command. [From the git
manual](https://git-scm.com/docs/git-config): "if the alias is
prefixed with an exclamation point, it will be treated as a shell
command". [Other sources around the same time][advanced-git-aliases]
also began promoting that same trick. It lets you handle all the other
arguments passed in, so that typing `git foo bar baz` will print the
string "foobar: bar baz".

That trick opens the door to let you create all sorts of clever
aliases like [`git browse`][git-alias-open-url], which navigates to
the remote URL of your repo in a web browser. The 'browse' git alias
Phil published in that link is very Windows/Git Bash specific, and it
doesn't work for repos cloned with the `git@github.com:my/repo` form,
so I had to modify it in my config to become:

```properties
[alias]
  browse = "!f() { REPO_URL=$(git config remote.origin.url | sed -e 's|^.*@|https://|' -e 's|.git$||' -e 's|:|/|2'); git web--browse $REPO_URL; }; f"
```

And now I have this lovely little mess of an alias. Notice that long
horizontal scroll? Yeah... that one-liner is about 150 characters long
and pretty gnarly. But it works, so I kept it and moved on. For 10
years I popped useful aliases in and out of my gitconfig, many with
this inline shell pattern.

Recently, I stumbled upon an [Oh-My-Zsh][omz] plugin called
[git-commit][git-commit] which takes this pattern to a new extreme. It
stores the function in a string, and then generates 12 little alias
monsters like this:

```properties
[alias]
  build = "!a() { if [ \"$1\" = \"-s\" ] || [ \"$1\" = \"--scope\" ]; then local scope=\"$2\"; shift 2; git commit -m \"build(${scope}): ${@}\"; else git commit -m \"build: ${@}\"; fi }; a"
```

It's really tricky (if not nearly impossible) to read and understand
that. Especially with all the backslash escaping. Now, the authors of
`git-commit` aren't intending for that alias to actually be maintained
in its `gitconfig` form - that result is generated from a string in a
script. But even so, forget about changing that easily, customizing it
to your needs, or testing it without copy/pasting it into something
usable. They even added some extra magic to update the aliases in your
`gitconfig` whenever OMZ updates! Imagine the chaos if that had a bug!

I started to wonder why they didn't just include an actual script file
(let's call it `omz_git_commit`), and then have the aliases simply
call out to that shell script. If they did that, then most of the
craziness goes away, like so:

```properties
[alias]
  build = "!omz_git_commit build"
  chore = "!omz_git_commit chore"
  ci = "!omz_git_commit ci"
  docs = "!omz_git_commit docs"
  feat = "!omz_git_commit feat"
  fix = "!omz_git_commit fix"
  perf = "!omz_git_commit perf"
  refactor = "!omz_git_commit refactor"
  rev = "!omz_git_commit rev"
  style = "!omz_git_commit style"
  test = "!omz_git_commit test"
  wip = "!omz_git_commit wip"
```

They also wouldn't need to risk modifying people's gitconfigs every
time Oh-My-Zsh has a new commit. As I started to search more, I
realized that there's a whole lot of examples of the `foo = "!f() {
<YOLO!>; }; f"` pattern of git aliases, but not a lot of great
examples of writing a separate shell script.

## The antipattern

By now you can probably see where this is going - these kinds of
complex inline git aliases are an antipattern. My `gitconfig` was a
mess of shell scripts I could not easily read or understand. Add to
that the problem of mixing code and configuration in one file. For the
occasional one-off, sure. Fine. Do whatever. But once I got past a
certain number of these aliases at a certain complexity, it was time
to refactor.

> _Code should live in a place where it's easy to evaluate, execute,
> and evolve, not in some config strings where you can't do any of
> those things_.
>
> \- me, shower thoughts

## My solution

There are a couple different solutions here. First, instead of git
aliases, you could simply switch to regular shell aliases (or
functions). This is a perfectly acceptable option, and there are
[already discussions][reddit_aliases_git_or_bash] you can find on that
topic.

However, my solution was to improve my existing git aliases by simply
creating an external shell script and changing my gitconfig aliases to
call it. That way, I retain my muscle memory, but move the existing
code to someplace more appropriate.

That script needs to be in the shell's `$PATH` for git to find it.
With this script I can do more complicated actions without feeling
like I have to cram everything into one line. And, it's easy to see
what's in that script, understand what is being done, and evolve and
test all the code. This script file can be written as a POSIX script,
Bash, Zsh, Fish, Dash, Oil, Nushell, Xonsh - whatever you want. And,
you don't have to convert your git aliases wholesale - you can start
just with the ones that reach a certain complexity. Though I do find
it easier to have most of my git subcommand handlers in one place.

### Preparing to use your script

For demo purposes, we'll create a simple POSIX script for our "git
extensions" called `gitex`. We'll put it in `~/bin/gitex` (though you
could use `~/.local/bin` or anyplace else you prefer). We also need to
make it executable:

```sh
mkdir -p ~/bin && touch ~/bin/gitex
chmod u+x ~/bin/gitex
```

Make sure you have added `~/bin` to your `$PATH`. If you don't know
how to do that, consult your preferred shell's documentation.

#### Bash

For Bash, you might need to do something like this:

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
```

#### Zsh

For Zsh, you could add this to your `${ZDOTDIR:-$HOME}/.zshrc`:

```zsh
path=(~/bin $path)
```

#### Fish

For Fish, you could add this to your `config.fish`:

```fish
fish_add_path --global --prepend ~/bin
```

### Setting up your script

Now that we have our new `~/bin/gitex` script set up, let's make a
simple primary function so we can extend our script with subcommands
as we add new git aliases.

```sh
#!/bin/sh
##? gitex - git extensions; make git shell aliases that don't suck

# POSIX test for whether or not a function exists.
is_function() {
  [ "$#" -eq 1 ] || return 1
  type "$1" | sed "s/$1//" | grep -qwi function
}

# This main function serves to call your custom subcommand
# functions (eg: 'gitex_foo') in this same gitex file.
gitex() {
  local subcmd
  if is_function "$1"; then
    subcmd="$1"
    shift
    "gitex_${subcmd}" "$@"
  else
    echo >&2 "gitex: subcommand not found '$1'."
    return 1
  fi
}
gitex "$@"
```

### Extending your script

You can now add new subcommand functions to your `~/bin/gitex` script.
Let's add our `git browse` command.

```sh
##? browse: Open web browser to git remote URL
gitex_browse() {
  local url
  url="$(
    git config "remote.${1:-origin}.url" |
      sed -e 's|^.*@|https://|' -e 's|.git$||' -e 's|:|/|2'
  )"
  git web--browse "$url"
}
```

And now finally, you can add your new `gitex` aliases to your
`gitconfig`:

```properties
[alias]
  browse = "!gitex browse"
```

Hope this was helpful! For people who've been shell scripting for
awhile, this is probably old hat. But for folks who have only just
begun to dive into this space and have copy/pasted others' scripts
over time, hopefully this is a helpful next step in your shell
scripting journey.

### A special note for alternative shell users

If you are a [Fish shell][fish] user, it can be especially daunting
(or at the least, annoying) to deal in POSIX shell syntax. If you want
to write your `gitex` script in Fish (or another scripting language),
you can easily do that by changing the shebang from `#!/bin/sh` to
`#!/usr/bin/env fish` and then writing your script in that language.

For Fish users, there's another alternative. You can, in fact, write
plain old Fish functions and assign them to git aliases! While POSIX
is cross-platform and has a lot of advantages, if you use Fish you
probably know all those pros/cons and can weigh whether any of that
really matters to you - that's a whole different article. Let's assume
that you, for whatever reason, prefer to write your scripts in a
"sensible language" because you want to ["never write esac
again"][fish].

It's as simple as defining Fish functions (ex: `gitex_foo`,
`gitex_bar`, etc), and then add this to your `gitconfig`:

```properties
[alias]
  foo = "!fish -P -c 'gitex_foo $argv'"
  bar = "!fish -P -c 'gitex_bar $argv'"
  baz = "!fish -P -c 'gitex_baz $argv'"
```

## In conclusion

You can [check out my dotfiles][dotfiles] if you want to see [my
`gitex` implementation][gitex]. It has some helpful extras like
supporting kebab-case-aliases, as well as my favorite alias `git
cloner`, which enhances `git clone` with some extras:

- It lets you clone using repo short names (ohmyzsh/ohmyzsh)
- It assumes you want to clone a default location (`~/repos` which is
  configurable), instead of `$PWD`, unless you provide a destination
  directory arg (eg: `.`)
- It can add flags you might forget, but usually want like
  `--recurse-submodules`

Example:

```sh
$ git cloner --depth 1 sorin-ionescu/prezto
clone command modified to:
  git clone --recurse-submodules --depth 1 git@github.com:sorin-ionescu/prezto.git ~/repos/sorin-ionescu/prezto
```

Happy scripting!

[advanced-git-aliases]: https://www.atlassian.com/blog/git/advanced-git-aliases
[dotfiles]: https://github.com/mattmc3/dotfiles/
[fish]: https://fishshell.com/
[git-alias-open-url]: https://haacked.com/archive/2017/01/04/git-alias-open-url
[git-commit]: https://github.com/ohmyzsh/ohmyzsh/blob/master/plugins/git-commit/git-commit.plugin.zsh
[gitex]: https://github.com/mattmc3/dotfiles/blob/main/bin/gitex
[github-flow-aliases]: https://haacked.com/archive/2014/07/28/github-flow-aliases/
[omz]: https://github.com/ohmyzsh/ohmyzsh
[reddit_aliases_git_or_bash]: https://www.reddit.com/r/git/comments/9drimq/aliases_git_or_bash/
