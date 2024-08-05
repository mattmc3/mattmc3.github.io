+++
title = 'Mac Terminal.app vs iTerm2'
date = 2024-08-05T08:02:00-04:00
categories = 'Tech'
tags = ["Terminal"]
+++

Every once in awhile I wonder if I'm using an unnecessary 3rd party app when what
comes built-in on macOS would be sufficient. Terminal.app is one such app - I've used
[iTerm2] for many years, but I don't use a bunch of its features and every time I open
its settings it feels like overkill.

I was on my spouse's Mac over the weekend helping her get some things set up, used
Terminal.app and was pretty happy with it, so I decided to try it on my Mac. Nope. It
still only supports 256 colors, which I had forgotten. This post is here simply to
remind me next time I get the bug to use this macOS built-in - just don't.

To replicate this test, run this:

```zsh
curl -fsSLo ~/bin/color-spaces https://raw.githubusercontent.com/robertknight/konsole/master/tests/color-spaces.pl
chmod u+x ~/bin/color-spaces
color-spaces
```

## Terminal.app

![Terminal.app colors](/assets/img/terminal-dot-app-colors.png)

## iTerm2

![iTerm2 colors](/assets/img/iterm2-colors.png)


[iTerm2]: https://iterm2.com/
