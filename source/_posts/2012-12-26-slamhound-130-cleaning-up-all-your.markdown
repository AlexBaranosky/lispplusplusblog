---
layout: post
title: "Slamhound 1.3.0 - Cleaning up all your namespaces"
date: 2012-12-26T17:22:00-08:00
comments: false
---

Cleaning up namespaces in Clojure is a chore.  So noone does it, and the namespaces get worse.

Enter [Slamhound](https://github.com/technomancy/slamhound).  Slamhound's a library by Phil Hagelberg (the [Leiningen](http://leiningen.org/) guy) that attempts to reconstruct a namespace for a file.

For a while now Slamhound has been pretty unusable, do to it not having a good heuristic for figuring out which namespace to use for a require/as clause. For me, it would fail on most real-world namespaces I tried it on.

Over Christmas break I decided to get it working again, because I was pretty sick of fixing up legacy namespaces by hand.  It now works, and also, as an added bonus, converts all your uses to requires, as per the Clojure 1.4 good practice.

The original Slamhound `reconstruct` function returned a string representing the new namespace, but it was still tedious to run it on 50+ namespaces and copy-n-paste the new namespaces in.

For this reason, I decided to automate the process.  Long story short, the slam.hound namespace in version 1.3.0 now also has a -main function that takes a file or directory and replaces the namespace and writes it back to the file, or to all the files in the directory.  Like magic.

Also, if you're on Leiningen 2 you can define a partial alias:

``` clojure
:aliases {"slamhound" ["run" "-m" "slam.hound"]}
```

and then call it from within your project's root directory:

``` bash
$ lein slamhound . 
```

Then take a look at the diff for your repo.  You'll see a lot of files have been altered.

Caveats: Slamhound is still not perfect. It can lose metadata, or comments in the namespace form, and sometimes it picks the wrong dependencies to use in a namespace.
