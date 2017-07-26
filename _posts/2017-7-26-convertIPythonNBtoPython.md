---
layout: post
title: command line Python script for converting IPython to Python
tags: IPython PythonconvertIPythonNBtoPython
---

I am quite liking the Jupyter/IPython notebook approach to early stage development and exploration, but there are times when I just want the .py version.  It is nice to quickly refer to with vi or do some other testing.  Anyway, I decided to write a script to convert the notebook format files to python files using the existing libraries.  First, we gather a list of likely files using the find command and the .pynb.  The nice thing about using find is that if we put a directory in TOP_PATH it will do a recursive check into all directories below that too.  We do some manipulations in order to get the path name and create an output file name.  We check to make sure that the input file actually has something in it, that is that it is non-zero in length.  I considered validating that the file was JSON, but ended up just putting a try, except clause in the convertNotebook routing, which is just an amalgamation of two stack exchange answers.  Neither worked for me on Fedora 25, so I combined and adapted them. We also don't create a new one if the file already exists.  You may want to add some time stamp comparison, but I chose to just skip the conversion altogether if it exists. That is it. Let me know if this helps you.

```
#!/bin/env python3
#
# Purpose: convert ipynb files to py if they do not already exist
# in all directories under TOP_PATH.
#
# Chris Harwell
# Jul 26, 2017
#
# ADJUST TOP_PATH
#


from IPython import nbformat
from IPython.nbconvert import PythonExporter
import os
from subprocess import check_output


# TOP_PATH = '/home/crh/git/nyc17_ds12/postgres-from-jupyter-test.ipynb'
TOP_PATH = '/home/crh/git/nyc17_ds12'


def convertNotebook(notebookPath, modulePath):

    try:
        with open(notebookPath) as fh:
            nb = nbformat.read(fh, nbformat.NO_CONVERT)

        exporter = PythonExporter()
        source, meta = exporter.from_notebook_node(nb)

        with open(modulePath, 'w+') as fh:
            fh.writelines(source)
    except nbformat.reader.NotJSONError:
        print('JSON object issue')
        print('skipping {}'.format(notebookPath))


if __name__ == '__main__':
    # List the files and put them into a list.
    files = check_output(['find', TOP_PATH, '-name', '*ipynb']).decode("utf8")
    files = [f.strip() for f in files.split('\n') if len(f) > 0]
    for fn in files:
        fn_path, fn_in = os.path.split(fn)
        fn_base, fn_ext = os.path.splitext(fn_in)
        assert fn_ext == ".ipynb"
        fn_out = fn_base + '.py'
        full_in = fn_path + '/' + fn_in
        full_out = fn_path + '/' + fn_out
        if not os.path.getsize(full_in):
            print('skipping zero length file {}'.format(full_in))
        else:
            # Be careful, save time,
            # do not over-write an already existing file.
            if os.path.exists(full_out):
                print('skipping {} already exists'.format(full_out))
            else:
                print('read {}'.format(full_in))
                convertNotebook(full_in, full_out)
                print('wrote {}'.format(full_out))

```
