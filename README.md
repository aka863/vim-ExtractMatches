EXTRACT MATCHES   
===============================================================================
_by Ingo Karkat_

DESCRIPTION
------------------------------------------------------------------------------

This plugin provides a toolbox of commands to copy all (or only unique first)
search matches / matches of a passed pattern / entire lines matching, to a
register, or directly :put them into the buffer. The commands are easier to
remember and quicker to type than the various idioms for that, and they are
robust, i.e. also support patterns spanning multiples lines.

### SOURCE

- [Implementation inspiration](http://vim.wikia.com/wiki/Copy_the_search_results_into_clipboard)
- [Use case inspired from a post by Luc Hermitte at](http://www.reddit.com/r/vim/comments/ef9zh/any_better_way_to_yank_all_lines_matching_pattern/)

### ALTERNATIVES

One can employ a sub-replace-expression to capture the matches, as described
in
    http://stackoverflow.com/questions/9079561/how-to-extract-regex-matches-using-vim
The idea is to use the side effect of add() in the expression, and force an
empty return value from it through the inverse range of [1:0]. To avoid text
modification, we make the pattern match nothing by appending /\zs; with
this, \0 will be empty, so we have to capture the match as \1:

    let t=[] | %s/\(fo*)\zs/\=add(t, submatch(1))[1:0]/g

Since this has the side effect of setting 'modified', anyway, we can
alternatively have add() return the last added element [-1]; this saves us
from the zero-width match and capture:

    let t=[] | %s/fo*/\=add(t, submatch(0))[-1]/g

### SEE ALSO

- The PatternsOnText.vim plugin ([vimscript #4602](http://www.vim.org/scripts/script.php?script_id=4602)) provides commands that
  print, substitute, or delete certain duplicates or matches directly in the
  buffer.

### RELATED WORKS

- The yankitute plugin ([vimscript #4719](http://www.vim.org/scripts/script.php?script_id=4719)) provides a similar
  :[range]Yankitute[register]/{pattern}/[string]/[flags]/[join] command.
- yankmatches
  (https://github.com/thoughtstream/Damian-Conway-s-Vim-Setup/blob/master/plugin/yankmatches.vim)
  can yank / delete entire lines with / without matches, similar to
  :GrepToReg.

USAGE
------------------------------------------------------------------------------

    All commands default to the entire buffer if the [range] is omitted.

    :[range]GrepToReg[!] /{pattern}/[x]
    :[range]GrepToReg[!] [{pattern}]
                            Yank all lines in [range] that match {pattern} (or the
                            last search pattern if omitted), with !: do not match,
                            into register [x] (or the unnamed register).

    :[range]GrepRangeToReg[!] /{pattern}/ {range} [x]
    :[range]GrepRangeToReg[!] {range} [x]
                            Yank all lines in [range] that match {pattern} (or the
                            last search pattern if omitted), with !: do not match,
                            and all lines in {range} around it, into register [x]
                            (or the unnamed register).
                            With this, you can emulate grep's context line control
                            -A -B -C / --after-context --before-context --context
                            (but without the "--" group separator).

    :[range]YankMatches[!] /{pattern}/[x]
    :[range]YankMatches[!] [{pattern}]
                            Yank text matching {pattern} (or the last search
                            pattern if omitted) in [range] into register [x] (or
                            the unnamed register). Each match is put on a new
                            line. This works like "grep -o". With [!]: Yank only
                            the first match in each line.
    :[range]YankMatches[!] /{pattern}/{replacement}/[x]
                            Grab text matching {pattern} (or the last search
                            pattern if omitted) in [range], and put {replacement}
                            into register [x] (or the unnamed register). You can
                            refer to the match via s/\& and submatches (s/\1).
                            The matches are simply concatenated without a newline
                            character here. Append \n at {replacement} to have
                            one. When {replacement} is "&...", ... is assumed to
                            be a (literal) separator and is removed from the last
                            element; if you don't want that, use \0 instead of &.
                            With [!]: Yank only the first match in each line.

    :[range]YankUniqueMatches[!] /{pattern}/[x]
    :[range]YankUniqueMatches[!] [{pattern}]
                            Yank text matching {pattern} (or the last search
                            pattern if omitted) in [range] into register [x] (or
                            the unnamed register), but only once. Each match is
                            put on a new line. With [!]: Yank only the first match
                            in each line.
    :[range]YankUniqueMatches[!] /{pattern}/{replacement}/[x]

    :[range]PrintMatches[!] /{pattern}/
    :[range]PrintMatches[!] [{pattern}]
                            Print text matching {pattern} (or the last search
                            pattern if omitted) in [range]. Each match is printed
                            on a new line. This works like "grep -o". With [!]:
                            Print only the first match in each line.
    :[range]PrintMatches[!] /{pattern}/{replacement}/
                            Like :YankMatches, but print the replacement instead
                            of yanking.
    :[range]PrintUniqueMatches[!] /{pattern}/
    :[range]PrintUniqueMatches[!] [{pattern}]
    :[range]PrintUniqueMatches[!] /{pattern}/{replacement}/
                            Like :YankUniqueMatches, but print instead of
                            yanking.

    :[range]SubstituteAndYank /{pattern}/{replacement}/[flags]/{yank-replacement}/[x]
                            Replace all matches of {pattern} in the current line /
                            [range] with {replacement}, like with :substitute
                            (using [flags] as :s_flags), and put the
                            {yank-replacement} (simply concatenated without a
                            newline) into register [x] (or the unnamed register).
                            Supports the same replacements as :YankMatches;
                            additionally,  \# is replaced with a (1-based) count
                            of the current yank and in a sub-replace-expression,
                            v:key stands for the 0-based index.

    :[range]SubstituteAndYankUnique /{pattern}/{replacement}/[flags]/{yank-replacement}/[x]
                            Like :SubstituteAndYank, but only add unique matches
                            to the register. For non-unique matches, \# and v:key
                            refer to the corresponding existing match in the
                            register.

    :[line]PutMatches[!] /{pattern}/
    :[line]PutMatches[!] [{pattern}]
    :[line]PutMatches[!] /{pattern}/{replacement}/
                            Put text matching {pattern} (or the last search pattern
                            if omitted) after [line] (default current line). Each
                            match is put on a new line (except when {replacement}
                            is specified; see :YankMatches). This works like
                            "grep -o".
                            With [!]: Put only the first match in each line.
                            Cp. :YankMatches.

    :[line]PutUniqueMatches[!] /{pattern}/
    :[line]PutUniqueMatches[!] [{pattern}]
    :[line]PutUniqueMatches[!] /{pattern}/{replacement}/
                            Put text matching {pattern} (or the last search pattern
                            if omitted) after [line] (default current line). Each
                            match is once put on a new line.
                            With [!]: Put only the first match in each line.
                            Cp. :YankUniqueMatches.

INSTALLATION
------------------------------------------------------------------------------

The code is hosted in a Git repo at
    https://github.com/inkarkat/vim-ExtractMatches
You can use your favorite plugin manager, or "git clone" into a directory used
for Vim packages. Releases are on the "stable" branch, the latest unstable
development snapshot on "master".

This script is also packaged as a vimball. If you have the "gunzip"
decompressor in your PATH, simply edit the \*.vmb.gz package in Vim; otherwise,
decompress the archive first, e.g. using WinZip. Inside Vim, install by
sourcing the vimball or via the :UseVimball command.

    vim ExtractMatches*.vmb.gz
    :so %

To uninstall, use the :RmVimball command.

### DEPENDENCIES

- Requires Vim 7.0 or higher.
- Requires the ingo-library.vim plugin ([vimscript #4433](http://www.vim.org/scripts/script.php?script_id=4433)), version 1.035 or
  higher.

CONTRIBUTING
------------------------------------------------------------------------------

Report any bugs, send patches, or suggest features via the issue tracker at
https://github.com/inkarkat/vim-ExtractMatches/issues or email (address
below).

HISTORY
------------------------------------------------------------------------------

##### 1.41    04-Nov-2018
- Move PatternsOnText#ReplaceSpecial(), and PatternsOnText#DefaultReplacer()
  to ingo-library.
- Does not require the PatternsOnText.vim plugin ([vimscript #4602](http://www.vim.org/scripts/script.php?script_id=4602)), version
  2.00 or higher for the :SubstituteAndYank[Unique] commands any longer.
  __You need to update to ingo-library ([vimscript #4433](http://www.vim.org/scripts/script.php?script_id=4433)) version 1.035!__

##### 1.40    24-Jan-2017
- ENH: Add :GrepRangeToReg command.
- :YankMatches does not handle magicness modifier atoms (\v, \M, etc.) before
  / after \zs / \ze. They get cut away, and then the remaining pattern does
  not match any longer, and a custom {replacement} is not applied. Normalize
  the magicness in the pattern. Additionally, also keep a case-sensitivity
  atom (\c, \C). Reported by bjornmelgaard on Stack Overflow.
  __You need to update to ingo-library ([vimscript #4433](http://www.vim.org/scripts/script.php?script_id=4433)) version 1.029!__

##### 1.32    07-Dec-2016
- In PatternsOnText.vim version 2.0, PatternsOnText#Selected#ReplaceSpecial()
  has been moved to PatternsOnText#DefaultReplacer().

##### 1.31    06-Dec-2014
- BUG: :GrepToReg runs into endless loop when the last line of the buffer
  belongs to the range and is matching.
- Refactoring: Use ingo#cmdargs#pattern#ParseUnescaped().
  __You need to update to ingo-library ([vimscript #4433](http://www.vim.org/scripts/script.php?script_id=4433)) version 1.020!__

##### 1.30    13-Mar-2014
- CHG: Rename :Yank[Unique]MatchesToReg to :Yank[Unique]Matches; the
  "register" part is implied by the yank.
- CHG: Change default range of :SubstituteAndYank[Unique] to current line
  instead of buffer, to be consistent with :substitute and the :Substitute...
  commands defined by PatternsOnText.vim.
- Add :Print[Unique]Matches variant of :Yank[Unique]Matches.
- FIX: Inline pasting (with replacements) doesn't use the specified line and
  doesn't create a new empty line.
- FIX: Typo in variable name prevented elimination of \ze.
- FIX: Remove escaping of a:replacement to apply the DWIM trailing separator
  removal also to \\, \n, \t etc.
- Handle \r, \n, \t, \b in replacement, too.

##### 1.20    20-Feb-2014
- Add :SubstituteAndYank and :SubstituteAndYankUnique commands.
- All commands now properly abort on errors.

##### 1.10    18-Feb-2014
- DWIM: When {replacement} is "&...", assume ... is a (literal) separator and
  remove it from the last element.
- Add heuristic that drops \zs, \ze, and all location-aware atoms (like \%v)
  for the separate substitution for {replacement}, to allow it to match.
  Beforehand, either nothing or the entire match have been wrongly returned as
  the result.

##### 1.00    11-Dec-2013
- First published version.

##### 0.01    09-Dec-2010
- Started development.

------------------------------------------------------------------------------
Copyright: (C) 2010-2018 Ingo Karkat -
The [VIM LICENSE](http://vimdoc.sourceforge.net/htmldoc/uganda.html#license) applies to this plugin.

Maintainer:     Ingo Karkat <ingo@karkat.de>
