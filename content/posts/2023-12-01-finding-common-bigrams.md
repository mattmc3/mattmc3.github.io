+++
title = 'Finding common bigrams'
date = 2023-12-01T00:00:00-04:00
categories = 'tutorial'
tags = ['shell', 'typing']
+++

# Find common bigrams

Bigrams are 2-letter combos. When designing a keyboard layout, it's common to optimize
for comfort and speed by analyzing bigrams. Here's a simple shell script to do a
quick-and-dirty bigram analysis.

## Start with a corpus

Download Shai's corpus for Colemak:

```sh
cd ~/Downloads
curl -fsSLo corpus.txt.xz https://colemak.com/pub/corpus/iweb-corpus-samples-cleaned.txt.xz
```

Extract the .txt file:

```sh
unxz corpus.txt.xz
```

## Split into individual words

Separate that corpus into individual words, one per lined, and all lowecase letters:

```sh
tr '[:upper:]' '[:lower:]' < corpus.txt | tr '[:space:]' '\n' > corpus_word_list.txt
```

## Count bigram frequency

Run this awk script to analyze the word list and generate bigrams:

```sh
awk '{
  for (i = 1; i < length($0); i++) {
    pair = substr($0, i, 2)
    pairs[pair]++
  }
}
END {
  for (pair in pairs) {
    printf "%s: %d\n", pair, pairs[pair]
  }
}' corpus_word_list.txt | sort -k2,2nr > results.txt
```

And now, show the top ten most used bigrams:

```sh
$ head -n 10 results.txt
th: 10712957
he: 8729312
in: 8166065
an: 6560562
er: 6377359
re: 6028056
on: 5209590
at: 4541673
or: 4383020
nd: 4375566
```
