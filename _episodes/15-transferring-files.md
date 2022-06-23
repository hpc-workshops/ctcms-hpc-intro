---
title: "Transferring files with remote computers"
teaching: 15
exercises: 15
questions:
- "How do I transfer files to (and from) the cluster?"
objectives:
- "Transfer files to and from a computing cluster."
keypoints:
- "`wget` and `curl -O` download a file from the internet."
- "`scp` and `rsync` transfer files to and from your computer."
---

Performing work on a remote computer is not very useful if we cannot get files
to or from the cluster. There are several options for transferring data between
computing resources using CLI and GUI utilities, a few of which we will cover.

## Download Lesson Files From the Internet

One of the most straightforward ways to download files is to use either `curl`
or `wget`. One of these is usually installed in most Linux shells, on Mac OS
terminal and in GitBash. Any file that can be downloaded in your web browser
through a direct link can be downloaded using `curl` or `wget`. This is a
quick way to download datasets or source code. The syntax for these commands is

* `curl -O https://some/link/to/a/file`
* `wget https://some/link/to/a/file`

Try it out by downloading some material we'll use later on, from a terminal on
your local machine.

```
{{ site.local.prompt }} curl -O {{ site.url }}{{ site.baseurl }}/files/hpc-intro-code.tar.gz
```
{: .language-bash}
or
```
{{ site.local.prompt }} wget {{ site.url }}{{ site.baseurl }}/files/hpc-intro-code.tar.gz
```
{: .language-bash}


> ## `tar.gz`?
>
> This is an archive file format, just like `.zip`, commonly used and supported
> by default on Linux, which is the operating system the majority of HPC
> cluster machines run. You may also see the extension `.tgz`, which is exactly
> the same. We'll talk more about "tarballs" later, since "tar-dot-g-z" is a
> mouthful.
{: .discussion}

## Transferring Single Files and Folders With `scp`

To copy a single file to or from the cluster, we can use `scp` ("secure copy").
The syntax can be a little complex for new users, but we'll break it down.
The `scp` command is a relative of the `ssh` command we used to
access the system, and can use the same public-key authentication
mechanism.

To _upload to_ another computer:

```
{{ site.local.prompt }} scp local_file {{ site.remote.user }}@{{ site.remote.login }}:remote_path
```
{: .language-bash}

Note that everything after the `:` is relative to our home directory on the
remote computer. We can leave it at that if we don't have a more specific
destination in mind.

Upload the lesson material to your remote home directory like so:

```
{{ site.local.prompt }} scp hpc-intro-code.tar.gz {{ site.remote.user }}@{{ site.remote.login }}:
```
{: .language-bash}

> ## Why Not Download on {{ site.remote.name }} Directly?
>
> Most computer clusters are protected from the open internet by a _firewall_.
> This means that the `curl` command will fail, as an address outside the
> firewall is unreachable from the inside. To get around this, run the `curl`
> or `wget` command from your local machine to download the file, then use the
> `scp` command to upload it to the cluster.
>
> Try downloading the file directly. Note that it may well fail, and that's
> OK!
>
> > ## Commands
> >
> > ```
> > {{ site.local.prompt }} ssh {{ site.remote.user }}@{{ site.remote.login }}
> > {{ site.remote.prompt }} curl -O {{ site.url }}{{ site.baseurl }}/files/hpc-intro-code.tar.gz
> > # or
> > {{ site.remote.prompt }} wget {{ site.url }}{{ site.baseurl }}/files/hpc-intro-code.tar.gz
> > ```
> > {: .language-bash}
> {: .solution}
>
> Did it work? If not, what does the terminal output tell you about what
> happened?
{: .discussion}

## Transferring a Directory

If you went ahead and extracted the tarball, don't worry! `scp` can handle
entire directories as well as individual files.

To copy a whole directory, we add the `-r` flag for "**r**ecursive": copy the
item specified, and every item below it, and every item below those... until it
reaches the bottom of the directory tree rooted at the folder name you
provided.

```
{{ site.local.prompt }} scp -r hpc-intro-code {{ site.remote.user }}@{{ site.remote.login }}:~/
```
{: .language-bash}

> ## Caution
>
> For a large directory -- either in size or number of files --
> copying with `-r` can take a long time to complete.
{: .callout}

## What's in a `/`?

