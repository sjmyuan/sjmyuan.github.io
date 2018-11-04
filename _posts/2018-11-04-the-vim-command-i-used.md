---
title: The Vim Commands I Used
tags:
  - Vim
---

Vim is an awesome editor and you can find the Vim simlulator in lots of popular editors(such as Intellij Idea, Atom, Sublime Text). it will make your life more easy if you familiar with the common Vim commands.

# Basic Knowledge

Not like the normal editor, Vim group the text operations into three common modes: Normal, Insert, Visual.[^1]

When we open a document using Vim, we are in `Normal` mode. In this mode we can browse the content of document and do some delete and paste operations. We can always go back to this mode by `Esc`

If we want to add some text, we need to switch to` Insert` mode, there are multiple keys which can lead us to this mode, such as i, a, o, O.

If we want to copy some text, we need to switch to `Visual` mode. the key `v` can lead us to this mode.

The common pattern of command in Vim is `mode+motion`, we will see how it works in next section.

`Normal` mode is the main mode, we always need to go back this mode and then switch to another mode.

# Commands

## Navigation

### Character

| Scenario                                                   | Mode   | Command |
| ---------------------------------------------------------- | ------ | ------- |
| Go to the next character of current line                      | Normal | l       |
| Go to the previous character of current line                  | Normal | h       |
| Go to the beginning of current line                           | Normal | 0       |
| Go to the ending of current line                                 | Normal | $       |
| Go to the next nth character of current line                  | Normal | \<n\>l  |
| Go to the previous nth character of current line              | Normal | \<n\>h  |
| Find the next position of given character in current line     | Normal | f\<c\>  |
| Find the previous position of given character in current line | Normal | F\<c\>  |

### Word

| Scenario                                 | Mode   | Command |
| ---------------------------------------- | ------ | ------- |
| Go to the next ending of  word           | Normal | e       |
| Go to the next beginning of word         | Normal | w       |
| Go to the previous beginning of word     | Normal | b       |
| Go to the previous ending of word        | Normal | ge      |
| Go to the next nth beginning of word     | Normal | \<n\>w  |
| Go to the next nth ending of word        | Normal | \<n\>e  |
| Go to the previous nth beginning of word | Normal | \<n\>b  |
| Go to the previous nth ending of word    | Normal | \<n\>ge |

### Line

| Scenario                                | Mode   | Command |
| --------------------------------------- | ------ | ------- |
| Go to the beginning of current document | Normal | gg      |
| Go to the ending of current document    | Normal | G       |
| Go to the next line                     | Normal | k       |
| Go to the previous line                 | Normal | j       |
| Go to the next nth line                 | Normal | \<n\>j  |
| Go to the previous nth line             | Normal | \<n\>k  |
| Go to the given line number             | Normal | \<n\>gg |

### Block

