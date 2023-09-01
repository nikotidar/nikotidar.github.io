---
layout: post
title: "Find big files on Linux"
author: "rndtx"
---

What's that, young man? Oh, you're woman? I **am** sorry... Anyway;  what was it you wanted? You're running out of disk space and you want to find all the big files on your linux machine so you can blindly delete them? Try this:

```sh
find / -type f -size +20000k -exec ls -lh {} \; | awk '{ print $9 ": " $5 }'
```

You're welcome :)