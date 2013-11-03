---
layout: post
title: "Emacs and GChat"
date: 2013-11-02 19:08
comments: true
categories: 
  - emacs
  - gchat
---

I was chatting with a friend on GChat today.  I found having to context switch to my browser so much to be distracting, and half-jokingly and off-handedly said: "I should setup my Emacs with GChat".

A little Googling led me to [Emacs Wiki: GoogleTalk](http://www.emacswiki.org/emacs/GoogleTalk).

These are the steps I used to get GChat running pretty quickly:

*  `M-x package-list-packages`, installing 'jabber'
*  add the below to my `init.el`
 
``` scheme
    (setq jabber-account-list           
      '(("my.email@gmail.com"                 
         (:network-server . "talk.google.com")           
         (:connection-type . ssl))))                     
``` 
 
*  `brew install gnutls` since I'm on OSX
*  `M-x jabber-connect-all`, entering in my password at the prompt
*  Then I can switch to the `*-jabber-roster-*` buffer, search for my friend's name in the list, and hit `RET` to start a chat with them.
