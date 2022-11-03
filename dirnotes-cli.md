---
title: dirnotes-cli
section: 1
header: User Manual
footer: dirnotes-cli 1.0
date: Sept 26, 2022
---

# NAME

dirnotes-cli - view and manage Dirnotes file annotations

# SYNOPSIS

<!--- the indent below is using unicode hard space u00a0, and space at EOL  --->   
    **dirnotes-cli** [-h] [--help] [-V] [--version]   

  List:   
    **dirnotes-cli** [-v] [-j] [-l] [-d|-x]  
    **dirnotes-cli** [-v] [-j] [-n] [-d|-x] [filename]...   
    **dirnotes-cli** [-v] [-j] -H [filename]...  
  
  Create:  
    **dirnotes-cli** -c "comment" filename...  
    **dirnotes-cli** -a "comment" filename...  
    **dirnotes-cli** -e filename...

  FileCopy:  
    **dirnotes-cli** -C [-y] src-file... <dest-file | dest-dir>  
    **dirnotes-cli** -M [-y] src-file... <dest-file | dest-dir>

  Cleanup:  
    **dirnotes-cli** -z [filename]...   
    **dirnotes-cli** -Z

# DESCRIPTION

The **dirnotes** family of apps allows you to add a descriptive comment to 
a file. The descriptions are stored in two places:

  * in the xattr properties of the file
  * in a database located in the user's home directory

The **dirnotes-cli** is designed for use on the command line, or in
scripting.

The *list* commands will display the comment from the database or xattr (as
determined by the config file or as specified with **-d**/**-x** options). 
If the database and xattr comments differ, the
comment will be terminated by a '\*' character. 
The **-H** option displays the history of comments from the database.

The output of the *list* commands can be in .json format (**-j**) , 
and can optionally display the comment creator and the comment date (**-v**)

The *create* commands will attempt to store the file comments in _both_ 
the xattr of the file, and in the database. 
If either of these fail, they fail silently. Use the **-c** to create a comment, 
use **-a** to append to an existing comment, and **-e** to erase a comment.

The *filecopy* commands will copy/move the file to a destination, and 
preserve the file comments. [See notes below about LIMITATIONS]

The *cleanup* commands can clean up the history of comments in the database.

# LIMITATIONS

The file comments are located in two locations: a database, and in the xattr 
properties of the file. Each of these storage locations has its own benefits 
and limitations:

## xattr

Because _xattr_ comments are bound to the filesystem, other command line tools 
may be used to manage them. Comments stored in the _xattr_ properties can be 
copied/moved with the file, if you
use the correct options for _cp_. The _mv_ utility automatically preserves _xattr_.
Other programs can also be coerced into perserving _xattr_ properties:

* rsync
* tar
* mksquashfs

Not all file systems support _xattr_ properties (vfat/exfat does not). 

The current implementation of _sshfs_ and _scp_ do not support the copy of 
_xattr_ properties. If you want to copy files to a remote machine and 
include the _xattr_ comments, use _rsync_ with the _-X_ option. Or _tar_ of course.

Some editing apps (like _vim_) will create a new file when saving the data, 
which orphans the _xattr_ comments. For these apps, use the _database_ system.

Of course, once you start to manipulate _xattr_ comments outside of the **dirnotes**
programs, the _xattr_ comments will become out of sync with the _database_ comments.


## database

Comments stored in the database work for all filesystem types 
(including vfat/exfat/sshfs)
Comments are usually stored in a _per user_ database; thus another user on 
the same system will not see these comments.

Files are indexed by their complete path name. Removable filesystems should be
mounted in a consistent way, so that the complete path name is reproducable.

If you are using _sshfs_, you can use the **dirnotes** programs to copy a file, 
and the database comments will work properly.

Comments stored in the database do not travel with the files when
they are moved or copied outside of using the **dirnotes**-family of tools.


# OPTIONS
  
**-a**
: append to a comment on a file

**-c**
: add a comment to a file

**-C**
: attempt to copy the file(s) and comments to a destination; if multiple files are copied, the destination must be a directory; the last argument is the destination

**-d**
: use database comments as the primary source; cannot be used with **-x**

**-D**
: print debugging information

**-e**
: erase the comments on the file(s)

**-h** **--help**
: display help and usage

**-H**
: output the history of comments for the file(s)

**-j**
: output (to stdout) the file comment in .json format 

**-l**
: list all files, including those without _dirnotes_ comments

**-M**
: move the file(s) and comments to the destination; if multiple files are moved, the destination must be a directory

**-n**
: output only the comment; this may be useful for scripting

**-v**
: list full path names, also include the comment author and date in the output

**-V** **--version**
: display version number

**-x**
: use xattr comments as the primary source; connot be used with **-d**

**-y**
: allow file overwrite for **-C** copy or **-M** move

**-z**
: erase history comments associated with the file(s); keep the current comment

**-Z**
: erase all the historic comments in the user's database

# EXAMPLES

To display the comment for a file:

> <code>$ dirnotes-cli car_notes.txt</code>  
> <code>car_notes.txt: 'notes on the car repair'</code>

To extract _only_ the comment from a file, use the _-n_ option (useful for scripts):

> <code>$ dirnotes-cli -n car_notes.txt</code>    
> <code>notes on the car repair</code>

Or, in json format:

> <code>$ dirnotes-cli -j car_notes.txt</code>  
> <code>[{"file": "car_notes.txt", "comment": "notes on the car repair"}]</code>

[NOTE: with json output, consider piping the results to <code>jq</code> for
pretty-print]

To append to a comment:

> <code>$ dirnotes-cli -a 'first quote: \$1,400' car_notes.txt</code>  
> <code>$ dirnotes-cli car_notes.txt</code>  
> <code>car_notes.txt: 'notes on the car repair; first quote: $1,400'</code>


# SEE ALSO

dirnotes, dirnotes-tui, dirnotes-cli

The **dirnotes** program provides a GUI for accessing the files descriptions.

The **dirnotes-tui** program provides a **curses** based (terminal) access to the descriptions.

The **dirnotes-cli** program provides command line access, and can be scripted.

# CONFIGURATION FILE

By default, the file **~/.dirnotes.conf** will be used to load the user's config. There are
three attributes described in that file that are important:

> * xattr_tag  [default: user.xdg.comment]   
> * database   [default: ~/.local/share/dirnotes/dirnotes.db]  
> * start_mode [default: xattr]  

The default location for the config file is ~/.config/dirnotes/dirnotes.conf


