---
author: volodymyr
comments: true
date: 2015-02-19 00:56:00+00:00
layout: post
slug: working-with-csv-and-co
title: CSV &amp; Co &mdash; Analyzing The Data From The Legacy Systems
categories:
- Scripting Languages
tags:
- python
- clojure
- scripting
- awk
- bash
- csv
---

From time to time I need to analyze lots of data from legacy systems. Usually, it's
__CSV__ or some similar format that utilizes flat text files. When implementing integration
with such systems __it's important to know if input data is correct__ and find any inconsistencies
in it to understand the source of possible problems. Some people like trial-and-error path,
while I prefer more deterministic process. In this post I'm describing tools I use to work
with such files.


<!-- more -->


Issues often discovered in the input data
=========================================
The most popular problem I met almost every day is file format
that differs from the specification. For example:

* missing columns
* odd columns
* invalid column format (text instead of a numeric data, exceeded field length, invalid characters, etc)
* fixed column format instead of line-delimited and vice versa
* empty lines

Usually it's easy to catch any of these in case of small files. But what should we do if there are millions of rows in the CSV?


Determining number of columns in the CSV
---------------------------------------
We can easily __determine number of columns__ in the CSV or other similar format using some simple shell scripting.

*Example of an input file(number-of-columns.csv):*


	Column 1,Column2,Column3,Column4
	Value 1,Value 2,Value 3,Value 4
	Value 1,Value 2,Value 3,Value 4
	Value 1,Value 2,Value 3
	Value 1,Value 2,Value 3,Value 4,Value 5

*Possible solution:*

	awk -F\, '{print NF-1}' number-of-columns.csv | sort -u


It will print out on the console:

	2
	3
	4



How it works:

* `-F\,` tells awk to use `,` as a field separator
* `'{print NF-1}'` makes awk print number of separators on each line
* `sort -u` sorts all resulting values (number of columns) and returns only unique numbers. It allows us to find out how many different number of columns the file has. If we remove this part then we'll get N lines with values where N is the number of lines in the input file.


Everything looks perfectly, but there is a hidden problem that sometimes can cause
false-positive assumptions about file consistency. Imagine we have such CSV:

	Column 1,Column2,Column3,Column4
	Value 1,Value 2,Value 3,Value 4
	Value 1,Value 2,Value 3,Value 4
	Value 1,Value 2,"Fruits,Vegetables and Chocolate",Value 4

Running the command from above will return such results:

	3
	4

This can be easily solved by removing quoted values from the input:

	awk -F\, '{ gsub(/,".*",/, ",,") ;print NF-1 }' number-of-columns-false-positive.csv | sort -u

Under the hood:

* `gsub(/,".*",/, ",,")` part of the awk command means "replace quotes and everything inside them with nothing if quotes are between commas

It will work even in this case:

	Column 1,Column2,Column3,Column4
	Value 1,Value 2,Value 3,Value 4
	Value 1,Value 2,Value 3,Value 4
	Value 1,Value 2,"Fruits,
	Vegetables and Chocolate",Value 4

So we are OK with line breaks inside columns if they are quoted!

Finding lines with issues
----------------------
It can be done by adding a condition to awk:

	awk -F\, '{ gsub(/,".*",/, ",,") ; if (NF-1 != 3) print "Line " NR " has " NF-1 " columns"}' number-of-columns.csv

This command will produce:

	Line 4 has 2 columns
	Line 5 has 4 columns


Checking if data meets constraints
----------------------------------

Sometimes we need to be sure columns in the CSV or similar files meet some specific constraints, e.g. should be a number.
Let's take a look on a sample CSV file:


	Value 1,15,Value 3
	Value 1,17,Value 3
	Value 1,a23,Value 3
	Value 1,42,Value 3

Obviously, row number 4 contains an invalid value in the 2-nd column.
We can easily find such rows using awk:

	awk -F\, '{if ($2 !~ /^[0-9]+$/ ) print "Line " NR " has non-numeric value in the third column"}' numeric-data.csv

This snippet will give us the following output:

	Line 3 has non-numeric value in the third column



Multiple data feeds
-----------------------------------
My favourite type of issues is inconsistencies in data files produced by different legacy systems.
For example we have two systems - one provides data files with customer profiles and another one gives us a feed with security permissions:


![](/images/posts/LegacySystem.png)

Let's imagine we need to run those feeds only once - on the initial system creation. One feed will provide us list of users, while another - their permissions.
Most common issue I faced is that one feed can contain data that is not represented in another feed. For example, we have 20k users and 10k permission assignments or vice versa.
Such difference can be easily spotted by comparing number of records. But what if numbers of records are equal, but actual contents (e.g. user names) are different?

I'm happy to announce GNU/Linux has an overwhelming tool to check such data - it's called "__comm__"!


What tools to use?
========================
First of all, I should admit it doesn't make any difference. Just use everything you can.
If the tool allows to save you an hour - then go for it. But there are some tools that you should now
about since they will definitely save you a lot of time.

![](/images/posts/Tools.jpg)

So here is my TOP 12 Tools List:

* Bash &mdash; no comments
* wc &mdash; allows to determine number of lines/words/characters of the text files
* grep &mdash; your best friend if you need to find something inside the file
* cut &mdash; allows reducing scope of the analysis by selecting only some particular columns
* head/tail &mdash; no comments also
* comm &mdash; amazing tool which allows working with files as sets. See the section about it below
* sort &mdash; allows sorting text files contents and remove duplicates
* awk/sed &mdash; transforming a text is their cup of tea;
* Python Programming Language &mdash; allows writing small tools for your every day tasks
* Clojure &mdash; very expressive language that makes data processing really easy
* Graphviz Tools &mdash; sometimes it's better to see it once
* Libre Office Calc &mdash; best friend of the CSV files



What tools do you use to get rid of manual work? Don't hesitate to share your experience in comments!
