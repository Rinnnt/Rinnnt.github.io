---
layout: post
title: "Regular Expressions - A Short Reference Guide"
date: 2022-02-25
categories: etc
excerpt_separator: <!--more-->
---

Regular Expressions (Regex) can be difficult to learn, but also very rewarding when mastered. Regex is used to check for specific patterns in strings. This is a quick guide to get started in Regex. <!--more-->

### Reference Guide

- `abc` - A plain sequence of characters, matches for exactly "abc"
- `[abc]` - Square brackets match for any character in the set, e.g. "a" or "b" both matches
- `[0-9]` - A hyphen matches for any character within the range (ordering of unicode), e.g. "1" would match
- `[^abc]` - A caret matches for any character except those in the set, e.g. "d" would match
- `x?` - A question mark matches for zero or one occurences
- `x*` - An asterisk matches for zero or more occurences
- `x+` - A plus sign matches for one or more occurences
- `x{y}` - Curly braces matches for exactly <i>y</i> occurences
- `x{y, z}` - Matches for at least <i>y</i> and at most <i>z</i> occurences
- `(x)` - Use parentheses to group regular expressions
- `x|y` - Use vertical bars for logical OR
- `\` - Use backslash to escape special characters like + and *, or use shortcuts
    * `\b` as a boundary marker
    * `\d` for digits
    * `\w` for alphanumeric chars
    * `\s` for whitespace (space, tab, newline etc.)
    * `\D` `\W` `\S` for any char <i>except</i> digits/alphanumeric/whitespaces
- `^` - Caret outside of square brackets matches for start of string
- `$` - Dollar sign for end of the string


[Back](/)