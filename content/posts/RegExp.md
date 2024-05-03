---
title: "regular expressions cheatsheet"
date: 2024-05-03T19:51:21+08:00
tags: ["regexp", "regular expressions"]
categories: ""
author: "Jiao shijie"
draft: false
hidemeta: true
math: false
---

# Regular Expressions

## Matching Table

| metacharacter | matching things                                                                                                     |
| :-:           | :-:                                                                                                                 |
| `\d` `\D`     | matching all the digits(0 to 9), matching any non-digit character                                                   |
| `.(dot)`      | matching any single character(letter, digit, whitespace, everything)                                                |
| `[]`          | matching a single letter inside `[]`(square brackets)                                                               |
| `[^]`         | matching any single character except for the letters inside `[]`(square brackets) with the `^`(hat)                 |
| `[-]` `[^-]`  | indicating a character range to match a single character                                                            |
| `\w` `\W`     | this is equivalent to the character range `[A-Za-z0-9_]`, matching any non-alphanumeric character                   |
| `{}`          | to match repetition of character                                                                                    |
| `*` `+`       | to match either `0 or more` or `1 or more` of the character that it follows(it always follows a character or group) |
| `?`           | this metacharacter allows to match either 0 or 1 of the preceding character or group                                |
| `\s` `\S`     | to match all whitespaces, matching any non-whitespace character                                                     |
| `␣`           | to match space                                                                                                      |
| `\t`          | to match tab                                                                                                        |
| `\n`          | to match new line                                                                                                   |
| `\r`          | to match carriage return(useful in Windows environments)                                                            |
| `^` `$`       | the start and the end of the line, **it's only used at the beginning or end of a line**                             |
| `()`          | match groups                                                                                                        |
| `(())`        | nested groups                                                                                                       |
| `(\|)`        | to denote different possible sets of characters                                                                     |
| `\b`          | to match the boundary between a word and a non-word character                                                       |

## Details

- `\d` the 123s
  - matching all the digits(0 to 9)
- `.(dot)` the dot
  - matching any single character(letter, digit, whitespace, everything). But when you want just to match a period, you need to escape the dot by using a slash `\.` accordingly.
- `[]` matching specific characters
  - matching a single letter inside square brackets
  - e.g. the pattern `[abc]` will only match a single a, b or c letter and nothing else.
- `[^]` excluding specific characters
  - matching any single character except for the letters inside `[]`(square brackets) with the `^`(hat)
  - e.g. the pattern `[^abc]` will match any single single character except for the letters a, b or c.
- `[-]` `[^-]` character ranges
  - e.g. `[0-6]` will only match any single digit character from zero to six and nothing else.
  - e.g. `[^n-p]` will only match any single character except for letters n to p.
  - **Multiple character ranges can also be used in the same set of brackets, along with individual characters.**
    - e.g. `[A-Za-z0-9_]`
- `{}` Catching some zzz's
  - e.g. `a{3}` will match the `a` character exactly three times.
  - **certain regular expression engines**(some engines may do not support) will even allow to specify a range for this repetition.
    - e.g. `a{1,3}` will match the `a` character no more than 3 times, but no less than once and `a{1,}` or `a{,3}` also can be use.
  - `w{3}`(three w's), `[wxy]{5}`(five characters, each of which can be a `w`, `x` or `y`) and `.{2,6}`(between two and six of any character)
- `*` `+`
  - e.g. `\d*` to match any number of digits(0 or more). `\d+` to ensures that the input string has at least one digit.
  - e.g. `a+`(one or more a's), `[abc]+`(one or more of any a, b or c character) and `.*`(zero or more of any character)
- `?` characters optional
  - e.g. `ab?c` will match either the strings "abc" or "ac" because the b is considered optional.
  - **To match a plain question mark character in a string, you will have to escape it using a slash `\?`**
- `\s`(all whitespaces) `␣`(space) `\t`(tab) `\n`(new line) `\r`(carriage return(in Windows))
- `^` `$` the start and the end of the line
  - e.g. ^success will match only a line that begins with the word "success"
- `()` match groups
  - e.g. ^(IMG\d+\.png)$ to capture and extract the full filename
  - e.g. ^(IMG\d+)\.png$ will only capture the part before the period
- `(())` nested group

  ```
  for example:
    Jan 1987    (Jan 1987) (1987)
    May 1969    (May 1969) (1969)
    Aug 2011    (Aug 2011) (2011)

  ^([A-Z][a-z]* (\d*))$
  ```
- `(|)` it's all conditional
  - `Buy more (milk|bread|juice)` to match only the strings `Buy more milk`, `Buy more bread` or `Buy more juice`

- `(?i)` makes the regex case insensitive
- `(?c)` makes the regex case sensitive

## Others

One concept that we will not explore in great detail in these lessons is back referencing, mostly because it varies depending on the implementation. However, many systems allow you to reference your captured groups by using `\0` (usually the full matched text), `\1` (group 1), `\2` (group 2), etc. This is useful for example when you are in a text editor and doing a search and replace using regular expressions to swap two numbers, you can search for `(\d+)-(\d+)` and replace it with `\2-\1` to put the second captured number first, and the first captured number second for example.
