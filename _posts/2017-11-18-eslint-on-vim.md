---
title: Fix the ESLint issue in Vim
excerpt: "Create a custom maker to allow Neomake use local ESLint"
---
# Contents
{:.no_toc}

* Toc
{:toc}

## Background
I'm doing some JavaScript code work on Neovim and heavily depend on the plugin Neomake.

I prefer to use the local configuration to do the syntax check.

## What Happen
When I save the file, there is nothing happen. seems the plugin has broken.

## What I Did
+ Set `g:neomake_verbose` to `10` to see the debug message

  The log just say Neomake start a async job, no more useful message

+ Set `g:neomake_logfile` to `/tmp/neomake.log` to see the log

  The log file record all the message and say ESLint can't find one plugin.


## How to fix
Neomake actually use ESLint to do the syntax check and it use the global ESLint in default.
If you want to use the local ESLint, you must create a custom maker and set the enabled maker to it.

~~~
  let g:neomake_javascript_local_eslint_maker = {
          \ 'exe': getcwd().'/node_modules/.bin/eslint',
          \ 'args': ['-f', 'compact'],
          \ 'errorformat': '%E%f: line %l\, col %c\, Error - %m,' .
          \   '%W%f: line %l\, col %c\, Warning - %m,%-G,%-G%*\d problems%#'
          \ }
  let g:neomake_javascript_enabled_makers = ['local_eslint']
~~~

We also should notice that we must install the dependency of ESLint in the same place with ESLint(global or local).
