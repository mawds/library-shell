---
title: "Shell Scripts"
teaching: 15
exercises: 0
questions:
- "How can I save and re-use commands?"
objectives:
- "Write a shell script that runs a command or series of commands for a fixed set of files."
- "Run a shell script from the command line."
- "Write a shell script that operates on a set of files defined by the user on the command line."
- "Create pipelines that include shell scripts you, and others, have written."
keypoints:
- "Save commands in files (usually called shell scripts) for re-use."
- "`bash filename` runs the commands saved in a file."
- "`$@` refers to all of a shell script's command-line parameters."
- "`$1`, `$2`, etc., refer to the first command-line parameter, the second command-line parameter, etc."
- "Place variables in quotes if the values might have spaces in them."
- "Letting users decide what files to process is more flexible and more consistent with built-in Unix commands."
---

We are finally ready to see what makes the shell such a powerful programming environment.
We are going to take the commands we repeat frequently and save them in files
so that we can re-run all those operations again later by typing a single command.
For historical reasons,
a bunch of commands saved in a text file is usually called a **shell script**,
but make no mistake:
these are actually small programs, which we can create and edit with a text editor.

> ## Which editor?
> If you are using a Linux machine, or a Mac, your machine will almost certainly have the `nano` editor installed.   This isn't installed as part of Git Bash, so we will use Windows' built in `notepad` editor instead.  In the examples that follow below substitute `notepad` for `nano`.  (If attempting to run notepad gives an error, try using `winpty notepad` instead).
> 
{: .callout}

> ## Text vs. Whatever
>
> We usually call programs like Microsoft Word or LibreOffice Writer "text
> editors", but we need to be a bit more careful when it comes to
> programming. By default, Microsoft Word uses `.docx` files to store not
> only text, but also formatting information about fonts, headings, and so
> on. This extra information isn't stored as characters, and doesn't mean
> anything to tools like `head`: they expect input files to contain
> nothing but the letters, digits, and punctuation on a standard computer
> keyboard. When editing programs, therefore, you must either use a plain
> text editor, or be careful to save files as plain text.
{: .callout}
Let's start by going back to the  `paine/`  directory and creating a new file, `countamerica.sh` which will
become our shell script:

~~~
$ cd paine
$ nano countamerica.sh
~~~
{: .bash}

The command `nano countamerica.sh` opens the file `countamerica.sh` within the text editor "nano"
(which runs within the shell).
If the file does not exist, it will be created.
We can use the text editor to directly edit the file -- we'll simply insert the following line:

~~~
bash countword K000934.000.txt america
~~~
{: .source}

This is a variation on the command we used earlier: It counts the number of times "america" appears in the file "K000934.000.txt"
Remember, we are *not* running it as a command just yet:
we are putting the command in a file.

Then we save the file (`Ctrl-O` in nano),
 and exit the text editor (`Ctrl-X` in nano).
Check that the directory `paine` now contains a file called `countamerica.sh`.

Once we have saved the file,
we can ask the shell to execute the commands it contains.
Our shell is called `bash`, so we run the following command:

~~~
$ bash countamerica.sh
~~~
{: .bash}

~~~
316 K000934.000.txt
~~~
{: .output}

Sure enough,
our script's output is exactly what we would get if we ran that command directly.


What if we want to count the number of times "america" appears in an arbitrary file?
We could edit `countamerica.sh` each time to change the filename,
but that would probably take longer than just retyping the command.
Instead, let's edit `countamerica.sh` and make it more versatile:

~~~
$ nano countamerica.sh
~~~
{: .bash}

Now, within "nano", replace the text `K000934.000.txt` with the special variable called `$1`:

~~~
bash countword "$1" america

~~~
{: .output}

Inside a shell script,
`$1` means "the first filename (or other parameter) on the command line".
We can now run our script like this:

> ## Double-Quotes Around Arguments
>
> We surround the `$1` with double quotes for the same reason that we put the loop variable inside double-quotes; 
> in case the filename happens to contain any spaces.
{: .callout}

~~~
$ bash countamerica.sh K000934.000.txt 
~~~
{: .bash}

~~~
316 K000934.000.txt 
~~~
{: .output}

or on a different file like this:

~~~
$ bash countamerica.sh K006500.000.txt 
~~~
{: .bash}

~~~
0 K006500.000.txt 
~~~
{: .output}


If our script takes more than one argument, we can refer to the second, third etc. arguments as `$2`, `$3`, etc. 


It's always a good idea to add **comments** to your scripts.  This way other people can figure out what your script does.  And *you* can figure out what it does when you come back to it several months after you wrote it.

