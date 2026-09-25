# Bandit Level 6

## Task
Find a file somewhere on the entire server that's owned by user `bandit7`, 
owned by group `bandit6`, and exactly 33 bytes in size. That file holds the 
password for bandit7.

## What I tried first
Started by checking my home directory with `ls -la`, but it was empty except 
default files — realized the target file had to be somewhere else on the 
whole system, not in my home folder.

Made several small syntax mistakes building the `find` command:
- Forgot the starting path entirely at first
- Wrote `bandit 7` with a stray space instead of `bandit7`
- Glued flags and paths together with no space (`./-user`, `-size33c`) instead 
  of separating each piece with a space
- Started searching from `.` (current folder) or `./` instead of `/` (the 
  root of the whole filesystem)

## What I learned
- `find` always needs a starting path right after it — `/` searches the 
  entire filesystem, `.` only searches the current folder
- Every flag and value needs a space before it — no gluing pieces together
- `find` can combine ownership filters (`-user`, `-group`) with size (`-size`) 
  in one single command
- Searching the whole filesystem hits folders you don't have permission to 
  read — redirecting those errors with `2>/dev/null` hides the noise so only 
  real results show up

## The working command
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null

## Takeaway
This was mostly a syntax-precision level rather than a new concept — every 
individual piece (`-user`, `-group`, `-size`) I already sort of knew from 
level 5, but stringing them together correctly, with the right starting path 
and proper spacing, took a few tries. Good reminder that small syntax details 
(a missing space, wrong starting path) can block an otherwise correct idea.
