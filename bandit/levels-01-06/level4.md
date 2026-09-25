# Bandit Level 4

## Task
Find one specific human-readable file among 10 files (`-file00` through 
`-file09`) inside a subdirectory — all the same size, all with names starting 
with a dash.

## What I tried first
Started to check files one by one with `cat`, but realized with 10 files that 
would be slow and I still wouldn't necessarily be able to tell which one was 
actually meant to be read versus junk data, since some could be binary.

## What I learned
- The `file` command identifies what *type* of content a file actually holds 
  (text, binary data, an image, etc.) without needing to open it — much safer 
  and faster than blindly `cat`-ing everything
- `file` can take a wildcard pattern (`*`) to check many files in a single 
  command instead of running it once per file
- Filenames starting with `-` still needed the same `./` prefix trick from 
  level 1 to be read correctly

## The working commands
file ./-file*
cat ./-file07

## Takeaway
`file` + wildcards turned a "check 10 things manually" problem into "check 10 
things in one command, then read only the one that matters." This is a 
pattern I'll keep reaching for anytime there's a pile of unknown files to sort 
through.
