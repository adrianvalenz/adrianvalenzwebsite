---
layout: post
title: Custom shell function in .zshrc to search strings
subtitle: This function will search any string within your files in your current directory and sub-directories
date: 2022-03-25 14:31:02 -0700
categories: code
author: adrian
---

```
seek() {
  if [ "$1" != "" ]
  then
    grep -H -r --exclude-dir=node_modules "$1" * | less
  else
    echo "need to type in a string"
  fi
}
```

I use this function when I join a new existing project and I need to get familiar with the codebase. I got tired of typing in `grep -H -r 'string' * | less` so I created this little function to make the process quicker.

Now we just type in `seek custom_method` and a list of files will appear in a project that contain that method or whatever string you are looking for. This is helpful when searching file names is not enough.

I'm pretty excited about this one. I hope it helps you out!
