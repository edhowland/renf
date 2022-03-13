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
XX is a zero padded number betweein 001-999. You can use renf to renumber with
a gap of 10 between each file name.

## Basic usage

```bash
source renf.sh
cd ./path/to/dir/w/numbered/md/files
ls *.md
001_file1.md 002_file.md 003_file.md 004.md
renf
ls*.md 
010_file.md 020_file.md 030_file.md 040_file.md
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

#### The arewegit function

```bash
mkdir foo; cd foo
arewegit
echo $?
1
git init .
arewegit
echo $?
0
mkdir bar; cd bar
echo $PWD
/home/<you/foo/bar
arewegit
echo $?
0
```


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
```

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
 

## Configuration and Environment variables

renf and friends will get their defaults by looking the following locations:

1. ./.renf : current directory
2. Any .renf file in parent folders upto, but not  exceeding the top-level .git
3. $HOME/.renf : If this exists, is used all child folders
4. $HOME/.config/renf : This is the expected default

All these config files are merely Bash language source scripts. They only
set environment variables like AUTOF, FMTF and PATF, .etc

The value of the most specific path to .renf can be inspected by:

```bash
echo $CONFIGF
/home/<you>/foo/.renf
```

### Environment variables

- CONFIGF : The path of the most specific .renf file (This is autoset)
- AUTOF : The numeric gap offset. (default: 10)
- PATF : The filename pattern. (default: [0-9][0-9][0-9]_*.md
LASTF : The last number which be added to $AUTOF  for the next invocation of newf
  * Set initially to the highest numbered file
- TPLTF : The path of the template file to be copied with the newf command
  * If unset, will default to the touch command
- FTPLT : The template function to be used in place of the setting of $TPLTF
- FMTF : The format string given to the 'printf' function. (default "%03d")



## Templates

The newf will, bby default, merely touch a new file given some combination
of $LASTF, $PATF and $AUTOF
However, if $TPLTF is set to a valid path, then that file will be 'cp' to the new patf filename

### Using the FTPLT function instead

If the $FTPLT variable is set, then that will override any possible setting
of $TPLF. This must be set to a function that takes the following argument:

```bash
# Note: nf_name will generate the next expected file name In this case: 020_file.md
# Inside newf:
$FTPLT $(nf_name)
020_file.md
```

#### Modifying the behaviour of the template function

newf will supply invocation-only variables before invocating the $FTPLT function.
These variable can be used by this function to create strings in the resulting file that is created.


- NUMF : The number of this file (E.g. 020
- INTF : The raw integer version of $NUMF(E.g. 20)
- FNF : The file part of the file (E.g. file)
- EXTF : The extension part of the file
- PREVF : The previous version of $LASTF. Useful for creating previous/next links. (E.g. 010)
- NEXTF : The value of $LASTF afternewf would have done its work. Useful to create next link (E.g. 030)
- FIRSTF : The value of the NUMF of the first file in the list. Useful to create the top link (E.g. 010)

```bash
# Full  complement of the $FTPLT invocation
NUMF=020 INTF=20 FILEF=file EXTF=.md PREVF=010 NEXTF=030 FIRSTF=010
```


Note: Obviously, any calls to renf that result in a reordering of the file
names. Use with caution.
will invalid any of these links




A better strategy is to allow the slide presentation software to both number and
link up the slides together. E.g. Reveal.js does this for you.

A plan for the future for this project will include a slide linking function

It is expected that the links will be contained out of band with the content here.



```bash
autof 1
renf
list --with-links
001_file.md
001_links.md
002_file.md
002_links.md
003_file.md
010_fi