When using `scp`, you may have noticed that a `:` _always_ follows the remote
computer name; sometimes a `/` follows that, and sometimes not, and sometimes
there's a final `/`. On Linux computers, `/` is the ___root___ directory, the
location where the entire filesystem (and others attached to it) is anchored. A
path starting with a `/` is called _absolute_, since there can be nothing above
the root `/`. A path that does not start with `/` is called _relative_, since
it is not anchored to the root.

If you want to upload a file to a location inside your home directory --
which is often the case -- then you don't need a leading `/`. After the
`:`, start writing the sequence of folders that lead to the final storage
location for the file or, as mentioned above, provide nothing if your home
directory _is_ the destination.

A trailing slash on the target directory is optional, and has no effect for
`scp -r`, but is important in other commands, like `rsync`.

> ## A Note on `rsync`
>
> As you gain experience with transferring files, you may find the `scp`
> command limiting. The [rsync] utility provides
> advanced features for file transfer and is typically faster compared to both
> `scp` and `sftp` (see below). It is especially useful for transferring large
> and/or many files and creating synced backup folders.
>
> The syntax is similar to `scp`. To transfer _to_ another computer with
> commonly used options:
>
> ```
> {{ site.local.prompt }} rsync -avzP hpc-intro-code.tar.gz {{ site.remote.user }}@{{ site.remote.login }}:
> ```
> {: .language-bash}
>
> The options are:
>
> * `a` (**a**rchive) to preserve file timestamps, permissions, and folders,
>    among other things; implies recursion
> * `v` (**v**erbose) to get verbose output to help monitor the transfer
> * `z` (compression) to compress the file during transit to reduce size and
>   transfer time
> * `P` (partial/progress) to preserve partially transferred files in case
>   of an interruption and also displays the progress of the transfer.
>
> To recursively copy a directory, we can use the same options:
>
> ```
> {{ site.local.prompt }} rsync -avzP hpc-intro-code {{ site.remote.user }}@{{ site.remote.login }}:~/
> ```
> {: .language-bash}
>
> As written, this will place the local directory and its contents under your
> home directory on the remote system. If the trailing slash is omitted on
> the destination, a new directory corresponding to the transferred directory
> will not be created, and the contents of the source
> directory will be copied directly into the destination directory.
>
> To download a file, we simply change the source and destination:
>
> ```
> {{ site.local.prompt }} rsync -avzP {{ site.remote.user }}@{{ site.remote.login }}:hpc-intro-code ./
> ```
> {: .language-bash}
{: .callout}

File transfers using both `scp` and `rsync` use SSH to encrypt data sent through
the network. So, if you can connect via SSH, you will be able to transfer
files. By default, SSH uses network port 22. If a custom SSH port is in use,
you will have to specify it using the appropriate flag, often `-p`, `-P`, or
`--port`. Check `--help` or the `man` page if you're unsure.

> ## Change the Rsync Port
>
> Say we have to connect `rsync` through port 768 instead of 22. How would we
> modify this command?
>
> ```
> {{ site.local.prompt }} rsync hpc-intro-code.tar.gz {{ site.remote.user }}@{{ site.remote.login }}:
> ```
> {: .language-bash}
>
> > ## Solution
> >
> > ```
> > {{ site.local.prompt }} man rsync
> > {{ site.local.prompt }} rsync --help | grep port
> >      --port=PORT             specify double-colon alternate port number
> > See http://rsync.samba.org/ for updates, bug reports, and answers
> > {{ site.local.prompt }} rsync --port=768 hpc-intro-code.tar.gz {{ site.remote.user }}@{{ site.remote.login }}:
> > ```
> > {: .language-bash}
> {: .solution}
{: .challenge}

## Archiving Files

One of the biggest challenges we often face when transferring data between
remote HPC systems is that of large numbers of files. There is an overhead to
transferring each individual file and when we are transferring large numbers of
files these overheads combine to slow down our transfers to a large degree.

The solution to this problem is to _archive_ multiple files into smaller
numbers of larger files before we transfer the data to improve our transfer
efficiency. Sometimes we will combine archiving with _compression_ to reduce
the amount of data we have to transfer and so speed up the transfer.

The most common archiving command you will use on a (Linux) HPC cluster is
`tar`. `tar` can be used to combine files into a single archive file and,
optionally, compress it.

