# Basic Data Wrangling
Niels Hanson  
October 30, 2014  

Pourpose of this document is to summarize some basic data wrangling tasks that commonly occuring with metagenomic datasets in the Hallam Lab.

## Unix Shell Commands: grep, awk, sed, and others

Some very-simple first-pass analyses can be performed with common Unix commands. Although there are others, many analysis and sanity checking tasks can be accomplished by the following commands:

* `grep`: `g`eneral `r`egular `e`xpression `p` program -- at its simplist a program that can **find text patterns** in the **names and contents of files**.
* `sed`: `s`tream `ed`itor -- a general pourpose program to modify a stream of text, which generally is a file or program output
* `awk`: (pronounced 'auk' or 'ach') is a **basic text processing** and **report generating language** developed by Alfred `a`ho, Peter `w`einberger, and Brian `k`ernighan at Bell Labs in the 1970s

### When to use?

Lets discuss some advantages and disadvantages and when these commands (by themselves) are a good option.

#### Advantages 

* these commands are basically found on every Unix environment
* fairly efficient and low-level and so will scale to fairly large files
* excellent for making back-of-the-envelope kinds of calculations to make sure files are properly formatted, the correct size, simple 'one-off' analyses

#### Disadvantages

* small isocyncracies in system experience needed to know command's behavior
* small mistakes can lead to huge headaches permenant data loss
* many commands have their GNU open-source sister program (e.g., `gawk` has different behavior than `awk`, but not always installed on every system) 
* behavior changes depending on version installed on OS (annoying)
* unless process is scripted (see next section) and documented carefully, complicated commands can be very diffcult to follow and reproduce (best to keep things simple)

### Examples

Lets go through an number of examples where I personally use these in the lab.

* if you're following along in the terminal, change directory `cd` to the `examples/unix_commands/` directory

```
cd examples/unix_commands/
```

#### grep

* Counting the number of sequences in a fasta file
    * the `-c` option means 'count'
    * this will return he number of times the `'>'` pattern is observed in the fasta file
    * if the file is well-formatted, this will accurately tell you how many sequences there are in the file

```
grep -c '>' fastas/GUAB.fasta
9948
```

* this can also be done to every file that matches a particular file-name pattern in a directory with the **unix glob operator** (e.g., `*.fasta`)

```
grep -c '>' fastas/*.fasta
```

* count the number of sequences in all files in all subdirectories

```
grep ">" -c -r *
fastas/.DS_Store:0
fastas/GUAB.fasta:9948
fastas/GUAC.fasta:8850
fastas/INX043_RawGSCdata_min2000.fasta.NR00314_J24.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NO00111_F15.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NO00111_J19.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NR0031_K08.fasta:1
fastas/sub_dir2/INX043_RawGSCdata_min2000.fasta.SCR021_C16.fasta:1
fastas/sub_dir2/INX043_RawGSCdata_min2000.fasta.SCR044_H13.fasta:1
fastas/sub_dir2/NapDC_July06_2011_trimmed.fasta:15360
fastas/TOLDC_Feb22_2011_trimmed.fasta:3072
```

* but, if we were only interested in the `fasta` files use the `--include '.fasta'` option

```
grep -c -r --include *.fasta* '>' *
fastas/GUAB.fasta:9948
fastas/GUAC.fasta:8850
fastas/INX043_RawGSCdata_min2000.fasta.NR00314_J24.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NO00111_F15.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NO00111_J19.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NR0031_K08.fasta:1
fastas/sub_dir2/INX043_RawGSCdata_min2000.fasta.SCR021_C16.fasta:1
fastas/sub_dir2/INX043_RawGSCdata_min2000.fasta.SCR044_H13.fasta:1
fastas/sub_dir2/NapDC_July06_2011_trimmed.fasta:15360
fastas/TOLDC_Feb22_2011_trimmed.fasta:3072
```

* how about if were only interested in files that had `INX043` and were `.fasta` files anywhere in the folder `fastas/`
   * use the anycharacter glob (`*`) to select text on both sides

```
grep -c -r --include *INX043*.fasta* '>' *
fastas/INX043_RawGSCdata_min2000.fasta.NR00314_J24.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NO00111_F15.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NO00111_J19.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NR0031_K08.fasta:1
fastas/sub_dir2/INX043_RawGSCdata_min2000.fasta.SCR021_C16.fasta:1
fastas/sub_dir2/INX043_RawGSCdata_min2000.fasta.SCR044_H13.fasta:1
```

* and if I just wanted to look into directories that had the pattern `sub_dir*`

```
grep -c '>' fastas/sub_dir*/*fasta
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NO00111_F15.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NO00111_J19.fasta:1
fastas/sub_dir1/INX043_RawGSCdata_min2000.fasta.NR0031_K08.fasta:1
fastas/sub_dir2/INX043_RawGSCdata_min2000.fasta.SCR021_C16.fasta:1
fastas/sub_dir2/INX043_RawGSCdata_min2000.fasta.SCR044_H13.fasta:1
fastas/sub_dir2/NapDC_July06_2011_trimmed.fasta:15360
```

* there is also an `--exclude` to do the opposite of `--include`
    * here we look for `'>'` for all files in all subdirectories that do not end in `fasta`

```
grep -c -r --exclude '*.fasta' '>' *
command_list.sh:6
fastas/.DS_Store:0
```

##### Asside: Unix globs

