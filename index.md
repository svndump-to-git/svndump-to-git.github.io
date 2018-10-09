# Whole Subversion conversions to Git

The [svndump-to-git/converter](https://github.com/svndump-to-git/converter) is a Java application useful for converting an entire subversion repository into a git repository with automatic branch detection.

Using spring profiles branch detection plugins can be written to override the defalt branch detection logic in certain cases.

Although in practice this is not always required.

The converter reads Bzip2 compressed subversion dump files (version 2) and branch detects the subversion paths into a branch part and directory part which are then saved into a bare git repository.

A key part of the subversion dump file is the Copy-from stanza which indicates that the file in question was either a direct copy or a copy and modify of an existing file from a previous subversion revision.

In order to facilitate this the git branches and the commits they point to are stored for each subversion revision.

During the conversion process (which can be incremental) this extra data is needed but after the conversion can be discarded (normally by pushing the converted repository references out to another repository.

Post conversion it is essential to check key branches and release tags between git and subversion to make sure the files are the same and there is no difference.

When differences are found they indicate a bug in the processing logic typically related to Copy-from data on prior revisions.  If these can be replicated by adding a new example to the test suite then they can be fixed.

# Contributions Welcome

Subversion to Git conversions are interesting in that they are sort of one off projects.  Please feel free to file issues or pull requests for any issues encountered and I'll apply and publish them for others.  There is a quite extensive test suite so a breaking test showing how the importer is loosing history or misidentifying standard branches, etc can be filed and investigated without needing access to any confidential or private repositories.


# Preparing Svn Dump Files for the Importer


The SvnDump to Git importer reads Subersion version 2 dump files and writes their content into a git repository.

In order to create a version 2 dump you need to have shell access to the directory containing the subversion repository.

An easy way to do this is to mirror the real svn repository and then dump out the files from the local mirror.

## Create svn mirror:

1. svnadmin create mirror

2. create a file called mirror/hooks/pre-revprop-change

2.1 add the following into the file

```
#!/bin/bash
exit 0
```

2.2 make the file executable (chmod +x mirror/hooks/pre-revprop-change)

3. initialize the sync

svnsync init file:///path/to/mirror https://url/of/remote/repo

svnsync sync file:///path/to/mirror

(wait)

It can take a long time to since a big repository so leave this running over night and come back to it later.

The sync can be added into the crontab later to automatically keep your mirror upto date.


## Create an incremental dump file


```
$ svnadmin dump --incremental -r 0:HEAD /absolute/path/to/subversion-repo | bzip2 > repoName-r0:HEAD.dump.bz2
```

Note that for larger repositories you might want to split the total number of revisions into say 10,000 revision chunks.  This is useful if you encounter any bug in the importer as you don't need to restart from the initial revision.

# Convert Svndump file version 2 into a Git Repository

The project has been published in maven central:

Download from: https://search.maven.org/artifact/io.github.svndump-to-git/git-importer/1.0/jar

```
$ java -jar git-importer.jar
0    [main] ERROR io.github.svndump_to_git.importer.GitImporterMain  - USAGE: <svn dump file> <git repository> <veto.log> <skipped-copy-from.log> <blob.log> <gc enabled> <svn repo base url> <repo uuid> <email host part> [<git command path>]
1    [main] ERROR io.github.svndump_to_git.importer.GitImporterMain  -  <veto.log> : which paths were veto's as not being a valid branch
1    [main] ERROR io.github.svndump_to_git.importer.GitImporterMain  -  <skipped-copy-from.log> : which copy-from-paths were skipped
1    [main] ERROR io.github.svndump_to_git.importer.GitImporterMain  -  <blob.log> : issues related to blobs (typically directory copy related)
1    [main] ERROR io.github.svndump_to_git.importer.GitImporterMain  -  <gc enabled> : set to 1 (true ever 500 revs) or 0 (false) to disable
1    [main] ERROR io.github.svndump_to_git.importer.GitImporterMain  -  <svn repo base url> : the svn repo base url to use in the git-svn-id
1    [main] ERROR io.github.svndump_to_git.importer.GitImporterMain  -  <repo uuid> : The svn repository uuid to use in the git-svn-id.
        It you are importing from a clone use this to set the field to the real repositories uuid.
1    [main] ERROR io.github.svndump_to_git.importer.GitImporterMain  -  <host:email host part|authors:authors.txt path> : The svn author becomes author@emailHostPart when imported.
1    [main] ERROR io.github.svndump_to_git.importer.GitImporterMain  -  <git command path> : the path to a native git to use for gc's which occur every 500 revs

```

| *Option* | *Description* |
| SVN Dumpfile | The bzip2 dump file.  Note: a repo can be imported by running the importer on several dump files but they have to be applied in order and the importer assumes you know what you are doing in this regard. |
| Git Repository | The initialized bare git repository that the svndump file contents will be imported into. |
| veto.log | not used but needs to be in the command line.  just use veto.log |
| skipped-copy-from.log | When processing the svndump copy-from rule this logs paths that were skipped.  This can be useful if the result of the import is different from the source. |
| blob.log | Certain blob problems are logged in this file. |
| GC enabled | Normally should be set to 1 to force a GC every 500 imported revs.  Works best if using C git. |
| svn repo base url | The commit message for each imported revision is suffixed with a message similiar to the git-svn-id: https://my-server.com/repos/project/branch/name/path@revision uuid |
| svn repo uuid | Used in the git-svn-id part of the commit message. |
| email host:company.com | Will make all subversion username become username@company.com in the converted repository |
| email authors:authors.txt | Will provide an authors.txt file that maps the subersion username into a "Firstname Lastname <firstname.lastname@company.com>".  The company part can vary. |
| path to C git command | The absolute path to the C git command.

