---
title: "Loops"
teaching: 15
exercises: 0
questions:
- "How can I perform the same actions on many different files?"
objectives:
- "Write a loop that applies one or more commands separately to each file in a set of files."
- "Trace the values taken on by a loop variable during execution of the loop."
- "Explain the difference between a variable's name and its value."
- "Explain why spaces and some punctuation characters shouldn't be used in file names."
- "Demonstrate how to see what commands have recently been executed."
- "Re-run recently executed commands without retyping them."
keypoints:
- "A `for` loop repeats commands once for every thing in a list."
- "Every `for` loop needs a variable to refer to the thing it is currently operating on."
- "Use `$name` to expand a variable (i.e., get its value). `${name}` can also be used."
- "Do not use spaces, quotes, or wildcard characters such as '*' or '?' in filenames, as it complicates variable expansion."
- "Give files consistent names that are easy to match with wildcard patterns to make it easy to select them for looping."
- "Use the up-arrow key to scroll up through previous commands to edit and repeat them."
- "Use `Ctrl-R` to search through the previously entered commands."
- "Use `history` to display recent commands, and `!number` to repeat a command by number."
---

**Loops** are key to productivity improvements through automation as they allow us to execute
commands repetitively. Similar to wildcards and tab completion, using loops also reduces the
amount of typing (and typing mistakes).
Suppose we have several hundred Etext files named `33504-0.txt`, `829-0.txt`, and so on.
In this example,
we'll use the data we used in previous lessons which only has two example files,
but the principles can be applied to many many more files at once.
We would like to modify these files, but also save a version of the original files, naming the copies
`original-33504-0.txt`, `original-829-0.txt`
We can't use:

~~~
$ cp *.txt original-*.txt
~~~
{: .bash}

because that would expand to:

~~~
$ cp 33504-0.txt 829-0.txt original-*.txt
~~~
{: .bash}

This wouldn't back up our files, instead we get an error:

