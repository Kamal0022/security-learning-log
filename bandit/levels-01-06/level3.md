# Bandit Level 3

## Task
Read the contents of a file hidden inside a subdirectory, whose name combined 
two tricky patterns at once: it started with dashes AND had spaces in it 
(something like `--spaces in this filename--`).

## What I tried first
1. `ls -la` inside the subdirectory to reveal the hidden file (files starting 
   with `.` don't show in a plain `ls`).
2. Tried `cat "--spaces in this filename--"` — quotes alone weren't enough. 
   The shell still read the leading `--` as "end of flags" marker combined 
   wrong with the rest, and `cat` threw an "unexpected argument" error.
3. Tried `cat ./--spaces in this filename--` — added `./` but dropped the 
   quotes by mistake, so the spaces broke the filename back into separate 
   words again, causing a new set of "No such file or directory" errors, one 
   per word.
4. The error message itself gave a hint: it suggested using `--` before the 
   argument to mark it as a plain value.

## What I learned
- This level needed BOTH fixes at once, not one or the other: `--` to stop the 
  leading dashes being read as flags, AND quotes to keep the spaces from 
  splitting the filename apart
- Order and spacing matter — `-- 'filename'` (with a space after the double 
  dash) works; `--'filename'` (no space) does not, because it glues the 
  characters together into one broken argument
- Reading the actual error message carefully gave the exact fix, instead of 
  guessing randomly

## The working command
cat -- '--spaces in this filename--'

## Takeaway
The biggest lesson here wasn't the command itself — it was learning to read 
error messages as clues instead of just noise. Combining two fixes at once 
(flags + quoting) is a pattern that likely comes back in later levels too.
