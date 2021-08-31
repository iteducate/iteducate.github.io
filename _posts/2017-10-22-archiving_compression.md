---

layout: single
title:  "Archiving and Compression"
date:   2017-10-23 12:20:00 +0300
tags: archiving compression
categories: linux
comments: true
toc: true
toc_label: "Содержание"
toc_sticky: true

---

<p><strong>File archiving</strong> is used when one or more files need to be transmitted or stored as efficiently as possible.  There are two aspects to this:</p>
<ul type="disc" class="">
<li>
<strong>Archiving</strong> – Combining multiple files into one, which eliminates the overhead in individual files and makes it easier to transmit</li>
<li>
<strong>Compressing</strong> – Making the files smaller by removing redundant information</li>
</ul>

# 1.1 Introduction

<p>You can archive multiple files into a single archive and then compress it, or you can compress an individual file.  The former is still referred to as archiving, while the latter is just called compression.  When you take an archive, decompress it and extract one or more files, you are <var>un-archiving</var> it.</p>

<p>Even though disk space is relatively cheap, archiving and compression still has value:</p>
<ul type="disc" class="">
<li>If you want to make a large number of files available, such as the source code to an application or a collection of documents, it is easier for people to download a compressed archive than it is to download individual files.</li>
<li>Log files have a habit of filling disks so it is helpful to split them by date and compress older versions.</li>
<li>When you back up directories, it is easier to keep them all in one archive than it is to version each file.</li>
<li>Some streaming devices such as tapes perform better if you’re sending a stream of data rather than individual files.</li>
<li>It can often be faster to compress a file before you send it to a tape drive or over a slower network and decompress it on the other end than it would be to send it uncompressed.</li>
</ul>

<p>As a Linux administrator, you should become familiar with the tools for archiving and compressing files.</p>

# 1.2 Compressing files
<p><var>Compressing</var> files makes them smaller by removing duplication from a file and storing it such that the file can be restored.  A file with human readable text might have frequently used words replaced by something smaller, or an image with a solid background might represent patches of that color by a code.  You generally don’t use the compressed version of the file, instead you decompress it before use.  The <var>compression algorithm</var> is a procedure the computer does to encode the original file, and as a result make it smaller.  Computer scientists research these algorithms and come up with better ones that can work faster or make the input file smaller.</p>

<p>When talking about compression, there are two types:</p>
<ul type="disc" class="">
<li>
<strong>Lossless</strong>:  No information is removed from the file.  Compressing a file and decompressing it leaves something identical to the original.</li>
<li>
<strong>Lossy</strong>:  Information might be removed from the file as it is compressed so that uncompressing a file will result in a file that is slightly different than the original.  For instance, an image with two subtly different shades of green might be made smaller by treating those two shades as the same.  Often, the eye can’t pick out the difference anyway.</li>
</ul>

<p>Generally human eyes and ears don’t notice slight imperfections in pictures and audio, especially as they are displayed on a monitor or played over speakers.  Lossy compression often benefits media because it results in smaller file sizes and people can’t tell the difference between the original and the version with the changed data.  For things that must remain intact, such as documents, logs, and software, you need lossless compression.</p>

<p>Most image formats, such as GIF, PNG, and JPEG, implement some kind of lossy compression.  You can generally decide how much quality you want to preserve.  A lower quality results in a smaller file, but after decompression you may notice artifacts such as rough edges or discolorations.  High quality will look much like the original image, but the file size will be closer to the original.</p>

<p>Compressing an already compressed file will not make it smaller.  This is often forgotten when it comes to images, since they are already stored in a compressed format.  With lossless compression, this multiple compression is not a problem, but if you compress and decompress a file several times using a lossy algorithm you will eventually have something that is unrecognizable.</p>

<p>Linux provides several tools to compress files, the most common is <code>gzip</code>.  Here we show a log file before and after compression.</p>