* Note that there is quite a bit of power to these Unix 'glob' operators:
    * `*`: means any number of characters
        * `beginning*end`: selects files with a particular `beginning` and `end` patterns
        * `beginning*middle*end`: selects files with particular `beginning`, `middle`, and `end` text
        * `sample*/blast_results/*parsed_blast.txt`: matches all `parsed_blast.txt` files that can be found in sub-directories `blast_results/` below directries starting with `sample*`
    * `?`: matches exactly one unknown character
        * `?at`: matches `fat`, `cat`, `hat`, `sat`, etc.
    * `[]`: specifies a range or set of characters to match
        * `[BC]at`: matches `Bat` or `Cat` but not `bat` or `cat`
        * `sample_[0-9][0-9]`: matches `sample_01`, `sample_02`, etc.
* the linux command `ls` can be used to list the set of files that are hit by a glob pattern
    * this is very important when writing critical commands like 'move' (`mv`) or 'remove' (`rm`)
        * everyone eventually burns themselves, its a fact of life while working on with the Unix command line
* *Note: Unix globs are not as sophisicated as full blown Perl or grep-based regular expressions. They don't have a complete feature set. Some things just can not be done with command line globs*

#### `grep` (con't)

* dropping the `-c`, `grep` will just print the matching line
    * match all fasta headers that end in `b1`

```
grep '>*b1' fastas/GUAB.fasta
>GUAB5590.b1
>GUAB3843.b1
>GUAB6002.b1
>GUAB6573.b1
```

* the `-n` option will print out the line number where the match was found
    if more than one file being searched the filename will be printed as well

```
grep -n '>*b1' fastas/GUAB.fasta
1:>GUAB5590.b1
3:>GUAB3843.b1
7:>GUAB6002.b1
11:>GUAB6573.b1
...
```

* if the fasta file format is well-formatted (contains the sequence only on one line after matching header), then you can use the 'after' `-A` flag to print a number of lines after the match.
    * *Note: there are also 'before' `-B` and 'context' `-C` flags that print out lines before the match and on both sides, respectively.* 
    * here we use it to get the sequence along with header patterns that end in `.b1`, because we know that it is the first line below in a well-formatted `.fasta` file
        * *Note: this won't work for fasta files where the sequence in on multiple lines

```
grep -A 1  '>*b1' fastas/GUAB.fasta
>GUAB5590.b1
ATCGAGTGTGTTCTTTTGCGCGATGGTATTCGACGTACCGTTTGCATTTCATCACAAGTCGGCTGTGCAATGGGGTGTGTATTCTGTGCAAGCGGTCTTGATGGAGTTATCCGAAATTTGACAACCGGTGAGATCATCGAGCAGTTATTACGACTCACTCGTTTACTCCCAACAGAAGAACGACTAAGTCATATTGTTGTCATGGGAATGGGTGAACCACTAGCAAACCTCGATCGTTTATTACCTGCTCTTGCAATCGCACAAAGTCCTGAGGGACTTGGTATATCTCAACGACGAATCACTATTTCAACTGTCGGGTTGCCATCGGCAATTGATCGGCTGTGTCAAGAAAATCCTGGATATCATCTTGCCGTATCACTCCATGCCGCTGACGATCCACTCCGAACAAAACTCGTTCCAGTCAATAAGTCGATCGGAGTTCATGCCATTTTAGCCGCAGCTGACCGATACTGGGAAACATCTGGCCGACGACTTACTTTCGAATATGTCTTACTCGGAAATCTTAATGATTCTCCAGATCATGCCCGTTCGTTAGCTCGGTTTATCGGCAAACGTGCAGCGCTCGTAAACATTATTCCGTACAACACAGTCGATGGCCTACCGTGGGAAGAGCCTACTGACATTTCTCGTGAACGATTTCTTGATGTCCTTTCGAATGCTGGCGTGAATGTTCAGACTCGAAAAAGGAG
--
>GUAB3843.b1
CTGGGTGGCAGCGGTGTAGAGAAAATAGTTGAACTTTCGCTTTTGCCAGATGAAAAAGTGGCATTCAATAAAAGTATCGATGCAGTTCGGGAACTTGTTGGCGCAATGGAGAGCCTGACGCCATAACAGAAGCCCCCCAGTCCTCCCAGCCTCCCTCTGTCGGCTAGTTTCGCGAAAGCTCTTCATTTTCACACGGAAATGCTGCCATTGCCCGGAGTGTGTGACTGCGTTAGAAACCTCCGACGCATTGCCATAAGTCAAATCTCTTGTGGCAATGCGTTAGCAATCACGCTGTGTATTTTTGCAAAAGGTTACGCGGAGGATCTGGAATGGTGCATCAGTCGACAGTATTTGATTATGAATTCAATCATTGTTCACCAAGCAGTAGAGGCTTTTTGTGCAACGAGGTGAATCTCCCTTCGAGTAAACCAGAGCAACTGTTTTTACCACGAGGGTATGAGTCGGGATACGACTACCCACTCCTCGTGTGGCTTCCGCAGTCAGACGATGCTCACTTTGACTTAGGTCGCACCATGATGCGAATGAGTTTAAGGAATTACATTGCAGTAGTACCTGCTGTCACGTCTGATTTGGAGAGTTGTTTTGAAGCTATTGACGGAATAATGAGTCAATATAGCGTTCACTCTCGCAGATTTTATCTTATTGGTGTTGAGGAAGGTGGAGAGAATGCCTTTCGTTATGCTTGCCAAAATC
...
```

* we can now use 'write to file' operator `>` to safe this output to separate `.fasta` file
* whatever is print to the terminal (standard out) will be saved in the file `my_fasta.fasta`

```
grep -A 1  '>*b1' fastas/GUAB.fasta > my_fasta.fasta
```

* *Note: this isn't totally correct, because `grep` puts `--` characters between matches, which could cause fasta parsers to choke on the input. Solving this problem actually takes us into our next section which is on the Uinx 'Sequence Editor' program `sed`*

#### `Sed`
