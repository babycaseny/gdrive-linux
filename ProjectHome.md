This is an attempt to develop a _Google Drive_ client for Linux. It will be written in Python, but we're not attempting to make it work on other OSes any time soon.

Three different modes of operation will be supported:

  * A command line interface that can query Google Drive, list files and folders, download files or folders, etc.

  * A sync daemon that will run in the background and keep a local tree synchronised with a Google Drive. Initially, this will be a read-only copy, until (if) we figure out how to handle all the corner cases.

  * A virtual file-system, implemented using the FUSE Python bindings, that will use the sync daemon.

This project is really just a learning experience: how to use the Google APIs, how to implement an FS using FUSE, etc.