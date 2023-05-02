# Bash Parameter Expansion: A Short Guide

Bash parameter expansion is a powerful feature that allows you to manipulate strings in shell scripts. It can be used to remove or replace substrings, handle error cases, and more.

Here's a quick overview of the most commonly used parameter expansion syntaxes:

## Basic Syntax

The basic syntax of parameter expansion is `${var}`. This expands to the value of the variable `var`. If `var` is unset or null, it expands to an empty string. You can also use `${var:-default}` to expand to the value of `var`, or `default` if `var` is unset or null.

Example:

``` bash
name="John Doe"
echo "Hello, ${name:-there}" # Output: "Hello, John Doe"
unset name
echo "Hello, ${name:-there}" # Output: "Hello, there"
```


## Removing Substrings

You can remove substrings from the beginning or end of a variable using `${var#pattern}`, `${var##pattern}`, `${var%pattern}`, or `${var%%pattern}`. The `#` and `%` characters remove the shortest match, while the `##` and `%%` characters remove the longest match.

Example:
``` bash
path="/path/to/some/file.txt"
echo ${path#/} # Output: "path/to/some/file.txt"
echo ${path##/} # Output: "file.txt"
echo ${path%.txt} # Output: "/path/to/some/file"
echo ${path%%/*} # Output: ""
```

## Replacing Substrings

You can replace substrings in a variable using `${var/pattern/replacement}` or `${var//pattern/replacement}`. The first syntax replaces the first match of `pattern` with `replacement`, while the second syntax replaces all matches.

Example:

``` bash
greeting="Hello, world!"
echo ${greeting/world/universe} # Output: "Hello, universe!"
echo ${greeting//o/0} # Output: "Hell0, w0rld!"
echo ${greeting/#Hello/Hi} # Output: "Hi, world!"
echo ${greeting/%!/... goodbye!} # Output: "Hello, world... goodbye!"
```


## Error Handling

You can use `${var:?error message}` to exit the script with an error message if `var` is unset or null.

Example:

``` bash
name=
echo "Hello, ${name:?Please provide a name}"

```


This will output an error message and exit the script if `name` is unset or null.

## Substring Length

You can use `${#var}` to get the length of a variable's value.

Example:

``` bash
word="hello"
echo ${#word} # Output: 5
```


## Default Value

You can use `${var:-default}` to provide a default value if `var` is unset or null. If `var` is set, `${var:-default}` expands to `var`.

Example:

``` bash
echo ${missing:-default} # Output: "default"
name="John Doe"
echo ${name:-default} # Output: "John Doe"
```


## Final Thoughts

Bash parameter expansion is a powerful tool that can help you write more efficient and concise shell scripts. By mastering the basic syntax and some of the most common patterns, you'll be able to manipulate strings with ease and handle error cases more gracefully.
