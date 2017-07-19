---
layout: post
title: Local data 2016 School District of the Chathams Salary info
tags: school salary pandas python
---


Recently I was looking for a source of data on the web which would be amenable to learning how to use the [Python](http://www.python.org) [Beautiful Soup library](https://www.crummy.com/software/BeautifulSoup/) for web scraping.  Ironically I found many web sites which provided csv or other raw data, but that wouldn't make use of web scraping.  On the other end of the spectrum I found web sites with intriguing data all hidden behind JavaScript layers, which would call for a tool more like selenium and I wanted to use Beautiful Soup.  Anyway, after much searching I found
[Data Universe of Asbury Park Press](http://php.app.com/agent/educationstaff) which not only some data in HTML, but had a nice sampling.  That is some data in tables, some data in just text and all spread around multiple pages.

The data covered educational staff salaries for all NJ public schools in 2016.  I chose to use a [subset](http://php.app.com/agent/educationstaff/search?last_name=&first_name=&county=MORRIS&district=SCH+DIST+OF+THE+CHATHAMS&school=).  of the data which had more meaning to me - just that from Morris County's School District of the Chathams where my kids go.  First, I browsed through the data and inspected it for regularness and pattern which would allow a simple program to digest it.  Luckily, it looked like the site followed a regular, predictable pattern.  

The tools used include
1. Python 3.5.3  [Python](http://www.python.org)   
2. Python3-requests-2.10.0-4.fc25.noarch [Requests: HTTP for humans](http://docs.python-requests.org/en/master/) 
3. Python3-beautifulsoup4-4.6.0-1.fc25.noarch  [Beautiful Soup library](https://www.crummy.com/software/BeautifulSoup/)  
4. python3-pandas-0.19.0-1.fc25.x86_64 [Pandas](http://pandas.pydata.org/)  


The overview listing pages with maybe ten staff per page had the page numbers were nicely appended to the URLs.  

```Python
base_url = 'http://php.app.com/agent/educationstaff/search/page:'
minpage = 1
maxpage = 13873
...
    for i in range(minpage, maxpage + 1):
        url = base_url + str(i)
        page = get_response(url)
        data = get_header_and_salary_data(page)
...
```

I started down that route, but after a few pages hit ctrl-C and interrupted it because, it turned out, as it often does, that I wanted just a little more information than was available on the overview listing pages.  We were to use this data for a linear regression model and I was curious how experience would relate to salary.  Fortunately, there were detail pages for each staff member and those had an id for each.  Also, from what I could tell the ids were ranges where all those in a county would be nearby - quite possibly this wouldn't work out for all districts or counties, but it seemed to for Morris County and School District of the Chathams.  Doing a quick sanity check, thinking about the number of schools, grades and students it seemed like a little less than 400 would be correct.  We got 383 or so by picking a range to fetch and store to files and then storing those to files with pickling and then later filtering upon read.

There were 13,873 pages in the big overview list for all of NJ - doable, but I was only interested in a subset.  By narrowing down to the page id range of 91953-92545 got it down to 592, then filtering down to 396.  Found a duplicate and dropped one row that wasn't complete (df.dropna(subset=[...]) and got down to 393.  

```Python
base_url = 'http://php.app.com/agent/educationstaff/details/'
minpage=int(91953)
maxpage=int(92545)
...
data = {}
salaries = []
for i in range(minpage, maxpage + 1):
	url = base_url + str(i)
	page = get_response(url)
	headers, next_salaries = get_employee_details(page)
	salaries.extend(next_salaries)

# Combine the data into a dictionary with
# headers and salaries.
data = {
        'headers': headers,
        'salaries': salaries
       }

fn = base_fn + str(minpage) + '_' + str(maxpage) + '.pkl'
pickle_data(data, fn=fn)
```

We then had these columns to use:
```Python
for idx, val in enumerate(df4.columns):
    print('{} column: {}'.format(idx + 1, val))
dprint('row * column: {}'.format(df4.shape))

1 column: first
2 column: last
3 column: salary
4 column: county
5 column: district
6 column: experience_district
7 column: school
8 column: experience_nj
9 column: primary_job
10 column: experience_total
11 column: fte
12 column: subcategory
13 column: certificate
14 column: highly_qualified
15 column: teaching_route
debug: row * column: (393, 15)
```

**2016 Teacher Salary Data Exploration**
**School District of the Chathams**
**Chatham, NJ**

Looking at only the salary we found a wide range from 21 to 164k
and that the middle was around 65k with the average slightly higher
at 71k, Interesting.

```Python
print('average: {:,.0f}'.format(sal.mean()))
print('middle: {:,.0f}'.format(sal.median()))
print('min,max range: {:,.0f} through {:,.0f}'.format(sal.min(), sal.max()))
print('iqr range for dispersion: {:,.0f}'.format(ss.iqr(sal)))

average: 71,931
middle: 65,035
min,max range: 21,236 through 164,303
iqr range for dispersion: 22,884

ss.describe(sal)
Out[16]:
DescribeResult(nobs=393, minmax=(21236.0, 164303.0), mean=71931.325699745546, variance=475049866.43446535, skewness=1.1999130808179492, kurtosis=2.1010797452043084)
```

Dividing the data up into small ranges or bins and then putting them on a bar chart gives a visual of the distribution.

![Histogram](/images/data_universe_edu_sal_ch_hist.png "Histogram")
