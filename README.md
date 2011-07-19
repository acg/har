# har: fast, random access filesystem archives #

The har(1) and unhar(1) programs are a little hack aimed at a [problem with tar archives](http://en.wikipedia.org/wiki/Tar_%28file_format%29#Random_access).

Like tar, har bundles a directory tree into a single archive file. Unlike tar, any file, directory listing or attribute can be extracted from a large .har archive in constant time, using constant memory, without waiting for wasteful disk reads. This is possible because a .har archive is just a [cdb](http://cr.yp.to/cdb.html).

A .har archive is also referred to as a "harball."

The name "HAR" stands for "H-ashed AR-chive".

The inspiration for har was Matthew Story's [d2cdb](https://github.com/matthewstory/d2cdb). Some of the code in har and unhar bears a striking resemblance to d2cdb, and they all happen to be [54 line shell scripts](http://54lines.com/). Let's hear it for Matt. Cheerio, my good man.

## The problem with tar ##

From the [wikipedia entry on the tar file format](http://en.wikipedia.org/wiki/Tar_%28file_format%29#Random_access):

> Another weakness of the tar format compared to other archive formats is that there is no centralized location for the information about the contents of the file (a "table of contents" of sorts). So to list the names of the files that are in the archive, one must read through the entire archive and look for places where files start. Also, to extract one small file from the archive, instead of being able to lookup the offset in a table and go directly to that location, like other archive formats, with tar, one has to read through the entire archive, looking for the place where the desired file starts. For large tar archives, this causes a big performance penalty, making tar archives unsuitable for situations that often require random access of individual files.

> The possible reason for not using a centralized location of information is rooted in the fact that tar was originally meant for tapes, which are bad at random access anyway: if the TOC were at the start of the archive, creating it would mean to first calculate all the positions of all files, which either needs doubled work, a big cache, or rewinding the tape after writing everything to write the TOC. On the other hand, if the TOC were at the end-of-file (as is the case with ZIP files, for example), reading the TOC would require that the tape be wound to the end, also taking up time and degrading the tape by excessive wear and tear. Compression further complicates matters, as calculating compressed positions for a TOC at the start would need compression of everything before writing the TOC, a TOC with uncompressed positions is not really useful (since you have to decompress everything anyway to get the right positions) and decompressing a TOC at the end of the file might require decompressing the whole file anyway, too.

## Requirements ##

* [tinycdb](http://www.corpit.ru/mjt/tinycdb.html)
* A POSIX shell

## Creating a .har Archive ##

    har file.har dir1 [ dir2 file3 ... ]

## Extracting a .har Archive ##

    unhar file.har [ dir1 dir2 file3 ... ]

## Working with .har Archives ##

Here are some other recipes for working with harballs.

Listing all paths in a harball:

    cdb -l -m test.har | sed -ne 's#^f/##p' | uniq

To extract the contents of a file to stdout, or print a listing of entries for directory members:

    cdb -q file.har f/path/to/member

Get the permissions of a file or directory (other metadata keys are "type", "size", "uid", "gid", and "mtime"):

    cdb -q file.har "`printf 'm/path/to/member\0perms'`"

Dump the raw contents of a harball to stdout (files + metadata):

    cdb -d file.har

## Benchmarks ##

Benchmarking extraction of a single member from the middle of an uncompressed archive containing 10,000 16k files. Results on my machine:

                elapsed      user   system   #inputs   #outputs   archive size   
    tar xf        9.06s     0.15s    0.69s    641192         32      332810240
    unhar         0.20s     0.04s    0.01s       648         32      331722653

As you can see,

  * unhar is ~50x faster than tar xf for this case
  * both processes are I/O bound...
  * ...but tar xf does ~1000x more IOs than unhar
  * the archives are about the same size (316MB)

This is a totally contrived example. It's easy to come up with examples where tar kicks that crap out of har: smaller archive sizes; fewer members per archive; archive creation, where har is slower because it's hashing keys, among other things.

The point is, for random access to members in an archive, tar just got its ass handed to it by a 54-line shell script. Okay, with the help of about 1.6Kloc of tinycdb C code.

## Issues and Todo Items ##

* Only works with linux currently. The GNU stat(1) invocation is very non-portable, and doesn't work on BSD or Mac OS X yet.
* The largest .har archive that can be created is 2GB. This is a limitation of cdb.
* *SECURITY NOTE* The unhar(1) program doesn't fully validate input yet. We're still at the "hey we just threw something together" stage. Don't use it on untrusted .har files.
* The unhar(1) program doesn't set ownership yet.
* The unhar(1) program has a bug where `unhar file.har path/to/subdir` will create a directory "subdir" in the current directory, instead of creating all the leading components ("path/to") as untar does.
* The unhar(1) program could probably be much faster when extracting many members from the archive. It execs cdb(1) multiple times for every member to be extracted from the archive.

## Authors ##

Alan Grow, Matthew Story  
Copyright (c) 2011  
Published under the BSD License

