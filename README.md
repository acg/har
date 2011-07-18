# har: fast, random access filesystem archives #

The har(1) program is a little hack aimed at a [problem with tar archives](http://en.wikipedia.org/wiki/Tar_%28file_format%29#Random_access).

Like tar, har bundles a directory in the filesystem into a single archive file. Unlike tar, any file, directory listing or attribute can be extracted from a large har archive in constant time, using constant memory, without waiting for wasteful disk reads. This is possible because a har archive is just a [cdb](http://cr.yp.to/cdb.html).

A har archive is also referred to as a "harball."

The name "har" comes from "Hashed ARchive".

The inspiration for har was Matthew Story's [d2cdb](https://github.com/matthewstory/d2cdb). Some of the code in har bears a striking resemblance to d2cdb, and they both happen to be [54 line shell scripts](http://54lines.com/). Let's hear it for Matt. Cheerio, my good man.

## The problem with tar ##

From the [wikipedia entry on the tar file format](http://en.wikipedia.org/wiki/Tar_%28file_format%29#Random_access):

> Another weakness of the tar format compared to other archive formats is that there is no centralized location for the information about the contents of the file (a "table of contents" of sorts). So to list the names of the files that are in the archive, one must read through the entire archive and look for places where files start. Also, to extract one small file from the archive, instead of being able to lookup the offset in a table and go directly to that location, like other archive formats, with tar, one has to read through the entire archive, looking for the place where the desired file starts. For large tar archives, this causes a big performance penalty, making tar archives unsuitable for situations that often require random access of individual files.

> The possible reason for not using a centralized location of information is rooted in the fact that tar was originally meant for tapes, which are bad at random access anyway: if the TOC were at the start of the archive, creating it would mean to first calculate all the positions of all files, which either needs doubled work, a big cache, or rewinding the tape after writing everything to write the TOC. On the other hand, if the TOC were at the end-of-file (as is the case with ZIP files, for example), reading the TOC would require that the tape be wound to the end, also taking up time and degrading the tape by excessive wear and tear. Compression further complicates matters, as calculating compressed positions for a TOC at the start would need compression of everything before writing the TOC, a TOC with uncompressed positions is not really useful (since you have to decompress everything anyway to get the right positions) and decompressing a TOC at the end of the file might require decompressing the whole file anyway, too.

## Creating a har Archive ##

    har dir file.har

## Working with har Archives ##

Until somebody writes unhar(1), here are some recipes for working with harballs.

    # Listing all paths in a harball:

    cdb -l -m file.har | sed -ne 's#^f/##p'


    # Extracting the contents of a file, or extracting a directory listing:

    cdb -q file.har f/path/to/member


    # Get the permissions of a file or directory (other metadata keys are "type", "size", "uid", "mtime"):

    cdb -q file.har "`printf 'm/path/to/member\0perms'`"


    # Dump all the contents of a harball (files + metadata):

    cdb -d file.har

## Issues and Todo Items ##

* Requires [tinycdb](http://www.corpit.ru/mjt/tinycdb.html).
* Only works with linux currently. The GNU stat(1) invocation hasn't been ported to BSD or Mac OS X yet.
* The unhar(1) program hasn't been written yet. ;)