{% highlight console %}
bob:tmp $ ls -l access_log*
-rw-r--r-- 1 sean sean 372063 Oct 11 21:24 access_log
bob:tmp $ gzip access_log
bob:tmp $ ls -l access_log*
-rw-r--r-- 1 sean sean 26080 Oct 11 21:24 access_log.gz
{% endhighlight %}

<p>In the example above, there is a file called <code class="console">access_log</code> that is 372,063 bytes.  The file is compressed by invoking the <code>gzip</code> command with the name of the file as the only argument.  After that command completes, the original file is gone and a compressed version with a file extension of <code class="console">.gz</code> is left in its place.  The file size is now 26,080 bytes, giving a compression ratio of about 14:1, which is common with log files.</p>

<p>Gzip will give you this information if you ask, by using the <code>–l</code> parameter, as shown here:</p>

{% highlight console %}
bob:tmp $ gzip -l access_log.gz
      compressed     uncompressed  ratio uncompressed_name
           26080           372063  93.0% access_log
{% endhighlight %}

<p>Here, you can see that the compression ratio is given as 93%, which is the inverse of the 14:1 ratio, i.e. 13/14.  Additionally, when the file is decompressed it will be called <code class="console">access_log</code>. </p>

{% highlight console %}
bob:tmp $ gunzip access_log.gz
bob:tmp $ ls -l access_log*
-rw-r--r-- 1 sean sean 372063 Oct 11 21:24 access_log
{% endhighlight %}

<p>The opposite of the <code>gzip</code> command is <code>gunzip</code>.  Alternatively, <code>gzip –d</code> does the same thing (<code>gunzip</code> is just a script that calls <code>gzip</code> with the right parameters). After <code>gunzip</code> does its work you can see that the <code class="console">access_log</code> file is back to its original size.</p>

<p>Gzip can also act as a filter which means it doesn’t read or write anything to disk but instead receives data through an input channel and writes it out to an output channel.  You’ll learn more about how this works in the next chapter, so the next example just gives you an idea of what you can do by being able to compress a stream.</p>

{% highlight console %}
bob:tmp $ mysqldump -A | gzip &gt; database_backup.gz
bob:tmp $ gzip -l database_backup.gz
         compressed        uncompressed  ratio uncompressed_name
              76866             1028003  92.5% database_backup
{% endhighlight %}

<p>The <code>mysqldump –A</code> command outputs the contents of the local MySQL databases to the console. The <code class="console">|</code> character (pipe) says “redirect the output of the previous command into the input of the next one”. The program to receive the output is <code>gzip</code>, which recognizes that no filenames were given so it should operate in pipe mode. Finally, the <code>&gt; database_backup.gz</code> means “redirect the output of the previous command into a file called <code class="console">database_backup.gz</code>. Inspecting this file with <code>gzip –l</code> shows that the compressed version is 7.5% of the size of the original, with the added benefit that the larger file never had to be written to disk.</p>

<p>There is another pair of commands that operate virtually identically to <code>gzip</code> and <code>gunzip</code>. These are <code>bzip2</code> and <code>bunzip2</code>. The <code>bzip</code> utilities use a different compression algorithm (called Burrows-Wheeler block sorting, versus Lempel-Ziv coding used by <code>gzip</code>) that can compress files smaller than <code>gzip</code> at the expense of more CPU time. You can recognize these files because they have a <code class="console">.bz</code> or <code class="console">bz2</code> extension instead of <code class="console">.gz</code>.</p>


# 1.3 Archiving Files
<p>If you had several files to send to someone, you could compress each one individually.  You would have a smaller amount of data in total than if you sent uncompressed files, but you would still have to deal with many files at one time.</p>

<p><var>Archiving</var> is the solution to this problem.  The traditional UNIX utility to archive files is called <code>tar</code>, which is a short form of TApe aRchive.  <var>Tar</var> was used to stream many files to a tape for backups or file transfer.  Tar takes in several files and creates a single output file that can be split up again into the original files on the other end of the transmission.</p>

