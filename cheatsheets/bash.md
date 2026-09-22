# bash cheatsheet

bash 5.x. These are the constructs I look up every time I write a script.

## Shebang / safety

```bash
#!/usr/bin/env bash
set -euo pipefail     # -e exit on error, -u unset=error, -o pipefail
set -x                # trace (debug)
IFS=$'\n\t'           # safer word splitting
```

`set -e` has exceptions (commands in `if`, `&&`, `||`, or negated don't trigger
it). Don't treat it as a blanket safety net.

## Variables

```bash
name="value"
readonly PI=3.14
local x=1                    # inside a function
export PATH="$HOME/bin:$PATH"

echo "${name}"               # always quote
echo "${name:-default}"      # default if unset/empty
echo "${name:=default}"      # assign default if unset
echo "${name:?must be set}"  # error out if unset
echo "${#name}"              # length
echo "${name:0:3}"           # substring
echo "${name%.txt}"          # strip shortest suffix
echo "${name##*/}"           # strip longest prefix (basename)
echo "${name#*.}"            # strip shortest prefix
echo "${name^^}"             # uppercase
echo "${name,,}"             # lowercase
echo "${name/old/new}"       # replace first
echo "${name//old/new}"      # replace all
```

## Arrays

```bash
arr=(one two three)
arr+=(four)
echo "${arr[0]}"             # first
echo "${arr[@]}"             # all (quote it!)
echo "${#arr[@]}"            # count
for x in "${arr[@]}"; do echo "$x"; done
unset 'arr[1]'
mapfile -t lines < file.txt  # file -> array, one line per element
```

## Conditionals

```bash
[[ -f "$f" ]]     # is a regular file
[[ -d "$d" ]]     # is a directory
[[ -e "$p" ]]     # exists
[[ -r "$f" ]]     # readable
[[ -s "$f" ]]     # non-empty
[[ -z "$s" ]]     # string empty
[[ -n "$s" ]]     # string non-empty
[[ "$a" == "$b" ]]
[[ "$a" =~ ^[0-9]+$ ]]        # regex
(( n > 10 ))                  # arithmetic
```

Use `[[ ]]`, not `[ ]`. It handles empty strings and doesn't word-split.

## Loops

```bash
for i in {1..5}; do echo "$i"; done
for f in *.txt; do echo "$f"; done
while read -r line; do echo "$line"; done < file.txt
while IFS=, read -r a b c; do echo "$a|$b|$c"; done < data.csv
until [[ -f done.flag ]]; do sleep 1; done
```

`while read` over a pipe runs in a subshell; variables set inside won't persist.
Use process substitution `done < <(cmd)` to keep them.

## Functions

```bash
log() {
  printf '%s %s\n' "$(date +%H:%M:%S)" "$*" >&2
}

greet() {
  local name="${1:-world}"
  echo "hello $name"
}
greet
greet ann
```

`"$*"` joins args with the first char of IFS; `"$@"` keeps them separate. Prefer
`"$@"`.

## Redirection / pipes

```bash
cmd > out.txt          # stdout to file (truncate)
cmd >> out.txt         # append
cmd 2> err.txt         # stderr
cmd &> all.txt         # both
cmd > out 2>&1         # both (order matters)
cmd 2>/dev/null        # discard stderr
cmd | tee out.txt      # to stdout AND file
cmd > /dev/null 2>&1   # discard everything
cmd < in.txt
```

Process substitution:

```bash
diff <(sort a.txt) <(sort b.txt)
while read -r x; do ...; done < <(cmd)
```

## Arithmetic

```bash
(( sum = a + b ))
(( count++ ))
echo $(( 10 / 3 ))            # integer division -> 3
echo "scale=2; 10/3" | bc     # floats need bc or awk
```

## Exit codes / error handling

```bash
cmd || { echo "failed" >&2; exit 1; }
cmd && echo "ok"
$?                            # last exit code
if ! cmd; then echo no; fi
```

Cleanup on exit:

```bash
tmp="$(mktemp -d)"
cleanup() { rm -rf "$tmp"; }
trap cleanup EXIT
trap 'echo interrupted; exit 130' INT
```

## Strings / parsing

```bash
basename /a/b/c.txt          # c.txt
dirname /a/b/c.txt           # /a/b
realpath ./x                 # absolute path
read -r a b c <<< "1 2 3"    # split into vars
printf '%s\n' "$@"           # print each arg on its own line
tr '[:upper:]' '[:lower:]' < f
cut -d, -f1,3 file.csv
awk -F: '{print $1}' /etc/passwd
sed -n '2,5p' file
```

## Gotchas

- Always quote `"$var"`. Unquoted breaks on spaces and globs.
- `$(...)` over backticks; nests cleanly and reads better.
- `local` only works inside functions.
- `set -e` won't catch a failure inside `$(...)` in some older bash; check `$?`
  if it matters.
- `read` without `-r` eats backslashes. Always `read -r`.
