---
layout: post
title: Data science tech support episode 1
---

Trying to do data science with Python and Jupyter notebook on a Mac and find jupyter doesn't want to automatically open your notebook like it used to? Do you not want to copy and paste, but would rather the URL opens in the web browser tab?

That is, if you see this or similar error message and don't want to
```console
$ jupyter notebook
.... 0:97: execution error: "http://localhost:8888/tree?    
token=c9c72f63d8482d80ee7dfd7aacc70c5d3048b6a7626b2859" doesn't understand the "open location" message. (-1708)  
```

The solution is to explicitly set the browser in
```console
~/.jupyter/jupyter_notebook_config.py:

c.NotebookApp.browser = u'Safari'
```

Apparently a recent (spring/summer 2017) Python update changed the default used by the webbrowser library.  That is it used to be that 
```
import webbrowser
browser=webbrowser.get(None)
browser.open("http://python.org")
```
reproduces that same error:
```
  0:33: execution error: "http://python.org" doesn’t understand the “open location” message. (-1708)
Out[4]: False
```

But, if you actually specify "Safari" in that webbrowser.get call it works as expected and that is what the above "fix" does.  Presumably you could specify whatever executable you wanted that could handle being called and opening a URL for you. u'lynx' anyone? ;>

I just added that configuration file because it wasn't present.  But, from the [docs](http://jupyter-notebook.readthedocs.io/en/latest/index.html),  it looks like you can generate a configuration file if you want using a command like
```console
jupyter notebook --generate-config
```