~~~
$ nano countamerica.sh
~~~
{: .bash}

~~~
# Count the number of times the word "america" appears in a file
# The match is case insensitive
bash countword "$1" america
~~~
{: .output}

A comment starts with a `#` character and runs to the end of the line.
The computer ignores comments,
but they're invaluable for helping people (including your future self) understand and use scripts.
The only caveat is that each time you modify the script,
you should check that the comment is still accurate:
an explanation that sends the reader in the wrong direction is worse than none at all.

What if we want to process many files in a single pipeline?
For example, if we want to count the number of times "america" appears in each text, and sort the output by the number of times it appears.

Remember in the previous lesson that we could count the number of time "america" appeared, using the following `for` loop:

~~~
$ for datafile in K*.txt; do bash ./countword $datafile "america"; done
~~~
{: .bash}

If we wish to sort the list of files by the number of times "america" appears, we pipe the output to the sort command:

~~~
$ for datafile in K*.txt; do bash ./countword $datafile "america"; done | sort -n
~~~
{: .bash}

The `-n` parameter on the `sort` command tells it to sort numerically. 


> ## Sorting by number
>
> What happens if you omit the `-n` parameter on the `sort` command?
>
> > ## Solution
> >
> > The data are sorted in character order. The `sort` command sorts by the first character of each row, followed by the second, third, etc. characters.  This means that, for example,  "163" will come before "15".
> {: .solution}
{: .challenge}

We could put this in a script,
but then it would only ever count "america" in file starting with `K` and ending `.txt` in the current directory.
If we want to be able to count "america" in other kinds of files, 
we need a way to get all those names into the script.
We can't use `$1`, `$2`, and so on
because we don't know how many files there are.
Instead, we use the special variable `$@`,
which means,
"All of the command-line parameters to the shell script."
We also should put `$@` inside double-quotes
to handle the case of parameters containing spaces
(`"$@"` is equivalent to `"$1"` `"$2"` ...)
Here's an example:

~~~
$ nano countamericas.sh
~~~
{: .bash}

~~~
# Count the number of times "america" appears in one or more filenames
# and sort by the number of times the word appears
# Usage: bash countamericas.sh one_or_more_filenames
for datafile in "$@";
    do bash ./countword $datafile "america";
done | sort -n
~~~
{: .output}

~~~
$ bash countamericas.sh K*.txt ../33504-0.txt ../829-0.txt 
~~~
{: .bash}

~~~
0 ../33504-0.txt 
0 K011684.000.txt 
0 K023226.000.txt 
...
163 K023262.000.txt 
166 K134413.000.txt 
316 K000934.000.txt 

~~~
{: .output}

Suppose we have just run a series of commands that did something useful --- for example,
to count and save the number of times that various words appeared in a series of texts.
We'd like to be able to repeate the analysis if we need to,
so we want to save the commands in a file.
Instead of typing them in again
(and potentially getting them wrong)
we can do this:

~~~
$ history | tail -n 4 > wordscript.sh
~~~
{: .bash}

The file `wordscript.sh` now contains:

~~~
 2219  for datafile in K*.txt; do bash ./countword $datafile "america"; done > americacount
 2220  for datafile in K*.txt; do bash ./countword $datafile "rights"; done > rightscount
 2221  for datafile in K*.txt; do bash ./countword $datafile "liberty"; done > libertycount
 2222  history | tail -n 4 > wordscript.sh
~~~
{: .source}

After a moment's work in an editor to remove the serial numbers on the commands,
and to remove the final line where we called the `history` command,
we have a completely accurate record of how we created that figure.

In practice, most people develop shell scripts by running commands at the shell prompt a few times
to make sure they're doing the right thing,
then saving them in a file for re-use.
This style of work allows people to recycle
what they discover about their data and their workflow with one call to `history`
and a bit of editing to clean up the output
and save it as a shell script:

~~~
for datafile in K*.txt; do bash ./countword $datafile "america"; done > americacount
for datafile in K*.txt; do bash ./countword $datafile "rights"; done > rightscount
for datafile in K*.txt; do bash ./countword $datafile "liberty"; done > libertycount
~~~
{: .source}

There is a trade-off between replicability and flexibility.  The script as written will always select all files beginning with "K" and ending ".txt".  We could modify the script to let the user pass in the files to be used with the `$@` variable:

~~~
for datafile in $@; do bash ./countword $datafile "america"; done > americacount
for datafile in $@; do bash ./countword $datafile "rights"; done > rightscount
for datafile in $@; do bash ./countword $datafile "liberty"; done > libertycount
~~~
{: .source}

