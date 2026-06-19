---
author: Kit
categories:
- Notes
date: 2018-09-25T20:05:02Z
title: How to list every Maven Dependency for a Project
---

I was working a Scala big data project with a lot of dependencies. Most of the time the dependencies exist on the server
already or get packaged up inside an uber jar. In this case, the server was missing the dependencies and I didn't want 
to package everything in the uber jar due to super slow upload speeds. This combined with the need to do a bunch of
trial and error meant I wanted to upload all the dependencies to the server and then just tweak my few lines of code. I
ended up using a nice maven command combined with a few lines of python to gather all of them up in one place. 

To generate the list of all dependencies:

```
mvn dependency:build-classpath
```

Then, a few lines of python to copy all the dependencies to a folder that I could then zip up and copy out to the
server.

```python
import os
import shutil

f = open("deps.txt")
deps = list(f)
dir = "lib"
os.makedirs(dir)
for dep in deps:
  shutil.copy(dep.strip(), dir)
```