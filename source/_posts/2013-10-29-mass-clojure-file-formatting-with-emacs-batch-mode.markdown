---
layout: post
title: "Mass Clojure File Formatting With Emacs Batch Mode"
date: 2013-10-29 00:51
comments: true
categories: 
  - clojure
  - emacs
  - shell
  - elisp
---

I do not profess to be a master of Emacs Lisp.  But after some time googling I figured out how I can mass code-format the code on some of our large projects at work.

It is pretty likely that you may need to tweak this to get it to work on your machine. Specifically, you'll have to point the script to the correct location of you clojure-mode installation.

This script calls `emacs` in [Emacs in --batch mode](http://www.emacswiki.org/emacs/BatchMode), which causes Emacs to run non-interacively.  We can use `--batch` to allow us to write scripts using Emacs.  Awesome, I know. :-)

``` bash
#!/bin/bash

find "/path/to/your/project/." | while read file
do
    if [[ "$file" =~ ^.*\.clj$ ]]; then
        emacs "$file" \
              --batch \
              --eval="(progn 
                        (load \"~/.emacs.d/elpa/clojure-mode-2.0.0/clojure-mode\")
                        (require 'clojure-mode) 
                        (clojure-mode) 
                        (indent-region (point-min) (point-max)) 
                        (save-buffer))"
    fi
done
```
