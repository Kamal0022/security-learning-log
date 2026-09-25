# Bandit Level 1

## Task
Read the contents of a file called `-` (a single dash) sitting in the home directory.

## What I tried first
Ran `cat -` directly, but the terminal got confused — a filename starting with 
a dash gets misread as a command flag instead of an actual filename.

## What I learned
- A dash at the start of a filename clashes with how commands read flags
- You can tell a command "treat this as a plain filename, not a flag" by 
  prefixing it with `./` (means "in this current folder") or using `--` first

## The working command
cat ./-

## Takeaway
Filenames that start with special characters like `-` need special handling — 
this pattern (using `./` or `--`) came back again in later levels with 
different tricky filenames.
