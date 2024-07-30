+++
title = 'Zsh - splitting a string into an array'
date = 2020-09-17T00:00:00-04:00
categories = 'Guide'
tags = ['Zsh', 'Shell']
+++

# Zsh - splitting a string into an array

## Intro

There are a ton of different ways to split up a string in Zsh. This post attempts to
show them with examples to help you build your own. I write Zsh scripts all the time,
and still reference back to this guide, so there you go.

## Using the split parameter expansion `${(s/x/)foo}`

From the Zsh docs on [Parameter Expansion
Flags](https://zsh.sourceforge.io/Doc/Release/Expansion.html#Parameter-Expansion-Flags)
(yeah - I know... how would anyone ever find that if they didn't know where to look!?)

> j:string:  Join the words of arrays together using string as a separator.
>
> s:string: Force field splitting at the separator string.

You can also read more by running `man zshexpn`. (Again... I know, right!? How would
anyone know to look there!?)

Example splitting a string on slash character in Zsh:

```zsh
$ str=part1/part2/part3
$ parts=(${(@s:/:)str})
$ echo $parts
part1 part2 part3
$ echo ${#parts[@]}
3
```

## Using split with bracketed indexing `${foo[start,end]}`

You can use split `${(@s:/:)str}` and indexing `[start,end]` to do more sophisticated
surgery on string parts. Note that Zsh array indexing starts a 1, not 0.

If you need to reassemble, simply join back on the same separator `${(@j:/:)str}`. Note:
weirdly, the colons can be swapped out for other symbols, so if you prefer periods for
example, this would also work: `${(@j./.)str}`. Since I'm splitting on slashes, I choose
not to use slash as my symbol.

```zsh
$ url="https://github.com/sorin-ionescu/prezto/blob/master/modules/history/init.zsh"
$ repo="${(@j:/:)${(@s:/:)url}[4,5]}"
$ echo $repo
sorin-ionescu/prezto
```

[An example from the Zsh
docs](https://zsh.sourceforge.io/Doc/Release/Expansion.html#Examples) which shows
splitting. Note the use of slash below unlike the colon above to surround the split
character - remember that symbol is swappable and doesn't change the behavior of the
split at all:

```zsh
$ foo=(ax1 bx1)
$ print -l -- ${(s/x/)foo}
a
1 b
1
```

Getting the first 2 parts:

```zsh
$ str=a/b/c/d/e/f
$ parts=(${(@s:/:)str})
$ echo ${(@j:/:)parts[1,2]}
a/b
```

## Pattern removal with `#` and `%`

You can also use `#` and `%` parameter expansion symbols: [see
docs](http://zsh.sourceforge.net/Doc/Release/Expansion.html#Parameter-Expansion). `#`
removes from the left side, and `%` from the right, which I remember by the fact that
`#` is to the left of `%` on your actual keyboard. `##` and `%%` use the longest match,
while `#` and `%` use the shortest.

```zsh
$ str=part1/part2/part3
$ echo ${str%%/*}
part1
$ echo ${str%/*}
part1/part2
$ echo ${${str%/*}#*/}
part2
$ echo ${str#*/}
part2/part3
$ echo ${str##*/}
part3
```

## Word splitting

Word splitting is done for every command you type to split out arguments. You can use
those same Zsh word splitting rules for your own splits if you are splitting on
whitespace. This is done with the '=' character:

```zsh
$ sentence="ls -l -A -h"
$ arr=(${=sentence})
$ print -l -- $arr
ls
-l
-A
-h
```

## Conclusion

Hope this is a helpful reference. There's [also a gist that I keep up-to-date as well
here](https://gist.github.com/mattmc3/76ad634f362b5a9a54f1779a4737d5ae).