~~~
cp: target `original-*.txt' is not a directory
~~~
{: .error}

This problem arises when `cp` receives more than two inputs. When this happens, it
expects the last input to be a directory where it can copy all the files it was passed.
Since there is no directory named `original-*.txt` in the `creatures` directory we get an
error.

Instead, we can use a **loop**
to do some operation once for each thing in a list.
Here's a simple example that displays the first three lines of each file in turn:

~~~
$ for filename in 33504-0.txt 829-0.txt
> do
>    head -n 3 $filename
> done
~~~
{: .bash}

~~~
The Project Gutenberg EBook of Opticks, by Isaac Newton

This eBook is for the use of anyone anywhere at no cost and with
The Project Gutenberg eBook, Gulliver's Travels, by Jonathan Swift
~~~
{: .output}

When the shell sees the keyword `for`,
it knows to repeat a command (or group of commands) once for each thing `in` a list.
For each iteration,
the name of the each thing is sequentially assigned to
the **variable** and the commands inside the loop are executed before moving on to 
the next thing in the list.
Inside the loop,
we call for the variable's value by putting `$` in front of it.
The `$` tells the shell interpreter to treat
the **variable** as a variable name and substitute its value in its place,
rather than treat it as text or an external command. 

In this example, the list is two filenames: `33504-0.txt` and `829-0.txt`.
Each time the loop iterates, it will assign a file name to the variable `filename`
and run the `head` command.
The first time through the loop,
`$filename` is `33504-0.txt`. 
The interpreter runs the command `head` on `33504-0.txt`, 
and the prints the 
first three lines of `basilisk.dat`.
For the second iteration, `$filename` becomes 
`829-0.txt`. This time, the shell runs `head` on `829-0.txt`
and prints the first three lines of `unicorn.dat`.
Since the list was only two items, the shell exits the `for` loop.

When using variables it is also
possible to put the names into curly braces to clearly delimit the variable
name: `$filename` is equivalent to `${filename}`, but is different from
`${file}name`. You may find this notation in other people's programs.

> ## Follow the Prompt
>
> The shell prompt changes from `$` to `>` and back again as we were
> typing in our loop. The second prompt, `>`, is different to remind
> us that we haven't finished typing a complete command yet. A semicolon, `;`,
> can be used to separate two commands written on a single line.
{: .callout}

> ## Same Symbols, Different Meanings
>
> Here we see `>` being used a shell prompt, whereas `>` is also
> used to redirect output.
> Similarly, `$` is used as a shell prompt, but, as we saw earler,
> it is also used to ask the shell to get the value of a variable.
>
> If the *shell* prints `>` or `$` then it expects you to type something,
> and the symbol is a prompt.
>
> If *you* type `>` or `$` yourself, it is an instruction from you that
> the shell to redirect output or get the value of a variable.
{: .callout}

We have called the variable in this loop `filename`
in order to make its purpose clearer to human readers.
The shell itself doesn't care what the variable is called;
if we wrote this loop as:

~~~
for x in 33504-0.txt 829-0.txt
do
    head -n 3 $x
done
~~~
{: .bash}

or:

~~~
for temperature in 33504-0.txt 829-0.txt
do
    head -n 3 $temperature
done
~~~
{: .bash}

it would work exactly the same way.
*Don't do this.*
Programs are only useful if people can understand them,
so meaningless names (like `x`) or misleading names (like `temperature`)
increase the odds that the program won't do what its readers think it does.

Here's a slightly more complicated loop:

~~~
for filename in *.txt
do
    echo $filename
    head -n 100 $filename | tail -n 20
done
~~~
{: .bash}

The shell starts by expanding `*.txt` to create the list of files it will process.
The **loop body**
then executes two commands for each of those files.
The first, `echo`, just prints its command-line parameters to standard output.
For example:

~~~
$ echo hello there
~~~
{: .bash}

prints:

~~~
hello there
~~~
{: .output}

In this case,
since the shell expands `$filename` to be the name of a file,
`echo $filename` just prints the name of the file.
Note that we can't write this as:

~~~
for filename in *.txt
do
    $filename
    head -n 100 $filename | tail -n 20
done
~~~
{: .bash}

because then the first time through the loop,
when `$filename` expanded to `33504-0.txt`, the shell would try to run `33504-0.txt` as a program.
Finally,
the `head` and `tail` combination selects lines 81-100
from whatever file is being processed
(assuming the file has at least 100 lines).

> ## Spaces in Names
>
> Whitespace is used to separate the elements on the list
> that we are going to loop over. If on the list we have elements
> with whitespace we need to quote those elements
> and our variable when using it.
> Suppose our data files are named:
>
> ~~~
> Swift Gulliver.txt
> Newton Opticks.txt
> ~~~
> {: .source}
> 
> We need to use
> 
> ~~~
> for filename in "Swift Gulliver.txt" "Newton Opticks.txt"
> do
>     head -n 100 "$filename" | tail -n 20
> done
> ~~~
> {: .bash}
>
> It is simpler just to avoid using whitespaces (or other special characters) in filenames.
{: .callout}

Going back to our original file copying problem,
we can solve it using this loop:

~~~
for filename in *.txt
do
    cp $filename original-$filename
done
~~~
{: .bash}

This loop runs the `cp` command once for each filename.
The first time,
when `$filename` expands to `33504-0.txt`,
the shell executes:

~~~
cp 33504-0.txt original-33504-0.txt
~~~
{: .bash}

The second time, the command is:

~~~
cp 829-0.txt original-829-0.txt
~~~
{: .bash}

Since the `cp` command does not normally produce any output, it's hard to check 
that the loop is doing the correct thing. By prefixing the command with `echo` 
it is possible to see each command as it _would_ be executed:

~~~
for filename in *.txt
do
    echo $filename
    cp $filename original-$filename
done
~~~
{: .bash}

## Processing many files

The `shell-lesson.zip` file which you downloded earlier contains a directory called `paine`.  This contains some of the works of Thomas Paine, in plain text format.  These are part of the [ECCO-TCP](http://www.textcreationpartnership.org/tcp-ecco/), and were downloaded from <https://github.com/Anterotesis/historical-texts>.

We'll first take a look at the contents of the directory.

~~~
$ cd paine
$ ls
~~~
{: .bash}

We see that all the files start 'K' and end '.txt'.


> ## Viewing files' contents
> We can view the contents of a file using the `less` command, followed by the file we wish to view.    Try viewing the contents of one of the texts.
{: .callout}

   The directory also contains a program (strictly a shell script--which we will cover in the next lesson) that will count the number of occurences 
of a word within a file (ignoring case).  The filenames, and the number of times the word occured will be printed. For example, if we had a text file containing: "The cat sat on the mat", and ran:

~~~
$ bash countword cat.txt "the"
> 2 cat.txt
~~~
{: .bash}

In this part of the lesson we will use the countword program and some loops to look at how often certain words occur in the texts.  We will use a _script_ to make it easier to re-run and adapt this analysis. A script is a collection of commands, stored in a text file.  The `countword` program used above is actually a script.

Let's use the `countword` program to count the number of times "America" occurs in each text.  We can do this using a loop.  As we're learning to use the shell, we will build up the required commands in stages.

Our first step is to select the right files.
We saw that all the files started with a  'K' and ended '.txt'. We only want to process these files.

~~~
$ for datafile in K*.txt
> do
>     echo $datafile
> done
~~~
{: .bash}

~~~
K000934.000.txt
K006500.000.txt
K011684.000.txt
...
K133948.000.txt
K134413.000.txt
~~~
{: .output}

Our next step is to call the countword program with the appropriate parameters for each file.  It's good practice to check that the script is going to run the commands you think it is by `echo`ing the commands to the screen first:


~~~
$ for datafile in K*.txt
> do echo bash ./countword $datafile "america"
> done
~~~
{: .bash}

~~~
bash ./countword K000934.000.txt america
bash ./countword K006500.000.txt america
bash ./countword K011684.000.txt america
...
bash ./countword K133430.000.txt america
bash ./countword K133948.000.txt america
bash ./countword K134413.000.txt america
~~~
{: .output}

We haven't actually run `countword` yet, but we can see the commands are correct.  Before we run the loop, let's take a brief diversion to look at how to save time and reduce the potential for errors.

## Keyboard shortcuts

Typing in commands over and over again is becoming tedious,
though.  In this section we look at some shortcuts that can save a lot of time, and reduce the potential for errors.

If we press the up arrow, 
the shell redisplays the whole loop on one line
(using semi-colons to separate the pieces):

~~~
$ for datafile in K*.txt; do echo bash ./countword $datafile "america"; done
~~~
{: .bash}

Using the left arrow key, we can delete the `echo` command:

~~~
$ for datafile in K*.txt; do bash ./countword $datafile "america"; done
~~~
{: .bash}

> ## Beginning and End
>
> We can move to the beginning of a line in the shell by typing `Ctrl-A`
> and to the end using `Ctrl-E`.
{: .callout}


> ## Those Who Know History Can Choose to Repeat It
>
> Another way to repeat previous work is to use the `history` command to
> get a list of the last few hundred commands that have been executed, and
> then to use `!123` (where "123" is replaced by the command number) to
> repeat one of those commands. For example, if we type: 
>
> ~~~
> $ history | tail -n 5
> ~~~
> {: .bash}
> ~~~
 2105  ls
 2106  for datafile in K*.txt; do echo countword $datafile "america"; done
 2107  for datafile in K*.txt; do echo bash ./countword $datafile "america"; done
 2108  for datafile in K*.txt; do bash ./countword $datafile "america"; done
 2109  history |tail -n 5
> ~~~
> {: .output}
>
> then we can re-run `countword` on our files by typing 
> `!2108`.
{: .callout}

> ## Other History Commands
>
> There are a number of other shortcut commands for getting at the history.
>
> - `Ctrl-R` enters a history search mode "reverse-i-search" and looks for
> matches to the text you enter next.
> Press `Ctrl-R` again to cycle through matches.
> - `!!` retrieves the immediately preceding command 
> (you may or may not find this more convenient than using the up-arrow)
> - `!$` retrieves the last word of the  last command.
> That's useful more often than you might expect: after
> `bash goostats NENE01729B.txt stats-NENE01729B.txt`, you can type
> `less !$` to look at the file `stats-NENE01729B.txt`, which is
> quicker than doing up-arrow and editing the command-line.
{: .callout}

> ## Variables in Loops
>
> This exercise refers to a directory containing the following files: 
>
> ~~~
> cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
> ~~~
> {: .output}
>
> What is the output of the following code?
>
> ~~~
> for datafile in *.pdb
> do
>     ls *.pdb
> done
> ~~~
> {: .bash}
>
> Now, what is the output of the following code?
>
> ~~~
> for datafile in *.pdb
> do
>	ls $datafile
> done
> ~~~
> {: .bash}
>
> Why do these two loops give different outputs?
>
> > ## Solution
> > The first code block gives the same output on each iteration through
> > the loop.
> > Bash expands the wildcard `*.pdb` within the loop body (as well as
> > before the loop starts) to match all files ending in `.pdb`
> > and then lists them using `ls`.
> > The expanded loop would look like this:
> > ```
> > for datafile in cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
> > do
> >	ls cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
> > done
> > ```
> > {: .bash}
> >
> > ```
> > cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
> > cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
> > cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
> > cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
> > cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
> > cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
> > ```
> > {: .output}
> >
> > The second code block lists a different file on each loop iteration.
> > The value of the `datafile` variable is evaluated using `$datafile`,
> > and then listed using `ls`.
> >
> > ```
> > cubane.pdb
> > ethane.pdb
> > methane.pdb
> > octane.pdb
> > pentane.pdb
> > propane.pdb
> > ```
> > {: .output}
> {: .solution}
{: .challenge}

> ## Saving to a File in a Loop - Part One
>
> In the same directory, what is the effect of this loop?
>
> ~~~
> for species in *.pdb
> do
>     echo $species
>     cat $species > alkanes.pdb
> done
> ~~~
> {: .bash}
>
> 1.  Prints `cubane.pdb`, `ethane.pdb`, `methane.pdb`, `octane.pdb`, `pentane.pdb` and `propane.pdb`,
>     and the text from `propane.pdb` will be saved to a file called `alkanes.pdb`.
> 2.  Prints `cubane.pdb`, `ethane.pdb`, and `methane.pdb`, and the text from all three files would be
>     concatenated and saved to a file called `alkanes.pdb`.
> 3.  Prints `cubane.pdb`, `ethane.pdb`, `methane.pdb`, `octane.pdb`, and `pentane.pdb`, and the text
>     from `propane.pdb` will be saved to a file called `alkanes.pdb`.
> 4.  None of the above.
>
> > ## Solution
> > 1. The text from each file in turn gets written to the `alkanes.pdb` file.
> > However, the file gets overwritten on each loop interation, so the final content of `alkanes.pdb`
> > is the text from the `propane.pdb` file.
> {: .solution}
{: .challenge}

> ## Saving to a File in a Loop - Part Two
>
> In the same directory, what would be the output of the following loop?
>
> ~~~
> for datafile in *.pdb
> do
>     cat $datafile >> all.pdb
> done
> ~~~
> {: .bash}
>
> 1.  All of the text from `cubane.pdb`, `ethane.pdb`, `methane.pdb`, `octane.pdb`, and
>     `pentane.pdb` would be concatenated and saved to a file called `all.pdb`.
> 2.  The text from `ethane.pdb` will be saved to a file called `all.pdb`.
> 3.  All of the text from `cubane.pdb`, `ethane.pdb`, `methane.pdb`, `octane.pdb`, `pentane.pdb`
>     and `propane.pdb` would be concatenated and saved to a file called `all.pdb`.
> 4.  All of the text from `cubane.pdb`, `ethane.pdb`, `methane.pdb`, `octane.pdb`, `pentane.pdb`
>     and `propane.pdb` would be printed to the screen and saved to a file called `all.pdb`.
>
> > ## Solution
> > 3 is the correct answer. `>>` appends to a file, rather than overwriting it with the redirected
> > output from a command.
> > Given the output from the `cat` command has been redirected, nothing is printed to the screen.
> {: .solution}
{: .challenge}

> ## Limiting Sets of Files
>
> In the same directory, what would be the output of the following loop?
>
> ~~~
> for filename in c*
> do
>     ls $filename 
> done
> ~~~
> {: .bash}
>
> 1.  No files are listed.
> 2.  All files are listed.
> 3.  Only `cubane.pdb`, `octane.pdb` and `pentane.pdb` are listed.
> 4.  Only `cubane.pdb` is listed.
>
> > ## Solution
> > 4 is the correct answer. `*` matches zero or more characters, so any file name starting with 
> > the letter c, followed by zero or more other characters will be matched.
> {: .solution}
>
> How would the output differ from using this command instead?
>
> ~~~
> for filename in *c*
> do
>     ls $filename 
> done
> ~~~
> {: .bash}
>
> 1.  The same files would be listed.
> 2.  All the files are listed this time.
> 3.  No files are listed this time.
> 4.  The files `cubane.pdb` and `octane.pdb` will be listed.
> 5.  Only the file `octane.pdb` will be listed.
>
> > ## Solution
> > 4 is the correct answer. `*` matches zero or more characters, so a file name with zero or more
> > characters before a letter c and zero or more characters after the letter c will be matched.
> {: .solution}
{: .challenge}

> ## Doing a Dry Run
>
> A loop is a way to do many things at once --- or to make many mistakes at
> once if it does the wrong thing. One way to check what a loop *would* do
> is to `echo` the commands it would run instead of actually running them.
> 
> Suppose we want to preview the commands the following loop will execute
> without actually running those commands:
>
> ~~~
> for file in *.pdb
> do
>   analyze $file > analyzed-$file
> done
> ~~~
> {: .bash}
>
> What is the difference between the two loops below, and which one would we
> want to run?
>
> ~~~
> # Version 1
> for file in *.pdb
> do
>   echo analyze $file > analyzed-$file
> done
> ~~~
> {: .bash}
>
> ~~~
> # Version 2
> for file in *.pdb
> do
>   echo "analyze $file > analyzed-$file"
> done
> ~~~
> {: .bash}
>
> > ## Solution
> > The second version is the one we want to run.
> > This prints to screen everything enclosed in the quote marks, expanding the
> > loop variable name because we have prefixed it with a dollar sign.
> >
> > The first version redirects the output from the command `echo analyze $file` to
> > a file, `analyzed-$file`. A series of files is generated: `cubane.pdb`,
> > `ethane.pdb` etc.
> > 
> {: .solution}
{: .challenge}

> ## Nested Loops
>
> Suppose we want to set up up a directory structure to organize
> some experiments measuring reaction rate constants with different compounds
> *and* different temperatures.  What would be the
> result of the following code:
>
> ~~~
> for species in cubane ethane methane
> do
>     for temperature in 25 30 37 40
>     do
>         mkdir $species-$temperature
>     done
> done
> ~~~
> {: .bash}
>
> > ## Solution
> > We have a nested loop, i.e. contained within another loop, so for each species
> > in the outer loop, the inner loop (the nested loop) iterates over the list of
> > temperatures, and creates a new directory for each combination.
> >
> > Try running the code for yourself to see which directories are created!
> {: .solution}
{: .challenge}