<p>Tar has 3 modes you will want to be familiar with:</p>
<ul type="disc" class="">
<li>
<strong>Create</strong>:  make a new archive out of a series of files</li>
<li>
<strong>Extract</strong>:  pull one or more files out of an archive</li>
<li>
<strong>List</strong>:  show the contents of the archive without extracting</li>
</ul>

<p>Remembering the modes is key to figuring out the command line options necessary to do what you want.  In addition to the mode, you will also want to make sure you remember where to specify the name of the archive, as you may be entering multiple file names on a command line.</p>

<p>Here, we show a <var>tar file</var>, also called a tarball, being created from multiple access logs.</p>

{% highlight console %}
bob:tmp $ tar -cf access_logs.tar access_log*
bob:tmp $ ls -l access_logs.tar
-rw-rw-r-- 1 sean sean 542720 Oct 12 21:42 access_logs.tar
{% endhighlight %}

<p>Creating an archive requires two named options.  The first, <code>c</code>, specifies the mode. The second, <code>f</code>, tells tar to expect a file name as the next argument. The first argument in the example above creates an archive called <code class="console">access_logs.tar</code>.  The remaining arguments are all taken to be input file names, either as a wildcard, a list of files, or both.  In this example, we use the wildcard option to include all files that begin with <code class="console">access_log</code>.</p>

<p>The example above does a long directory listing of the created file.  The final size is 542,720 bytes which is slightly larger than the input files. Tarballs can be compressed for easier transport, either by gzipping the archive or by having <code>tar</code> do it with the <code>z</code> flag as follows:</p>

{% highlight console %}
bob:tmp $ tar -czf access_logs.tar.gz  access_log*
bob:tmp $ ls -l access_logs.tar.gz
-rw-rw-r-- 1 sean sean 46229 Oct 12 21:50 access_logs.tar.gz
bob:tmp $ gzip -l access_logs.tar.gz
         compressed        uncompressed  ratio uncompressed_name
              46229              542720  91.5% access_logs.tar
{% endhighlight %}

<p>The example above shows the same command as the prior example, but with the addition of the <code>z</code> parameter.  The output is much smaller than the tarball itself, and the resulting file is compatible with <code>gzip</code>. You can see from the last command that the uncompressed file is the same size as it would be if you tarred it in a separate step.</p>

<p>While UNIX doesn’t treat file extensions specially, the convention is to use <code class="console">.tar</code> for <code>tar</code> files, and <code class="console">.tar.gz</code> or <code class="console">.tgz</code> for compressed <code>tar</code> files. You can use <code>bzip2</code> instead of <code>gzip</code> by substituting the letter <code>z</code> for <code>j</code> and using <code class="console">.tar.bz2</code>, <code class="console">.tbz</code>, or <code class="console">.tbz2</code> for a file extension (e.g. <code>tar –cjf file.tbz access_log*</code>).</p>

<p>Given a <code>tar</code> file, compressed or not, you can see what’s in it by using the <code>t</code> command:</p>

{% highlight console %}
bob:tmp $ tar -tjf access_logs.tbz
logs/
logs/access_log.3
logs/access_log.1
logs/access_log.4
logs/access_log
logs/access_log.2
{% endhighlight %}

<p>This example uses 3 options:</p>
<ul type="disc" class="">
<li>
<code>t</code>: list files in the archive</li>
<li>
<code>j</code>:  decompress with <code>bzip2</code> before reading</li>
<li>
<code>f</code>:  operate on the given filename <code class="console">access_logs.tbz</code>
</li>
</ul>

<p>The contents of the compressed archive are then displayed.  You can see that a directory was prefixed to the files.  Tar will recurse into subdirectories automatically when compressing and will store the path info inside the archive.</p>

<p>Just to show that this file is still nothing special, we will list the contents of the file in two steps using a pipeline.</p>

