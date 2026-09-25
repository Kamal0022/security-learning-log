# Bandit Level 5

## Task
Find one specific file somewhere inside a directory tree containing 20 
subfolders (`maybehere00` through `maybehere19`), based on it being human-
readable, exactly 1033 bytes, and not executable.

## What I tried first
Manually poked around with `cat` on a few folders and even accidentally left 
a `cat` command running with no arguments, which hung the terminal until I 
used `fg` and `Ctrl+C` to kill it. Quickly realized checking 20 folders (each 
likely containing more files) by hand wasn't realistic.

## What I learned
- `find` isn't just for names — it can filter by file type (`-type f` for 
  regular files only, excluding directories) and by exact size in bytes 
  (`-size 1033c`, where `c` means "count in bytes")
- These flags can be combined in a single command to search an entire 
  directory tree at once instead of checking folder by folder
- A backgrounded/stuck process shows up as "Stopped" in the terminal — `fg` 
  brings it back to the foreground so you can properly kill it with `Ctrl+C`, 
  instead of it sitting there silently

## The working commands
find . -type f -size 1033c
cat ./inhere/maybehere07/.file2

## Takeaway
Whenever there are too many files/folders to reasonably check by hand, `find` 
with the right combination of flags is almost always the answer — this is 
becoming the single most useful command so far.
