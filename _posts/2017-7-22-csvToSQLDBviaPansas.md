---
layout: post
title: Create database tables from csv files using Pandas to guess the column types.
tags: SQL Pandas Python CSV bash
---

YMMV, but this worked for me. I was looking for a way to automate SQL table creation and data type identification, so wrote some Python to create some bash to do the table creations and add a primary index. I happened to be using this for the [kaggle 2015 Traffic Fatalities](https://www.kaggle.com/nhtsa/2015-traffic-fatalities) data from the National Highway Traffic Safety Administration, but it may be useful for similar csv to SQL table work.

```
#!/bin/env python3
#
# Utility to go from csv files to a bash
# script which ought to do SQL table creation and load
# and add a primary index.
#
# Given a directory of csv files which one wants
# to do a fresh load into PostgreSQL,
# list the files and 
# then use a read into Python pandas to guess the type,
# then map that type to a SQL type
# print out the necessary shell.
#
# To use:
# 1. fill in DB and CSV_PATH in this script.
# 2. run this and make the output script executable.
# ./list_and_show_types.py > loadem.sh
# chmod u+x loadem.sh
# 3. Sanity check
# less loadem.sh
# 4. Try to run it.
# ./loadem.sh
#
# Note: I've setup access to my database to not require
# a password.
#
# Chris Harwell


import datetime
import numpy as np
import os.path as path
import pandas as pd
from pandas.compat import StringIO
from subprocess import check_output
import warnings

DB = 'traffic1'
CSV_PATH = '/tmp/2015-traffic-fatalities/' 


def dinfoToSQL(s):
    """
    Convert some types from pandas dataframe info()
    to PostgresSQL.
    INPUT: string
    OUTPUT: string
    """
    s = s.replace('int64', 'INT')
    s = s.replace('object', 'VARCHAR')
    s = s.replace('float64', 'DOUBLE PRECISION')
    return s

# List the files and put them into a list.
files = check_output(['find', CSV_PATH, '-name', '*csv']).decode("utf8")
files = [f.strip() for f in files.split('\n') if len(f) > 0]

s = """#!/bin/bash
set -e
set -u
set -o pipefail

# sudo -u postgres createdb {}

psql95 {} <<EOF
""".format(DB, DB)
print(s)

for fn in files:
    fn_path, fn_base = path.split(fn)
    fn_base, fn_ext = path.splitext(fn_base)
    assert fn_ext == ".csv"
    # Use low_memory to avoid mixed types within a single column
    # memory_map=False to mmap files and speed up reads,
    # encoding kept from getting started with traffic guide.
    d = pd.read_csv(fn, encoding="ISO-8859-1",
                    low_memory=False,
                    memory_map=True)
    a = StringIO()
    d.info(buf=a, memory_usage=False, null_counts=False, verbose=True)
    contents = a.getvalue().split('\n')
    """
    An example
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 192730 entries, 0 to 192729
    Data columns (total 4 columns):
    STATE      int64
    ST_CASE    int64
    VEH_NO     int64
    MDAREAS    int64
    dtypes: int64(4)

    OK, let's translate to PostgreSQL tables
    int64 -> INT
    object -> VARCHAR
    float64 -> DOUBLE PRECISION
    """
    contents = contents[3:]   # drop the first lines.
    contents = contents[:-1]  # drop the last line.

    print('CREATE TABLE {} ('.format(fn_base))

    for line in contents[:-1]:
        line = dinfoToSQL(line) + ','
        print(line)

    last_line = contents[-1]
    line = dinfoToSQL(last_line) + ');'
    print(line)
    print('')
    print("COPY {} FROM '{}' DELIMITER ',' CSV HEADER;".format(fn_base, fn))
    print('')
    print('ALTER TABLE {} ADD COLUMN {}ID BIGSERIAL PRIMARY KEY;'
          .format(fn_base, fn_base))
    print('')
print('EOF')

```
