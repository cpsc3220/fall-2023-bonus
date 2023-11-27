# CPSC/ECE 3220: Introduction to Operating System - Bonus Project

This project consists of one task that you will work on XV6. 
Task B aims to help you understand the index structure and the support of large files on XV6.

# Task B: Large Files (5% of Final Grade)
<code>Implement Large Files Support using doubly-indirect blocks on XV6</code>

## Overview - Large Files

In this task, you'll increase the maximum size of an xv6 file. 
Currently xv6 files are limited to 268 blocks, or 268*BSIZE bytes (BSIZE is 1024 in xv6). 
This limit comes from the fact that an xv6 inode contains 12 "direct" block numbers and 
one "singly-indirect" block number, which refers to a block that holds up to 256 more block numbers,
 for a total of 12+256=268 blocks.

Before writing code, you should read "Chapter 7: File system" from the xv6 book and study the corresponding code.

The `bigfile` user program creates the longest file it can, and reports that size:
```
$ bigfile
..
wrote 268 blocks
bigfile: file is too small
$
```
The test fails because the longest file is only 268 blocks.

You'll change the xv6 file system code to support a "doubly-indirect" block in each inode, 
containing 256 addresses of singly-indirect blocks, each of which can contain up to 256 addresses of data blocks. 
The result will be that a file will be able to consist of up to 256*256+256+11 blocks 
(11 instead of 12, because we will sacrifice one of the direct block numbers for the double-indirect block).


### Preliminaries

`mkfs` initializes the file system to have fewer than 2000 free data blocks, 
too few to show off the changes you'll make. Modify kernel/param.h to change FSSIZE from 2000 to 200,000:

```
    #define FSSIZE       200000  // size of file system in blocks
```

Rebuild `mkfs` so that is produces a bigger disk: `$ rm mkfs/mkfs fs.img; make mkfs/mkfs`


### What to Look At
+ The format of an on-disk inode is defined by struct dinode in fs.h. 
You're particularly interested in NDIRECT, NINDIRECT, MAXFILE, and the addrs[] element of struct dinode. 
Look at Figure 7.3 in the xv6 book for a diagram of the standard xv6 inode.

+ The code that finds a file's data on disk is in `bmap()` in fs.c. 
Have a look at it and make sure you understand what it's doing. 
`bmap()` is called both when reading and writing a file. 
When writing, `bmap()` allocates new blocks as needed to hold file content,
 as well as allocating an indirect block if needed to hold block addresses.

`bmap()` deals with two kinds of block numbers. The `bn` argument is a "logical block number" -- a block number 
within the file, relative to the start of the file. The block numbers in `ip->addrs[]`, and the argument to `bread()`,
 are disk block numbers. You can view `bmap()` as mapping a file's logical block numbers into disk block numbers.

## Specification

+ Your job is to modify bmap() so that it implements a doubly-indirect block, in addition to direct blocks and a singly-indirect block. 
+ You'll have to have only 11 direct blocks, rather than 12, to make room for your new doubly-indirect block. 
+ You're not allowed to change the size of an on-disk inode. 
   1. The first 11 elements of ip->addrs[] should be direct blocks;
   2. The 12th should be a singly-indirect block (just like the current one); 
   3. The 13th should be your new doubly-indirect block. 
+ You are done with this exercise when bigfile writes 65803 blocks and usertests runs successfully:
```
    $ bigfile
    ..................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
    wrote 65803 blocks
    done; ok
    $ usertests
    ...
    ALL TESTS PASSED
    $ 
```
bigfile will take at least a minute and a half to run.

## Hints:

+ Make sure you understand `bmap()`. Draw a diagram of the relationships between `ip->addrs[]`, 
the indirect block, the doubly-indirect block and the singly-indirect blocks it points to, and data blocks. 
+ Make sure you understand why adding a doubly-indirect block increases the maximum file size 
by 256*256 blocks (really -1, since you have to decrease the number of direct blocks by one).
+ Think about how you'll index the doubly-indirect block, and the indirect blocks it points to, 
with the logical block number.
+ If you change the definition of `NDIRECT`, you'll probably have to change the declaration of `addrs[]`
 in struct inode in file.h. 
+ Make sure that struct inode and struct dinode have the same number of elements in their addrs[] arrays.
+ If you change the definition of `NDIRECT`, make sure to create a new `fs.img`, 
since mkfs uses `NDIRECT` to build the file system.
+ If your file system gets into a bad state, perhaps by crashing, delete `fs.img` (do this from Unix, not xv6).
 `make` will build a new clean file system image for you.
+ Don't forget to `brelse()` each block that you `bread()`.
+ You should allocate indirect blocks and doubly-indirect blocks only as needed, like the original `bmap()`.
+ Make sure `itrunc` frees all blocks of a file, including double-indirect blocks. 

You can run `./grade-bigfile` to see your potential grade for this task.

## Submission
Please following the procedure below in your submission.

1. Switch to your project folder: 
```cd ~/path/to/your-projects-folder```
2. Clean up the folders
    ```
    cd ../TaskB
    make clean
    ```
3. List files that you have added or modified: ```git status```
4. Stage the new and modified  files in your local repo: ```git add files-you-created-or-modified```
5.  Commit the changes to your local repo: ```git commit -m "Finished Project #2"```
6. Push the changes to the remote repo (i.e. github): ```git push``` Or ```git push origin master```. 
7. Verify all changes have been push into the repo on github.
    + You can run "git status", "git log", and "git show" to the see the changes and commits you have made
    + You can also log into github to see if your changes have been actually pushed into your project repo on github.
8. (Optional but recommended)  For a further validation, you can check out the repo in a separate folder 
    on your computer and then verify that it has all the files for the programs to work correctly.