{% highlight console %}
bob:tmp $ bunzip2 -c access_logs.tbz | tar -t
logs/
logs/access_log.3
logs/access_log.1
logs/access_log.4
logs/access_log
logs/access_log.2
{% endhighlight %}

<p>The left side of the pipeline is <code>bunzip –c access_logs.tbz</code>, which decompresses the file, but the <code>-c</code> option sends the output to the screen. The output is redirected to <code>tar –t</code>. If you don’t specify a file with <code>–f</code> then <code>tar</code> will read from the standard input, which in this case is the uncompressed file.</p>

<p>Finally you can extract the archive with the <code>–x</code> flag:</p>

{% highlight console %}
bob:tmp $ tar -xjf access_logs.tbz
bob:tmp $ ls -l
total 36
-rw-rw-r-- 1 sean sean 30043 Oct 14 13:27 access_logs.tbz
drwxrwxr-x 2 sean sean  4096 Oct 14 13:26 logs
bob:tmp $ ls -l logs
total 536
-rw-r--r-- 1 sean sean 372063 Oct 11 21:24 access_log
-rw-r--r-- 1 sean sean    362 Oct 12 21:41 access_log.1
-rw-r--r-- 1 sean sean 153813 Oct 12 21:41 access_log.2
-rw-r--r-- 1 sean sean   1136 Oct 12 21:41 access_log.3
-rw-r--r-- 1 sean sean    784 Oct 12 21:41 access_log.4
{% endhighlight %}

<p>The example above uses the similar pattern as before, specifying the operation (eXtract), the compression (the <code>j</code> flag, meaning <code>bzip2</code>), and a file name (<code>-f</code> <code class="console">access_logs.tbz</code>).  The original file is untouched and the new <strong>logs</strong> directory is created. Inside the directory are the files.</p>

<p>Add the <code>–v</code> flag and you will get verbose output of the files processed. This is helpful so you can see what’s happening:</p>

{% highlight console %}
bob:tmp $ tar -xjvf access_logs.tbz
logs/
logs/access_log.3
logs/access_log.1
logs/access_log.4
logs/access_log
logs/access_log.2
{% endhighlight %}

<p>It is important to keep the <code>–f</code> flag at the end, as <code>tar</code> assumes whatever follows it is a filename.  In the next example, the <code>f</code> and <code>v</code> flags were transposed, leading to <code>tar</code> interpreting the command as an operation on a file called "<code class="console">v</code>" (the relevant message is in italics.)</p>

{% highlight console %}
bob:tmp $ tar -xjfv access_logs.tbz
tar (child): v: Cannot open: No such file or directory
tar (child): Error is not recoverable: exiting now
tar: Child returned status 2
tar: Error is not recoverable: exiting now
{% endhighlight %}

<p>If you only want some files out of the archive you can add their names to the end of the command, but by default they must match the name in the archive exactly or use a pattern:</p>

{% highlight console %}
bob:tmp $ tar -xjvf access_logs.tbz logs/access_log
logs/access_log
{% endhighlight %}

<p>The example above shows the same archive as before, but extracting only the <code class="console">logs/access_log</code> file. The output of the command (as verbose mode was requested with the <code>v</code> flag) shows only the one file has been extracted.</p>

<p>Tar has many more features, such as the ability to use patterns when extracting files, excluding certain files, or outputting the extracted files to the screen instead of disk. The documentation for <code>tar</code> has in depth information.</p>

# 1.4 ZIP files
<p>The de facto archiving utility in the Microsoft world is the ZIP file.  It is not as prevalent in Linux but is well supported by the <code>zip</code> and <code>unzip</code> commands.  With <code>tar</code> and <code>gzip</code>/<code>gunzip</code> the same commands and options can be used to do the creation and extraction, but this is not the case with <code>zip</code>.  The same option has different meanings for the two different commands.</p>

<p>The default mode of <code>zip</code> is to add files to an archive and compress it.</p>

