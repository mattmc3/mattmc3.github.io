+++ title = 'Using awk to colorize go output' date = 2023-05-23T00:00:00-04:00
categories = 'Tutorial' tags = ['awk', 'go', 'shell'] +++

From [my StackOverflow answer](https://unix.stackexchange.com/a/546920/84777) on the
subject of colorizing Golang test run output:

I like piping to a simple awk script to colorize output from other shell commands. That way, you can customize with whatever colors/patterns suit you. You want to colorize an output unique to your project? Go for it.

Simply save this awk script to `./bin/colorize` in your project, `chmod u+x
./bin/colorize`, and customize to your needs:

```awk
#!/usr/bin/awk -f

# colorize - add color to go test output
# usage:
#   go test ./... | ./bin/colorize
#

BEGIN {
    RED="\033[31m"
    GREEN="\033[32m"
    CYAN="\033[36m"
    BRRED="\033[91m"
    BRGREEN="\033[92m"
    BRCYAN="\033[96m"
    NORMAL="\033[0m"
}
         { color=NORMAL }
/^ok /   { color=BRGREEN }
/^FAIL/  { color=BRRED }
/^SKIP/  { color=BRCYAN }
/PASS:/  { color=GREEN }
/FAIL:/  { color=RED }
/SKIP:/  { color=CYAN }
         { print color $0 NORMAL }

# vi: ft=awk
```

And then call your tests and pipe to colorize with:

```sh
go test ./... | ./bin/colorize
```

I find this method much easier to read and customize than using `sed`, and much more
lightweight and simple than using some external tool like `grc`.

Note: For the list of color codes, see
[here](https://en.wikipedia.org/wiki/ANSI_escape_code#3-bit_and_4-bit).