| Scenario                 | Mode   | Command |
| ------------------------ | ------ | ------- |
| Go to next page          | Normal | Ctrl+f  |
| Go to previous page      | Normal | Ctrl+b  |
| Go to next paragraph     | Normal | }       |
| Go to previous paragraph | Normal | {       |

## Delete

### Character

| Scenario                                                     | Mode   | Command |
| ------------------------------------------------------------ | ------ | ------- |
| Delete the character under cursor                            | Normal | x       |
| Delete the character before cursor                           | Normal | X       |
| Delete the next nth characters(include current cursor) in current line | Normal | \<n\>x  |
| Delete the previous nth characters(not include current cursor) in current line | Normal | \<n\>X  |
| Delete all characters from cursor(include) until next given character(not include) in current line | Normal | dt\<c\> |
| Delete all characters from cursor(not include) until previous given character(not include) in current line | Normal | dT\<c\> |
| Delete all characters from cursor(include) to next given character(include) in current line | Normal | df\<c\> |
| Delete all characters from cursor(not include) to previous given character(include) in current line | Normal | dF\<c\> |
| Delete all characters after cursor(include) in current line  | Normal | D or d$ |
| Delete all characters before cursor(not include) in current line | Normal | d0      |

### Word

| Scenario                                                     | Mode   | Command  |
| ------------------------------------------------------------ | ------ | -------- |
| Delete all characters from cursor(include) to the next beginning of word | Normal | dw       |
| Delete all characters from cursor(include) to the next ending of word | Normal | de       |
| Delete all characters from cursor(not include) to the previous beginning of word | Normal | db       |
| Delete all characters from cursor(not include) to the previous ending of word | Normal | dge      |
| Delete all characters from cursor(include) to the next nth beginning of word | Normal | d\<n\>w  |
| Delete all characters from cursor(include) to the next nth ending of word | Normal | d\<n\>e  |
| Delete all characters from cursor(not include) to the previous nth beginning of word | Normal | d\<n\>b  |
| Delete all characters from cursor(not include) to the previous nth ending of word | Normal | d\<n\>ge |

### Line

| Scenario                                                     | Mode   | Command              |
| ------------------------------------------------------------ | ------ | -------------------- |
| Delete the current line                                      | Normal | dd                   |
| Delete the lines from beginning to current line(include)     | Normal | dgg                  |
| Delete the lines from current line(include) to ending        | Normal | dG                   |
| Delete the next nth lines(include current line)              | Normal | \<n\>dd or d\<n-1\>j |
| Delete the previous nth lines(include current line)          | Normal | d\<n-1\>k            |
| Delete the lines from current line(include) to the given line number | Normal | d\<n\>gg             |

### Block

| Senario                                                      | Mode   | Command |
| ------------------------------------------------------------ | ------ | ------- |
| Delete all characters from cursor to the ending of current paragraph | Normal | d}      |
| Delete all characters from the beginning of current paragraph to cursor | Normal | d{      |
| Delete the next nth paragraph                                | Normal | d\<n\>} |
| Delete the previous nth paragraph                            | Normal | d\<n\>{ |

### Column Mode

| Scenario                                  | Mode                                   | Command                  |
| ----------------------------------------- | -------------------------------------- | ------------------------ |
| Delete the same range of continuous lines | Normal<br/>Blockwise Visual<br/>Normal | Ctrl+v<br/>h/l<br/>x/d/X |

## Insert

### Character

| Scenario                  | Mode              | Command |
| ------------------------- | ----------------- | ------- |
| Insert text before cursor | Normal<br/>Insert | i       |
| Insert text after cursor  | Normal<br/>Insert | a       |

### Line

| Scenario                                  | Mode              | Command |
| ----------------------------------------- | ----------------- | ------- |
| Insert text before current line              | Normal<br/>Insert | O       |
| Insert text after current line               | Normal<br/>Insert | o       |
| Insert text at the beginning of current line | Normal<br/>Insert | I       |
| Insert text at the ending of current line       | Normal<br/>Insert | A       |

### Column Mode

| Scenario                                                  | Mode                                               | Command                       |
| --------------------------------------------------------- | -------------------------------------------------- | ----------------------------- |
| Insert same text at the same position of continuous lines | Normal <br/>Blockwise Visual<br/>Insert<br/>Normal | Ctrl+v<br/> j/k<br/>I<br/>Esc |

## Select

### Character

| Scenario                 | Mode                            | Command |
| ------------------------ | ------------------------------- | ------- |
| Select text by character | Normal<br/>Characterwise Visual | v       |

### Line

| Scenario            | Mode                       | Command |
| ------------------- | -------------------------- | ------- |
| Select text by line | Normal<br/>Linewise Visual | V       |

### Block

| Scenario             | Mode                        | Command |
| -------------------- | --------------------------- | ------- |
| Select text by block | Normal<br/>Blockwise Visual | Ctrl+v  |

## Copy

### Character

| Scenario                                                     | Mode   | Command |
| ------------------------------------------------------------ | ------ | ------- |
| Copy all characters from cursor until the next given character(not include) | Normal | yt\<c\> |
| Copy all characters from cursor to the next given character(include) | Normal | yf\<c\> |
| Copy all characters from the previous given character(not include) to cursor | Normal | yT\<c\> |
| Copy all characters from the previous given character(include) to cursor | Normal | yF\<c\> |

### Word

| Scenario                                                     | Mode   | Command  |
| ------------------------------------------------------------ | ------ | -------- |
| Copy all characters from cursor to the next beginning of word | Normal | yw       |
| Copy all characters from cursor to the next ending of word      | Normal | ye       |
| Copy all characters from cursor to the next nth beginning of word | Normal | y\<n\>w  |
| Copy all characters from cursor to the next nth ending of word  | Normal | y\<n\>e  |
| Copy all characters from cursor to the previous beginning of word | Normal | yb       |
| Copy all characters from cursor to the previous ending of word  | Normal | yge      |
| Copy all characters from cursor to the previous nth beginning of word | Normal | y\<n\>b  |
| Copy all characters from cursor to the previous nth ending of word | Normal | y\<n\>ge |

### Line

| Scenario                    | Mode   | Command   |
| --------------------------- | ------ | --------- |
| Copy current line           | Normal | yy or Y   |
| Copy the next nth lines     | Normal | \<n\>yy   |
| Copy the previous nth lines | Normal | y\<n-1\>k |

### Block

| Scenario                                         | Mode   | Command      |
| ------------------------------------------------ | ------ | ------------ |
| Copy current paragraph                           | Normal | y}           |
| Copy the next nth paragraph                      | Normal | y\<n\>}      |
| Copy the previous nth paragraph                  | Normal | y\<n\>{      |
| Copy all the characters in surround(not include) | Normal | yi\<symbol\> |
| Copy all the characters in surround(include)     | Normal | ya\<symbol\> |

## Paste

### Character

| Scenario           | Mode   | Command |
| ------------------ | ------ | ------- |
| Paste after cursor | Normal | p       |
| Paste befor cursor | Normal | P       |

### Line

| Scenario                                | Mode   | Command   |
| --------------------------------------- | ------ | --------- |
| Paste text before current line          | Normal | :pu!      |
| Paste text after current line           | Normal | :pu       |
| Paste text before the given line number | Normal | :\<n\>pu! |
| Paste text after the given line number  | Normal | :\<n\>pu  |

## Replace

### Character

| Scenario                                                     | Mode               | Command |
| ------------------------------------------------------------ | ------------------ | ------- |
| Replace all the characters from cursor to the ending of current line | Normal<br/>Replace | R       |
| Replace character under cursor with other character          | Normal             | r\<c\>  |
| Swith the character under cursor to uppercase/lowercase      | Normal             | ~       |

### Document

| Scenario                                                  | Mode   | Command                                   |
| --------------------------------------------------------- | ------ | ----------------------------------------- |
| Replace all occurances of given pattern to another string | Normal | :%s/\<given pattern\>/\<target string\>/g |

# Advance

According to this blog, you already can do most of work in Vim. If you want to make your Vim more fancy and tune your workflow, you may want to play some Vim plugins. The following steps may help you to enjoy it.

1. Choose a Vim implementation to play

   There are several implementations you can choose

   * [Vim](https://github.com/vim/vim)
   * [NeoVim](https://github.com/neovim/neovim)
   * [MacVim](https://github.com/macvim-dev/macvim)

2. Set up your `.vimrc` configuration

   The first thing you need to do is choosing a plugin manager, [vim-plug](https://github.com/macvim-dev/macvim) is a good choice for beginner. but the easiest way may just copy an existing dotfile from GitHub.

3. Try different plugins

   [vimawesome](https://vimawesome.com/) is a good place to find vim plugins, for beginner you may need the following feature

   * Theme
   * File Finder
   * Syntax
   * Motion
   * Code Complement
   * Code Format

That's it, Hope you enjoy the Vim journey!

# References

[^1]: [Learning the vi Editor/Vim Modes](https://en.wikibooks.org/wiki/Learning_the_vi_Editor/Vim/Modes)







