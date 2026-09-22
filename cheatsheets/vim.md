# vim cheatsheet

vim 9 / neovim 0.9. `<C-x>` = Ctrl+x. Default keybindings (no plugins).

## Modes

```
i / a / o     insert before / after cursor / new line below
I / A         insert at line start / end
O             new line above
v / V / <C-v> charwise / linewise / blockwise visual
Esc / <C-c>   back to normal
:             command-line
```

## Moving (the useful subset)

```
w / b         next / previous word
e             end of word
0 / ^ / $     line start / first non-blank / line end
gg / G        file start / file end
:<n>          go to line n
fx / Fx       next / prev occurrence of x on line
tx            till before x
%             matching bracket
{ / }         paragraph back / forward
H / M / L     top / middle / bottom of screen
<C-d> <C-u>   half page down / up
<C-o> <C-i>   jump back / forward (jump list)
m a / ' a     set mark a / jump to mark a
zz / zt / zb  center / top / bottom current line
```

## Editing

```
x / X         delete char under / before cursor
dd / yy / p   delete line / yank line / paste after
D / C         delete / change to end of line
cc            change whole line
r x           replace one char
~             toggle case
u / <C-r>     undo / redo
.             repeat last change
J             join line below
>> / <<       indent / outdent
=G            auto-indent to end of file
gq            reflow/format selected text
```

## Visual mode

```
v ... d       delete selection
v ... y       yank
V ... >       indent selected lines
<C-v> ... I   block insert (type, Esc, applies to all lines)
<C-v> ... d   block delete (columns)
gv            reselect last visual
```

## Search / replace

```
/pat          search forward
?pat          search backward
n / N         next / prev match
* / #         search word under cursor
:noh          clear highlight
:%s/old/new/g              all lines
:%s/old/new/gc             confirm each
:10,20s/old/new/g          line range
:'<,'>s/old/new/g          visual selection (type : from visual)
:s/\v(\w+)\s+(\w+)/\2 \1/  swap two words (very magic)
```

## Registers / clipboard

```
"ayy          yank into register a
"ap           paste from register a
"+y / "+p     system clipboard (if +clipboard)
:reg          show registers
"0p           paste last yanked (not deleted)
"1p           paste last deleted
```

## Files / windows / tabs

```
:e file       edit file
:w / :q / :wq save / quit / save+quit
:q!           quit without saving
:w !sudo tee %   save a root-owned file you opened without sudo
:sp file      horizontal split
:vsp file     vertical split
<C-w> h/j/k/l move between splits
<C-w> =       equalize splits
:tabnew / gt / gT   new tab / next / prev tab
:bn / :bp     next / previous buffer
```

## Macros

```
qa            start recording into register a
q             stop recording
@a            play macro a
@@            repeat last macro
5@a           play macro a five times
```

## Ranges / ex commands

```
:%d           delete all
:g/pat/d      delete lines matching pat
:g/pat/s/x/y/ substitute on matching lines only
:v/pat/d      delete lines NOT matching pat
:g!/^#/d      remove all non-comment lines
:r file       read file in below cursor
:r !cmd       insert command output
:!cmd         run shell command
```

## Marks / jumps

```
ma / 'a       set / jump to mark a
''            jump back to previous position
`.            jump to last change
g; / g,       walk the change list
```

## Useful commands

```
:set paste    avoid auto-indent mess when pasting (then :set nopaste)
:set nu       line numbers
:set hlsearch incsearch
:set list     show tabs/invisible chars
:retab        convert tabs to spaces per settings
:sort         sort lines
:sort u       sort + dedupe
:!python %    run current file with python
:%!jq .       pipe whole buffer through jq
:%!column -t  align columns
```

## Notes

- `.` + macros are the two biggest time savers. Learn `.` first.
- `:w !sudo tee %` saves a file you forgot to open with sudo. Genuinely useful.
- If paste looks like a staircase, it's auto-indent; use `:set paste`.
- `ciw` (change inner word), `ci"`, `da(` — text objects compose with any
  operator. Worth the muscle memory.
