# Dirnotes

Table of Contents
----------------

  * [SYNOPSIS](#synopsis)
  * [USAGE](#usage)
  * [INSTALLATION](#installation)
    * [CONFIG FILE](#config-file)
  * [LIMITATIONS](#limitations)
  * [PROGRAMMER NOTES](#programmer-notes)
    * [MacOS](#macos)
  * [DEVELOPMENT STATUS](#development-status)
  * [QUESTIONS](#questions)

## SYNOPSIS

The **dirnotes** family of apps allows you to add a descriptive comment to a file. 
The descriptions are stored in two places:

 * in the _xattr_ properties of the file
 * in a _database_ located in the user's home directory

[The [MacOS](#macos) stores its comments in a similar way.]

The <code>**dirnotes**</code> app is a GUI app, using the Qt5 framework. 
At startup, it displays the contents of the current directory, and the 
comments associated with any of the files or directories. 
Simple mouse clicks allow you to add or edit comments, tunnel down 
into directories, or rise up the file system. 
You can copy or move files (_with_ comments), and 
choose whether the _xattr_ or _database_ version of the comments 
have display priority. 

The <code>**dirnotes-tui**</code> is a very similar app, but uses the 
_curses_ framework to display its activity in a terminal window. 
This can be handy if you have to work across a network, 
or if terminal apps are your preference.

The <code>**dirnotes-cli**</code> is a command line tool, 
which may be handy for scripting. This all can also do maintenance on the database.

## USAGE

The <code>**dirnotes**</code> program displays usage and keystoke info 
when you press _F1_. The <code>**dirnotes-tui**</code> program display 
onscreen usage when you press the 'h' key, or _F1_. 
The <code>**dirnotes-cli**</code> program has a man page.

In short, you navigate <code>**dirnotes**</code> and 
<code>**dirnotes-tui**</code> by using the up/down arrow keys, 
<enter> to enter into a directory. 
The **-tui** version accepts _e_ for edit, _s_ for sort, _M_ to change 
between xattr/database priority.

The **<code>dirnotes-cli</code>** has options 
for _-l_ list and _-c_ create a comment. See also <code>dirnotes-cli.1</code> 
man page.

All three apps in the **dirnotes** family have the ability to 
copy/move files from the current directory, keeping the comments intact. 
All three apps have the **-h** option which shows command line usage. 
 
## INSTALLATION

Each of the 3 apps in the family is self contained. 
The <code>**dirnotes**</code> app requires _Python3_ and the _Qt5_ framework. 
The <code>**dirnotes-tui**</code> and <code>**dirnotes-cli**</code> apps 
simply require _Python3_.

Simply copy the 3 executable files into your path, to <code>\~/.local/bin/</code> for example. 

    cp dirnotes dirnotes-tui dirnotes-cli \~/.local/bin/
 
For a better GUI experience, copy 
<code>dirnotes.desktop</code> to <code>\~/.local/share/applications</code> and 
<code>dirnotes.xpm</code> to <code>\~/.local/share/icons/</code> 

### CONFIG FILE

By default, the file **~/.config/dirnotes/dirnotes.conf** will be used to 
load the user's config. 
This is a JSON file, with three attributes that are important:

* xattr_tag (default: <code>usrr.xdg.comment</code>)
* database (default: <code>~/.local/share/dirnotes/dirnotes.db</code>, sensible alt: <code>/var/lib/dirnotes.db</code>) 
* start_mode (_xattr_ or _db_ display priority)

The _config_file_ should be auto-generated the first time one of 
the **dirnotes** apps is run.

## LIMITATIONS 

The file comments are located in two locations: a database, and in the 
xattr properties of the file. Each of these storage locations has its 
own benefits and limitations. These can be summed up: **_xattr_** comments
follow the iNode, **_database_** comments follow the file name.


### xattr

Comments stored in the xattr properties can be copied/moved with the file, if you
use the correct options: <code>**cp&nbsp;-p&nbsp;_src&nbsp;dest_**</code>. 
The <code>**mv**</code> utility 
automatically preserves _xattr_. Other programs can also be coerced into 
perserving _xattr_ properties:

* <code>**rsync**</code>
* <code>**tar**</code>
* <code>**mksquashfs**</code>

Not all file systems support xattr properties (vfat/exfat does not).

_xattr_ comments may only be applied to files for which the user has _write_ permission.

The current implementation of <code>**sshfs**</code> and <code>**scp**</code> 
do not support copying of _xattr_ properties. **Dropbox** type mounts are 
unlikely to support _xattr_ comments.
If you want to copy files to a remote machine and include the _xattr_ comments, use <code>**rsync**</code> with the _-X_ option. Or <code>**tar**</code>.

Some editing apps (like _vim_) will create a new file when saving the data, which orphans the _xattr_ comments. For these apps, use the _database_ system.  

Removable disk devices (usb sticks) which are formatted with a Linux-based filesystem (ext2/3/4, btrfs, xfs, zfs) will carry the _xattr_ comments embedded in the filesystem metadata, and are portable to anther computer.

## database

Comments stored in the database work for all filesystem types 
(including vfat/exfat/sshfs)
 
The _database_ comments that are stored in 
<code>~/.local/share/dirnotes/dirnotes.db</code> are inherently associated 
with a single user. If the _database_ is located in 
<code>/var/lib/dirnotes.db</code>, it can be shared by all the users in the system.

Files are indexed by their complete path name. Removable filesystems should be
mounted in a consistent way, so that the complete path name is reproducable. 
Symlinks are _not_
dereferenced, so they may have comments bound to them.

Comments stored in the database do not travel with the files when
they are moved or copied, unless using the **dirnotes** family of tools. 

_Database_ comments may be applied to any visible file, _even if they are readonly_. 
For exmple, comments may be attached to the files in <code>/usr/bin/\*</code> even though they are probably owned by _root_.

## PROGRAMMER NOTES

Instead of an API, here is how you can get directly at the underlying comment data. 
If you intend to use the **dirnotes** apps, 
try to keep the two versions of the comments in sync.

  * xattr

Use the commands  

        xattr -l [filename]  

to display the comments/author/date on a file. For example:  

        $ xattr -l /etc/fstab
        user.xdg.comment: controls the default mount bindings
        user.xdg.comment.author: patb
        user.xdg.comment.date: 2022-09-29 08:07:42

The other options on the **xattr** command line tool allow you to 
write (*xattr -w*) or delete (*xattr -d*) the comments.

  * database

The comments are stored in an Sqlite3 database, usually 
located at "~/.local/share/dirnotes/dirnotes.db". 
The database itself is contained within that file, and its schema is this:

~~~~
    CREATE TABLE dirnotes (name TEXT, date DATETIME, size INTEGER, comment TEXT, comment_date DATETIME, author TEXT)
~~~~


| field | usage | example |
| -------- | -------- | -------- |
| name   |       the long filename, using python's os.path.abspath() |   /home/patb/projects/dirnotes/README.md | 
|date    |      the file's modified date  |                             2020-01-13 09:25:40 |
|size   |      the byte count of the file       |                      145  |
|comment     |  a utf-8 string |                                         the readme for the GIT page | 
|comment_date | the date of the comment itself   |                      2020-10-03 22:30:19 |
|author     |   the system name of the user who created the comment  |  patb | 


The _date_ and _size_ fields reflect the file's modification date and size 
at the time of the last edit of the file comment, which is stored in _comment_date_.

As comments are editted or appended, new records are added to the database. 
Older records are are not purged. This gives you a history of the comments, 
but it means that fetching the most recent comment involves something like

~~~~
  SELECT * FROM dirnotes WHERE name=? ORDER BY comment_date DESC
~~~~

and just fetch the first record.

The database is created the first time one of the **dirnotes** apps is run.

  * misc

The <code>**dirnotes**</code> gui app has a desktop icon built into the code. 
There is not need for an external .icon file, but there is an .xpm file included
in the project, which can be copied to ~/.local/share/icons/

### comment date & author

The <code>copy()/move()</code> methods that are built into the **dirnotes** library
will ask the operating system to copy/move the file _with_ xattr intact. 
The entry in the database is created _at the time of invocation_. 
Therefore, the xattrs will reflect the original author+date on the comments, 
whereas the database version is updated on each copy/move; 
the dirnotes-comments details will therefor diverge over time.

There was _no_ consideration given for language translation. Email [me](mail:patb@pbeirne.com) if you want this, or can help.

All these apps only accomadate a single line comment. An embedded newline will 
cause unpredictable behaviour. 

### MacOS

The **MacOS** inherently supports file comments. The Finder app manages most of the user activity. It handles file comments in a similar manner to **Dirnotes**. Comments are stored in two places:

  * in the xattr properties of the file  
    * using a different xattr-tag (<code>com.apple.metadata:kMDItemFinderComment</code>) 
    * the comment string is wrapped in a <code>pList</code>
  * in a database located in each directory  
    * in the .DS-Store file
 
The user can examine the file comments by opening the GetInfo dialog, and scrolling down to "Comment"

If the Finder is used to copy/move files, the comments are moved properly 
to both destinations. If you use the os to copy/move the files, 
you can ask that the xattr properties get moved, but the .DS-Store 
file will not be updated. That means the Finder will not see 
file comments on the destination file.

**MacOS** has AppleScript, by which you can ask the Finder to perform the file copy/move. In this case, the comments are moved properly.

## DEVELOPMENT STATUS

Each app is a standalone file. That means there is a lot of redundancy between 
the three apps. And there _may_ be some inconsistency.

2022-10-04
: All three apps are functioning and usable.  
  The _config_file_ is fully implemented.  
  Themes are not implemented.   
  Comments are intended to be _utf-8_, but are _strings_ in some places.  
  MacOS code is not written yet.   
  The _help_ dialogs in <code>**dirnotes-tui**</code> are meagre.    
  The _qt-gui_ app is working pretty well.  


## QUESTIONS:

There are several open-ended questions that need to be answered. 
Does anyone have an opinion?

1. How important is multi-line comments?

2. Is it ok to put the config file and database file buried in ~/.config and ~/.local? 

    These directories exist on computers with a gui/windowing system installed, 
    but don't neccessarily exist on headless servers. 
    Perhaps the default locations should be in the user directory? 
    (~/.dirnotes.conf and ~/.dirnotes.db)

3. Who needs translations?

4. Does anybody have a better edit-window for CURSES?

5. What about storing the database _per directory_. That's what MacOS does (the .DS_Store file)

    - PRO  
        - copy of an _entire directory_ will transport the database with it  
        - database stored on a remote directory (sshfs, nfs, etc) will work properly  
        - the database only needs to store the basename, not the entire path name
        - if a removable drive is remounted on a different mount point, it'll still work

    - CON  
        - won't work on directories where the user doesn't have write permission  
        - so, for example, the user can't add comments to /usr/bin/\*

6. Is anyone interested in the MacOS version?

TTD: add a screenshot of the gui and tui versions; add to the examples for dirnotes-cli -j where it pipes to <code>jq</code>
