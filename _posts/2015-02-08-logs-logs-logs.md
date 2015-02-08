---
layout:     post
title:      "Logs, logs, logs!"
---

As services become more and more complex, log analysis has become
increasingly imperative when trying to diagnose bugs. Services now
generate megabytes or even gigabytes of log files on an average
day, and it's more often than not impossible to look through data manually
from each file, or even open them in certain text editors (\*ahem\*
**Notepad**). So how do we work with these files?

# Unix Command-Line Utilities
Unix command line utilities are known for (1) their composability and
(2) doing one thing and doing it well. Below are a few of the most
useful tools for text processing.

## cat
**Manpage**: <http://linux.die.net/man/1/cat>

cat is one of the simplest Unix utilities. It con**cat**enates the
contents of the filename parameters and prints the result to stdout.
Alternatively, if there are no parameters, it echos all input until EOF
to stdout. cat is most often used to put the contents of a file into a
pipe to be read by another program, but can also be handy for forcing
weird programs like `man` to output to stdout in a jiffy.

### Examples
Printing from stdin.

    $ echo "Hello world!" | cat
    Hello world!

Count the lines in test.txt.

    $ cat test.txt | wc -l
    3

## grep
**Manpage**: <http://linux.die.net/man/1/grep>

grep is the bread and butter of text search. At the basic level, grep
searches file parameters or stdin line-by-line for a specified regex and
lists the lines that match.

Most programmers are more familar with Perl Compatible Regular Expressions
(PCRE) than POSIX regex, grep's default. This mode can be enabled by using
the `-P` parameter.

Other commonly used switches include `-o`, which prints only the text that
matches your regex; -c`, which prints the number of matches; and `-v`,
which inverts your search (only prints lines that don't match).

### Examples
List matches of regex.

    $ echo "The cat in the hat" | grep -o at
    at
    at

Invert search.

    $ echo "Hello world" > file.txt
    $ echo "Foo bar" >> file.txt
    $ grep -v "Hello" file.txt
    Foo bar

## cut
**Manpage**: <http://linux.die.net/man/1/cut>

cut is a great tool for working with delimited data. The default
delimiter is tab, but can be easily changed with the `-d` flag. The `-f`
flag then specifies which delimited field(s) to print. The
[manpage](http://linux.die.net/man/1/cut) details how to select a range
of fields:

> Each range is one of:
>
> N 	N'th byte, character or field, counted from 1
> N-	from N'th byte, character or field, to end of line
> N-M	from N'th to M'th (included) byte, character or field
> -M	from first to M'th (included) byte, character or field

Likewise, using `-f -` selects all fields.

### Examples
Print the first 3 fields of a CSV line.

    $ echo "eggs,bananas,oranges,pears" | cut -d',' -f1-3
    eggs,bananas,pears

Print from character 22 to the end.

    $ echo "eggs,bananas,oranges,pears" | cut -c22- 
    pears

Convert a CSV line to TSV.

**Note**: You can insert tab into the commandline with &lt;CTRL&gt; v +
&lt;TAB&gt;

    $ echo "eggs,bananas,oranges,pears" | cut -d',' -f - --output-delimiter='   '
    eggs    bananas oranges pears

## find
**Manpage**: <http://linux.die.net/man/1/find>

While not directly involved in text manipulation, `find` is a handy
utility for searching for files based on specific criterion.
`find` is commonly used with the `-exec` flag, which allows you to run
commands (like those discussed in this section) on files matching the
search parameters.

### Examples
Search files in current directory for filenames beginning with string
"bacon".

    $ find . -name "bacon*"

List contents of all directories in the current directory.

    $ find . -type d -exec ls {} \;

## sort
**Manpage**: <http://linux.die.net/man/1/sort>

True to its name, `sort` sorts lines from stdin or a file. It's important
to note that, by default, `sort` sorts character by character from the
beginning on the string. This can be problematic when sorting numbers.
Luckily, `sort` comes with the `-n` flag which sorts numbers.

### Examples
Sorting without -n flag.

    $ echo -e "123\n34\n678" | sort
    123
    34
    678

Sorting with the `-n` flag.

    $ echo -e "123\n34\n678" | sort -n
    34
    123
    678

## uniq
**Manpage**: <http://linux.die.net/man/1/uniq>

`uniq` is a useful utility for dealing with potentially duplicated lines
in data. By default, `uniq` de-dupes contiguous duplicate lines in the
input. When using the `-d` flag, uniq does the opposite; it prints only
lines that have occurred multiple times in a row.

It's especially important to keep in mind that uniq de-dupes
**contiguous duplicate lines**.  Practically, this means that data is
usually piped through `sort` before `uniq`.

### Example
De-duping a list of fruits.

    $ echo -e "apples\noranges\npears\napples\noranges\nbananas" | sort | uniq
    apples
    bananas
    oranges
    pears

Printing only strings that are duplicated.

    $ echo -e "apples\noranges\npears\napples\noranges\nbananas" | sort | uniq -d
    apples
    oranges

# Scripting Languages
For more complex work, scripting languages shine. Popular scripting
languages for text processing include awk, ruby, perl, and python. While
scripting languages are powerful, don't forget that you can compose
one-liners with command-line tools to avoid reinventing the wheel, and
to make your life much, much easier.

## Awk
Great awk tutorial: <http://ferd.ca/awk-in-20-minutes.html>

## Ruby
Ruby is my scripting language of choice when it comes to text
manipulation, simply because it is most familiar to me. Here's how to
leverage ruby one-liners to do your bidding.

### One-liners
Like perl, ruby has an `-e` switch which executes the specified argument
as ruby code.

    $ ruby -e 'puts "Hello world!"'
    Hello world!

The `-n` switch compliments that flag by wrapping the ruby one-liner in
a `while gets ... end`. This means the one-liner is run on each line of
input from stdin and stored in the variable \$\_.

For a file `file.txt` containing:

    fruit,price,amount
    apples,1.50,15
    bananas,0.50,2
    oranges,1.00,7
    pears,2.50,5
    pineapples,3.00,4

Say we didn't know the above command-line tools and we wanted to list
the prices of fruit in file.txt. We would do:

    $ cat file.txt | ruby -ne 'puts $_.split(",")[1]'
    price
    1.50
    0.50
    1.00
    2.50
    3.00