Let's start with the file we downloaded from the lesson site,
`hpc-intro-code.tar.gz`. The "gz" part stands for _gzip_, which is a
compression library. This kind of file can usually be interpreted by reading
its name: it appears somebody took a folder named "hpc-intro-code," wrapped up
all its contents in a single file with `tar`, then compressed that archive with
`gzip` to save space. Let's check using `tar` with the `-t` flag, which prints
the "**t**able of contents" without unpacking the file, specified by
`-f <filename>`, on the remote computer. Note that you can concatenate the two
flags, instead of writing `-t -f` separately.

```
{{ site.local.prompt }} ssh {{ site.remote.user }}@{{ site.remote.login }}
{{ site.remote.prompt }} tar -tf hpc-intro-code.tar.gz
hpc-intro-code/
hpc-intro-code/amdahl
hpc-intro-code/README.md
hpc-intro-code/LICENSE.txt
```
{: .language-bash}

This shows a folder which contains a few files. Let's see about that
compression, using `du` for "**d**isk **u**sage".

```
{{ site.remote.prompt }} du -sh hpc-intro-code.tar.gz
3.4K     hpc-intro-code.tar.gz
```
{: .language-bash}

> ## Files Occupy at Least One "Block"
>
> If the filesystem block size is larger than 3.4 KB, you'll see a larger
> number: files cannot be smaller than one block.
> You can use the `--apparent-size` flag to see the exact size, although the
> unoccupied space in that filesystem block can't be used for anything else.
{: .callout}

Now let's unpack the archive. We'll run `tar` with a few common flags:

* `-x` to e**x**tract the archive
* `-v` for **v**erbose output
* `-z` for g**z**ip compression
* `-f` for the file to be unpacked

When it's done, check the directory size with `du` and compare.

> ## Extract the Archive
>
> Using the four flags above, unpack the lesson data using `tar`.
> Then, check the size of the whole unpacked directory using `du`.
>
> Hint: `tar` lets you concatenate flags.
>
> > ## Commands
> >
> > ```
> > {{ site.remote.prompt }} tar -xvzf hpc-intro-code.tar.gz
> > ```
> > {: .language-bash}
> >
> > ```
> > hpc-intro-code/
> > hpc-intro-code/amdahl
> > hpc-intro-code/README.md
> > hpc-intro-code/LICENSE.txt
> > ```
> > {: .output}
> >
> > Note that we did not type out `-x -v -z -f`, thanks to the flag
> > concatenation, though the command works identically either way --
> > so long as the concatenated list ends with `f`, because the next string
> > must specify the name of the file to extract.
> >
> > ```
> > {{ site.remote.prompt }} du -sh hpc-intro-code
> > 16K    hpc-intro-code
> > ```
> > {: .language-bash}
> {: .solution}
>
> > ## Was the Data Compressed?
> >
> > Text files (including Python source code) compress nicely: the "tarball" is
> > one-quarter the total size of the raw data!
> {: .discussion}
{: .challenge}

If you want to reverse the process -- compressing raw data instead of
extracting it -- set a `c` flag instead of `x`, set the archive filename,
then provide a directory to compress:

```
{{ site.local.prompt }} tar -cvzf compressed_code.tar.gz hpc-intro-code
```
{: .language-bash}

> ## Working with Windows
>
> When you transfer text files from a Windows system to a Unix system (Mac,
> Linux, BSD, Solaris, etc.) this can cause problems. Windows encodes its files
> slightly different than Unix, and adds an extra character to every line.
>
> On a Unix system, every line in a file ends with a `\n` (newline). On
> Windows, every line in a file ends with a `\r\n` (carriage return + newline).
> This causes problems sometimes.
>
> Though most modern programming languages and software handles this correctly,
> in some rare instances, you may run into an issue. The solution is to convert
> a file from Windows to Unix encoding with the `dos2unix` command.
>
> You can identify if a file has Windows line endings with `cat -A filename`. A
> file with Windows line endings will have `^M$` at the end of every line. A
> file with Unix line endings will have `$` at the end of a line.
>
> To convert the file, just run `dos2unix filename`. (Conversely, to convert
> back to Windows format, you can run `unix2dos filename`.)
{: .callout}

{% include links.md %}

[rsync]: https://rsync.samba.org/
