# Whole Subversion conversions to Git

The svndump-to-git-converter is a Java Spring based application useful for converting everything from large subversion repositories into git repositories.

It implements automatic branch detection logic with extension points for project specific.

Other conversion solutions exist but they are either not-native (The Perl based git-svn) or require you to know exactly which part of the repository you want to convert (svn2git as used by the KDE project).

The svndump-to-git-converter will convert the entire repository and then it can be sliced up into sensible parts going forward.  

Post conversion you will want to identify giant binary files and consider removing them from the repository.  For example if you plan on uploading the converted repository into github then you will need to make sure that no file is larger than 100 MB.

The converter works by reading in subversion dump files (in the version 2 format) and writing the corresponding files into a bare git repository.  Branch detection will automatically split the overall path to a file in a branch part and tree part.

