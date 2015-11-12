An auto-update system has two parts: a client which polls 
for updates at a specified interval, and a server which 
publishes them. The client downloads updates, installs them,
switches versions and restarts the app on a new version.

## Goals

### Allow switching between versions

Sometimes a new version of a program is worse than the old one. 
Sometimes it's an alpha or beta that people might want to try 
but not stick with. Sometimes the new version breaks a plugin 
that the user cannot live without. For all these reasons users
should be allowed to switch back and forth between app versions
anytime.

Switching between versions requires restarting the app, 
but that's ok because good apps stop and start quickly, don't
require user interaction when stopping, and restore the user 
context when restarted.

The app might have ways to upgrade itself without restarting
but here we're talking about a general solution that is 
transparent to the app.

### Allow frequent updates

Old versions should stick arround and be completely isolated from
each other at runtime but still allow for frequent updates without
filling up the hard drive.

### Mitigate bad updates

Sometimes an update introduces a fatal bug which crashes the app
before the UI to select a different version gets a chance to appear.
This creates a use case for automatic rollback. For this to work,
the code that performs the rollback should be stable and not 
frequently updated.

## Implementation

It should work like this:
- client knows which version is currently running (initially none)
- client requests the list of versions which the server publishes
- client downloads the latest version if newer and unzips it into a new folder
- client sets a "new version" flag on the filesystem and restarts the app
- the app sees the flag and starts on the new version but sets a "trying" 
flag before loading any files; after a while (when the app deems that 
everything went ok), the app resets both flags; if the app crashes before 
resetting the flags, next time the app starts it loads the old version

## API

### Low-level API

remote_versions() -> {v1, ...}   (ask server for available versions)
local_versions() -> {v1, ...}   (downloaded versions)
download(ver) -> true|progress|nil,err
remove(ver)
restart()
run(ver)   (exec a version and exit)
state([t]) /-> t  (get/set state)

### Hi-level API

switchto(ver) -- set flag for the next restart
isvalid(ver) -> t|f  check if a version is valid
validate(t|f)  (validate or invalidate that the current version didn't crash
current_version() -> version
last_version() -> last version (if any) that ran successfully before this version

## Implementation

- a http server publishes a text file with the available versions and the urls for where to download each version
- a version is a zip file with the main exe and any dependencies
- the main exe is a non-console app which executes the current version exe
- the server can publish `<ver1>-<ver2>-diff.zip` files which only contain files that need to be updated from ver1 to ver2.  If the client already has ver1 it can load this smaller file and do a copy & overwrite,
- the client should be able to download everything asynchronously

## Git integration

- gen. version list from git tags
- checkout each tag to a diff. work-tree
- gen. zip files (incl. diff zips) from all work-trees