{% highlight console %}
bob:tmp $ zip logs.zip logs/*
  adding: logs/access_log (deflated 93%)
  adding: logs/access_log.1 (deflated 62%)
  adding: logs/access_log.2 (deflated 88%)
  adding: logs/access_log.3 (deflated 73%)
  adding: logs/access_log.4 (deflated 72%)
{% endhighlight %}

<p>The first argument in the example above is the name of the archive to be operated on, in this case it is <code class="console">logs.zip</code>.  After that, is a list of files to be added. The output shows the files and the compression ratio. It should be noted that <code>tar</code> requires the <code>–f</code> option to indicate a filename is being passed, while <code>zip</code> and <code>unzip</code> require a filename and therefore don’t need you to explicitly say a filename is being passed.</p>

<p>Zip will not recurse into subdirectories by default, which is different behavior than <code>tar</code>. That is, merely adding <code class="console">logs</code> instead of <code class="console">logs/*</code> will only add the empty directory and not the files under it. If you want <code>tar</code> like behavior, you must use the <code>–r</code> command to indicate recursion is to be used:</p>


{% highlight console %}
bob:tmp $ zip -r logs.zip logs
  adding: logs/ (stored 0%)
  adding: logs/access_log.3 (deflated 73%)
  adding: logs/access_log.1 (deflated 62%)
  adding: logs/access_log.4 (deflated 72%)
  adding: logs/access_log (deflated 93%)
  adding: logs/access_log.2 (deflated 88%)
{% endhighlight %}

<p>In the example above, all files under the logs directory are added because it uses the <code>–r</code> option.  The first line of output indicates that a directory was added to the archive, but otherwise the output is similar to the previous example.</p>

<p>Listing files in the <code>zip</code> is done by the <code>unzip</code> command and the <code>–l</code> option (list):</p>

{% highlight console %}
bob:tmp $ unzip -l logs.zip
Archive:  logs.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  10-14-2013 14:07   logs/
     1136  10-14-2013 14:07   logs/access_log.3
      362  10-14-2013 14:07   logs/access_log.1
      784  10-14-2013 14:07   logs/access_log.4
    90703  10-14-2013 14:07   logs/access_log
   153813  10-14-2013 14:07   logs/access_log.2
---------                     -------
   246798                     6 files
{% endhighlight %}

<p>Extracting the files is just like creating the archive, as the default operation is to extract:</p>



{% highlight console %}
bob:tmp $ unzip logs.zip
Archive:  logs.zip
   creating: logs/
  inflating: logs/access_log.3
  inflating: logs/access_log.1
  inflating: logs/access_log.4
  inflating: logs/access_log
  inflating: logs/access_log.2
{% endhighlight %}

<p>Here, we extract all the files in the archive to the current directory.  Just like <code>tar</code>, you can pass filenames on the command line:</p>

{% highlight console %}
bob:tmp $ unzip logs.zip access_log
Archive:  logs.zip
caution: filename not matched:  access_log
bob:tmp $ unzip logs.zip logs/access_log
Archive:  logs.zip
  inflating: logs/access_log
bob:tmp $ unzip logs.zip logs/access_log.*
Archive:  logs.zip
  inflating: logs/access_log.3
  inflating: logs/access_log.1
  inflating: logs/access_log.4
  inflating: logs/access_log.2
{% endhighlight %}

<p>The example above shows three different attempts to extract a file.  First, just the name of the file is passed without the directory component.  Like <code>tar</code>, the file is not matched.</p>

<p>The second attempt passes the directory component along with the filename, which extracts just that file.</p>


<p>The third version uses a wildcard, which extracts the 4 files matching the pattern, just like <code>tar</code>.</p>


<p>The <code>zip</code> and <code>unzip</code> man pages describe the other things you can do with these tools, such as replace files within the archive, use different compression levels, and even use encryption.</p>

*Все материалы взяты с официального курса <i class="fa fa-link"></i> [NDG Linux Essential](https://www.netacad.com/ru/courses/ndg-linux-essentials/)*
