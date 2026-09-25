# Bandit Level 2

## Task
Read the contents of a file with spaces in its name, sitting in the home directory.

## What I tried first
Ran `cat` followed by the filename typed out normally, but since the name had 
spaces in it, the shell split it into multiple separate words instead of 
treating it as one filename — so `cat` couldn't find anything matching any of 
those individual pieces.

## What I learned
- A space normally tells the shell "this is a new argument," which breaks 
  filenames that contain spaces
- Wrapping the whole filename in quotes tells the shell "treat everything 
  inside these quotes as one single argument," spaces included

## The working command
cat "spaces in this filename"

## Takeaway
Same underlying idea as level 1 (special characters in filenames need special 
handling) but a different fix — quotes for spaces, `./` or `--` for a leading 
dash. Good first sign that a small set of tricks (quotes, `./`, `--`) covers a 
lot of "weird filename" problems.
