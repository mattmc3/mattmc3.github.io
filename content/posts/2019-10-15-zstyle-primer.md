+++
title = 'zstyle primer'
date = 2019-10-15T00:00:00-04:00
categories = 'Guide'
tags = ['Zsh', 'Shell']
+++

From [my StackOverflow answer](https://unix.stackexchange.com/a/546920/84777) on the
subject of Zstyles:

When it comes to learning about the Zsh `zstyle`, it's hard to find a lot of good
examples that describe what zstyles are used for and how to use them. The Zsh
documentation notoriously obtuse. To learn about `zstyle`, I recommend spending some
time looking at how [Prezto][1] uses it, as well as [reading the docs][2] and trying
some things in an interactive Zsh session.

`zstyle` is mainly used for completions, but is actually really good for other purposes,
like storing data in a way that's more sophisticated than plain-old-environment
variables.

[This gist][3] shows how you might use zstyle to store and retrieve information:

```zsh
# reference: http://zsh.sourceforge.net/Doc/Release/Zsh-Modules.html#The-zsh_002fzutil-Module

# list all zstyle settings
zstyle -L

# store value in zstyle
zstyle :example:favorites fruit apple

# store multiple values in zstyle
zstyle :example:list fruits banana mango pear

# retrieve from zstyle and assign new $fav variable with -g
zstyle -g fav ':example:favorites' fruit && echo $fav

# retrieve from zstyle and be explicit about the assignment data type:
# -a: array, -b: boolean, -s: string
zstyle -a :example:list fruits myfruitlist && echo $myfruitlist

# test that a zstyle value exists with -t
if zstyle -t ':example:favorites' 'fruit' 'apple'; then
    echo "an apple a day keeps the dr. away"
fi
if ! zstyle -t ':example:favorites:vegtable' 'broccoli'; then
    echo "Broccoli is the deadliest plant on Earth - why, it tries to warn you itself with its terrible taste"
fi

# delete a value with -d
zstyle -d ':example:favorites' 'fruit'

# list only zstyle settings for a certain pattern
zstyle -L ':example:favorites*'
```

Zstyles can be really helpful when working with boolean values. For example, we showed
how `-t` will test the value of a style, but additionally, it understands boolean words
like 'yes', 'true', 'on', or '1', so you can let a user configure the features they want
like so:

```zsh
# Turn on foo feature, and turn off bar feature... baz isn't specified
zstyle ':example:foo' enabled 'yes'
zstyle ':example:bar' enabled 'no'

# Now, test whether the user wanted those features
for feat in foo bar baz; do
  zstyle -t ":example:$feat" enabled && echo "enabling $feat" || echo "disabling $feat"
done
```

The `-t` flag assumes false if a zstyle is not set, but you can reverse that with `-T`,
which will assume true:

```zsh
# If you prefer non-specified options to default to true
# instead of false, swap the `-t` flag for `-T`, which will
# make baz enabled instead of disabled in our example
# Now, test whether the user wanted those features
for feat in foo bar baz; do
  zstyle -T ":example:$feat" enabled && echo "enabling $feat" || echo "disabling $feat"
done
```

And finally, perhaps the most powerful part of zstyles is the ability to use cascading
settings via '*' patterns. Let's imagine you have a plugin that has 3 features: foo,
bar, and baz. Those features let you specify whether you want them to create Zsh
aliases. You can specify that you want all aliases enabled except for the ones 'bar'
provides like so:

```zsh
zstyle ':example:*:aliases' enabled 'yes'
zstyle ':example:bar:aliases' enabled 'no'
```

Then, even though we didn't specify foo, `zstyle -t ':example:foo:aliases' enabled`
still works because of the `*` pattern.

[1]: https://github.com/sorin-ionescu/prezto
[2]: http://zsh.sourceforge.net/Doc/Release/Zsh-Modules.html#The-zsh_002fzutil-Module
[3]: https://gist.github.com/mattmc3/449430b6654aaab0ba7160e8efe8291b