One possible solution to this issue is to write a master analysis script that will call this version of `wordscript.sh` with the appropriate wildcard:

~~~
# Analysis script for TCP texts
# Count americas and save
bash wordscript.sh K*.txt
# Do other analysis here
# ...

~~~
{: .source}

All the analyses can then be run by calling `bash wordscript.sh`


> ## Variables in Shell Scripts
>
> In the `authors` directory, imagine you have a shell script called `script.sh` containing the
> following commands:
>
> ~~~
> head -n $2 $1
> tail -n $3 $1
> ~~~
> {: .bash}
>
> While you are in the `authors` directory, you type the following command:
>
> ~~~
> bash script.sh '*.txt' 1 1
> ~~~
> {: .bash}
>
> Which of the following outputs would you expect to see?
>
> 1. All of the lines between the first and the last lines of each file ending in `.txt`
>    in the `authors` directory
> 2. The first and the last line of each file ending in `.txt` in the `authors` directory
> 3. The first and the last line of each file in the `authors` directory
> 4. An error because of the quotes around `*.txt`
>
> > ## Solution
> > The correct answer is 2. 
> >
> > The special variables $1, $2 and $3 represent the command line arguments given to the
> > script, such that the commands run are:
> >
> > ```
> > $ head -n 1 Addison.txt Aikin.txt Brown.txt Cowley.txt Defoe.txt Goldsmith.txt Pope.txt Trusler.txt
> > $ tail -n 1 Addison.txt Aikin.txt Brown.txt Cowley.txt Defoe.txt Goldsmith.txt Pope.txt Trusler.txt
> > ```
> > {: .bash}
> > The shell does not expand `'*.txt'` because it is enclosed by quote marks.
> > As such, the first argument to the script is `'*.txt'` which gets expanded _within_ the
> > script by `head` and `tail`.
> {: .solution}
{: .challenge}


> ## Find the Longest File With a Given Extension
>
> Write a shell script called `longest.sh` that takes the name of a
> directory and a filename extension as its parameters, and prints
> out the name of the file with the most lines in that directory
> with that extension. For example:
>
> ~~~
> $ bash longest.sh /tmp/data txt
> ~~~
> {: .bash}
>
> would print the name of the `.txt` file in `/tmp/data` that has
> the most lines.
>
> > ## Solution
> >
> > ```
> > # Shell script which takes two arguments: 
> > #    1. a directory name
> > #    2. a file extension
> > # and prints the name of the file in that directory
> > # with the most lines which matches the file extension.
> > 
> > wc -l $1/*.$2 | sort -n | tail -n 2 | head -n 1
> > ```
> > {: .source}
> {: .solution}
{: .challenge}

> ## Why Record Commands in the History Before Running Them?
>
> If you run the command:
>
> ~~~
> $ history | tail -n 5 > recent.sh
> ~~~
> {: .bash}
>
> the last command in the file is the `history` command itself, i.e.,
> the shell has added `history` to the command log before actually
> running it. In fact, the shell *always* adds commands to the log
> before running them. Why do you think it does this?
>
> > ## Solution
> > If a command causes something to crash or hang, it might be useful
> > to know what that command was, in order to investigate the problem.
> > Were the command only be recorded after running it, we would not
> > have a record of the last command run in the event of a crash.
> {: .solution}
{: .challenge}

> ## Script Reading Comprehension
>
> For this question, consider the `authors` directory once again.
> This contains a number of `.txt` files. 
> Explain what a script called `example.sh` would do when run as
> `bash example.sh *.txt` if it contained the following lines:
>
> ~~~
> # Script 1
> echo *.*
> ~~~
> {: .bash}
>
> ~~~
> # Script 2
> for filename in $1 $2 $3
> do
>     cat $filename
> done
> ~~~
> {: .bash}
>
> ~~~
> # Script 3
> echo $@.out
> ~~~
> {: .bash}
>
> > ## Solutions
> > Script 1 would print out a list of all files containing a dot in their name.
> >
> > Script 2 would print the contents of the first 3 files matching the file extension.
> > The shell expands the wildcard before passing the arguments to the `example.sh` script.
> >
> > Script 3 would print all the arguments to the script (i.e. all the `.txt` files),
> > followed by `.out`, i.e:
> > 
> > ```
> > Addison.txt Aikin.txt Brown.txt Cowley.txt Defoe.txt Goldsmith.txt Pope.txt Trusler.txt.out
> > ```
> > {: .output}
> {: .solution}
{: .challenge}
