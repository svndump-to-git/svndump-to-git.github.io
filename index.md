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

