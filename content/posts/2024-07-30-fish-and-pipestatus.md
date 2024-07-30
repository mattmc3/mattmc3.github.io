+++
title = 'Fish and pipestatus'
date = 2024-07-19T17:00:00-04:00
categories = 'tutorial'
draft = true
tags = ['shell', 'Fish']
+++

# Fish and `$pipestatus`

_Note: This morning I answered [a Reddit
post](https://www.reddit.com/r/fishshell/comments/1efpruv/about_checking_pipestatus/)
about `$pipestatus`, and after learning that there's not a lot of good information out
there, I thought making a blog post might be helpful for other future Fish users wanting
to know more about using Fish's `$pipestatus`._

## What's an exit status?

In Fish, like other shells, you can tell whether a command failed via its return value,
sometimes called an exit code or an exit status. An exit status is a number between 0
and 255 (well, really 0-254, with 255 signaling any out-of-range exit code above 254).
Canonically, an exit status of 0 means success, while any non-zero value indicates an
error.

_Note: Don't make the mistake of thinking 0 means FALSE and 1 means TRUE - that's not
how exit statuses work, and is in fact the exact opposite of what they mean in this
case._

According to the [Linux Documentation
Project](https://tldp.org/LDP/abs/html/exitcodes.html), certain exit codes are reserved
to have certain meanings:

| Exit Code        | Meaning                        |
| ---------------- | ------------------------------ |
| 0                | Success                        |
| 1                | Catchall for general errors    |
| 2                | Misuse of shell built-ins      |
| 126              | Command invoked cannot execute |
| 127              | Command not found              |
| 128              | Invalid argument to exit       |
| 128+n            | Fatal error signal "n"         |
| 130              | Script terminated by Control-C |
| 255              | Exit status out of range       |

In Bash or Zsh, the special variable `$?` holds the exit status. In Fish, you can see
the exit status from a command if you examine the contents of the special `$status`
variable.

Now, `$status`, like `$?` in POSIX shells, is very volatile. Every Fish command you run
changes the value of `$status`. It is common to see scripts where the exit status is
stored in a variable so that it does't get blown away by subsequent commands. So, in
Fish, that might look something like this:

```fish
somecmd --arg1 foo bar
set --local last_status $status # store off the status
if test $last_status -ne 0
    # Write to stderr and return the status
    echo >&2 "Your command errored with status: $last_status..."
    return $last_status
end
```

If you tried to use `$status` here without saving it in a variable, every new command
you run in the script would have modified its value with their own exit status. So in
this case, `somecmd`, `test` and `echo` are commands, and each will change `$status`.

Do to its volatility, you need to use or store `$status` immediately after a command
completes, otherwise it will change when the next command runs and you will lose its
prior value.

## Piping commands

When shell scripting, it is common to pipe commands. What we mean by piping commands is
running a command and piping its output as the input to another command, and so on.
Let's take a quick example in Fish where we print the URL to this blog post and pipe
that to `string match` to pull out the slug name:

```fish
> echo "https://mattmc3.github.io/posts/2024/07/fish-and-pipestatus/" |
>   string match -rg '/([^/]+)/$'
fish-and-pipestatus
```

Here we just piped 2 commands, but you could pipe many more:

```fish
# How many blog posts did I write this year?
ls | grep '2024' | wc -l | string trim
```

Remember how we just said `$status` is volatile? Well, now we have 4 commands we just
ran together, and each would change the value of `$status` as it went along, leaving
`$status` to only reflect the result of running `string trim`.

Enter `$pipestatus`.

## Getting the exit status from piped commands

Bash and Zsh have a way of handling command piping - setting the `$PIPESTATUS` or
`$pipestatus` variables respectively. These variables are arrays which contain the exit
status of each individual command. Let's demonstrate how that works in Bash:

```bash
$ true | false | true | true  # Fake 4 piped commands
$ echo ${PIPESTATUS[@]}  # Show the exit status of those 4 cmds
0 1 0 0
```

For many years, Fish did not have an equivalent. But in 2019, [ticket
#2039](https://github.com/fish-shell/fish-shell/issues/2039) was closed and Fish gained
its own `$pipestatus` variable.

Now, let's demonstrate that same Bash script in Fish:

```fish
> true | false | true | true
> echo $pipestatus
0 1 0 0
```

Behold! The magic of `$pipestatus`!

## Using `$pipestatus`

Now that we know about `$pipestatus`, it might be tempting to test for errors like so:

```fish
# Oops... this looks like it should work, but it won't!
true | false
if test $pipestatus[1] -ne 0 || test $pipestatus[2] -ne 0
    echo >&2 "An error was found: $pipestatus"  # <-- Unreachable code!
end
```

This fails spectacularly! What happened? Remember how we talked about how volatile
`$status` is? Well, `$pipestatus` is the same - it changes after every new command. In
the above script, when the first `test` was  called, it reset `$status` and
`$pipestatus` to reflect its own exit status, and then when `test $pipestatus[2]` tries
to run, `$pipestatus` no longer has 2 elements.

So what's the fix? Store the results of `$pipestatus` **IMMEDIATELY** in a variable, and
then use that variable to do your error handling.

```fish
# This works since we save the volatile $pipestatus contents
true | false
set --local last_pipestatus $pipestatus  # Save ourselves pain!
if test $last_pipestatus[1] -ne 0 || test $last_pipestatus[2] -ne 0
    echo >&2 "An error was found: $last_pipestatus"
end
```

Remember - `$status` and `$pipestatus` are weird Schrödinger's variables. They change
every time you peek in the box. Every. Single. Command.

That's not unique to Fish. Bash has the same behavior:

```bash
$ true | false | true | true  # simulate piping in Bash
$ echo ${PIPESTATUS[@]}  # echo is about to change $PIPESTATUS
0 1 0 0
$ echo ${PIPESTATUS[@]}  # AND... it did
0
```

## Better ways to test $pipestatus

Let's say you have 6 piped commands, do you really have to test each result like this?

```fish
# Don't do this
true | false | true | true | false | true
set --local last_pipestatus $pipestatus
if test $last_pipestatus[1] -ne 0
      or test $last_pipestatus[2] -ne 0
      or test $last_pipestatus[3] -ne 0
      or test $last_pipestatus[4] -ne 0
      or test $last_pipestatus[5] -ne 0
      or test $last_pipestatus[6] -ne 0

    echo >&2 "error"
end
```

The answer is no. This gets really messy to read and offers no benefit. You could
choose to treat your array like a string:

```fish
# Better, but don't do this either
if test "$last_pipestatus" != "0 0 0 0 0 0"
    echo >&2 "error"
end
```

This is certainly more readable, but what if you miscounted your pipes and got the
number wrong? This can lead to subtle bugs. Instead, my preferred test for `$pipestatus`
is `string match -qr '[^0]' $last_pipestatus`. What this does is quietly (`-q`) tests
every element in our saved `$last_pipestatus` array with a regex (`-r`) pattern looking
for any non-zero value (`[^0]`).

Here's the full example:

```fish
# This test works no matter how many things we have piped
true | false | true | true
set --local last_pipestatus $pipestatus
if string match -qr '[^0]' $last_pipestatus
    echo >&2 "Errors were seen: $last_pipestatus"
end
```

## Conclusion

Hopefully this was a helpful introduction to handling exit statuses in Fish.
