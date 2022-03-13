# renf
Bash functions to manage numbered files and directories


## Abstract

renf is a Bash source of functions to managed numbered files and folders.
If you have files that you want to sort in numeric order, (I.e. lexicographically), then
renf is for you.

renf takes as its inspiration from BASIC's 'REN' command. The base idea is when
you want to add another file between 2 existing files and you do not want
to mv them manually. For example, say you are creating a slide deck from Markdown
source files. Files might be named with the following pattern: XXX_title.md where
XX is a zero padded number betweein 001-999.

## Basic usage

```bash
source renf.sh
ls *.md
001_file1.md 002_file.md 003_file.md 004.md
renf
ls 010_file.md 020_file.md 030_file.md 040_file.md
```

## Changing the gap behaviour of renf

By default, renf will renumber files with gaps of 10 numbers. This default
can be changed with the autof function.

```bash
# autof without any arguments reports the current setting
autof
10
# The value is stored in $AUTOF
echo $AUTOF
10
# using the previous example
auto 100
100
renf
ls
100_file.md 200_file.md 300_file.md 400_file.md
```


# renf is idempotent

Running the renf command a second or more time without first creating new files
or changing the autof value from its previous setting, will have no effect.


### Note on the use of the 'mv' command.

By default, renf will detect if the current top-level directory is under git
version control, and if it is, will use the command 'git mv instead of 'mv'
This behaviour can be overridden 

## The listf function

Of course, you can always use a normal 'ls' command in any directory. However,
listf takes its behaviour from BASIC's list command with similar results and options.

```bash
# Again, from above
listf
100_file.md
200_file.md
300_file.md
400_file.md
# Given some options:
listf 100-200
100_file.md
200_file.md



## The newf function

The newf function creates a newly numbered file (or directory)
It expects some template file to use as the basis for creating the next file.
It remembers which file it last created and knows the current $AUTOF setting.
Its argument should vbe thetitle or filename part of the pathname.
If there is no template set (the default setting), newf will simply 'touch'
the new file.

```bash
# newf will assume you want you add a new file to end of the current list.
newf my_new_file
500_my_new_file.md
ls *.md
100_file.md 200_file.md 300_file.md 400_file.md 500_my_new_file.md
# Given a numeric  first argument, will use that as the number part
newf 110 file_110
110_file_110.md
```

### Inspecting which next file number will be created

The LASTF variable is set whenever any of renf or friendas functions are run
and is the number of the highest numbered file. It can be inspected with the lastf function.

```bash
lastf
050
echo $LASTF
50
# Create a new interleaved file after renumbering by 100, leabbing AUTOF set to 10
renf ,100
newf 210 history
210_history.md
lastf
210
newf history_2
220_history_2.md
lastf
220
```

Note: the lastf command formatted $LASTF with the $FMTF format

## renf arguments

Like its BASIC forebears, renf takes optional arguments.

- Range: XXX-YYY
- Gap value: XX

### The range

You might want to only renumber a selected range of file names.

```bash
# Letsrenumber only files between 020 and 040
ls *.md
010_file.md 020_file.md 030_file.md  035_file.md 040_file.md 045_file.md 080_file.md
renf 20-499
ls *.md
010_file.md 020_file.md 030_file.md 040_file.md 060_file.md 070_file.md 080_file.md
```


#### Note regarding conflicts

If a renf operation would result in a conflicting matching file name with
an existing file name, renf will abort with the following message.

```bash
newf title_22
022_title_22.md
renf 20-40
Error: Cannot renumber file 030_file.md to 040_file.md because the latter already exists
```

You can always use the --dry-run option to renf to see what would be done and
check for any conflicts. Also, the -c, --check  will check if any conflicts
exist and return a non-zero return code if any do exist, and report them.
If you give the '-q, --quiet option, the error message will be suppressed.

### The gap argument

The gap argument is always preceded with a ',' and will not set the $AUTOF
environment setting

```bash
renf 10-30, 1
ls *.md
010_file.md 011_file.md 012_file.md 040_file.md 050_file.md
# Without a range:
renf ,100
ls *.md
100_file.md 200_file.md 300_file.md 400_file.md 500_file.md
# $AUTOF remains unchanged
echo $AUTOF
10
autof
10
```


## File patterns

How does renf and newf know how to construct new files or know which files to rename?
How does listf know what to list?
This is specified by the $PATF regexp. This regexp can be queried and set
with the patf function.

```bash
# What is the current PATF regular expression?
patf
[0-9][0-9][0-9]_*.md
# Set the extension
patf .txt
[0-9][0-9][0-9]_*.txt
# Change the file part of the pattern
patf 'slide_*'
[0-9][0-9][0-9]_slide_*.txt
renf
newf conclusion
list 40-50
040_slide_file.txt
050_slide_conclusion.txt
# Change the '_' delimiter to '-'
patf -
[0-9][0-9][0-9]-slide_*.txt
```

### Note on numeric formatting

The 'printf' Unix command is used to format the numeric part of the file
name. By default, it is set to "%03d" If the patf numeric is changed, the value will
be updated accordingly. You can query this value
with printing the value of $FMTF:

```bash
echo $FMTF
%03d
# Lets change the  patf numeric value
patf '[0-9][0-9]'
echo $FMTF
%02d
```
 

## Configuration



renf and friends will get their defaults by looking the following locations:

1. ./.renf : current directory
2. Any .renf file in parent folders upto, but not  exceeding the top-level .git
3. $HOME/.renf : If this exists, is used all child folders
4. $HOME/.config/renf : This is the expected default

All these config files are merely Bash language source scripts. They only
set environment variables like AUTOF, FMTF and PATF, .etc


