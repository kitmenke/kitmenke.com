---
tags: ["Python"]
title: Python Flask Bash on Windows
author: Kit
categories: []
date: 2017-09-14T16:19:54-05:00
draft: true
---

http://flask.pocoo.org/docs/0.12/installation/#installation

sudo apt-get install python-setuptools
sudo easy_install pip
sudo pip install virtualenv

kitmenke@DESKTOP-HQ4KLVM:~$ mkdir myproject
kitmenke@DESKTOP-HQ4KLVM:~$ cd myproject/
kitmenke@DESKTOP-HQ4KLVM:~/myproject$ virtualenv venv
New python executable in /home/kitmenke/myproject/venv/bin/python
Installing setuptools, pip, wheel...done.
kitmenke@DESKTOP-HQ4KLVM:~/myproject$ . venv/bin/activate
(venv) kitmenke@DESKTOP-HQ4KLVM:~/myproject$ python --version
Python 2.7.6
(venv) kitmenke@DESKTOP-HQ4KLVM:~/myproject$ pip install Flask

(venv) kitmenke@DESKTOP-HQ4KLVM:~/myproject$ vim hello.py
(venv) kitmenke@DESKTOP-HQ4KLVM:~/myproject$ export FLASK_APP=hello.py
(venv) kitmenke@DESKTOP-HQ4KLVM:~/myproject$ flask run
 * Serving Flask app "hello"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

 deactivate

kitmenke@DESKTOP-HQ4KLVM:~/Code/python/youtubeml$ . venv/bin/activate

pip install sklearn
pip install quandl
pip install pandas

import pandas as pd
import quandl
df = quandl.get('WIKI/GOOGL')
print df.head
df2 = df[['Adj. Open', 'Adj. H+++igh', 'Adj. Low', 'Adj. Close', 'Adj. Volume']]
df2['HL_PCT'] = (df['Adj. High'] - df['Adj. Close']) / df['Adj. Close'] * 100.0
df2['PCT_change'] = (df['Adj. Close'] - df['Adj. Open']) / df['Adj. Open'] * 100.0