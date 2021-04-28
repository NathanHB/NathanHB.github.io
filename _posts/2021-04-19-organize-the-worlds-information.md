---
title: Organize the world's information
date: April, 09 2021
author: Nathan Habib
navlevel: header
---

Work in progress..

## Bibliography
* [The Anatomy of a Large-Scale Hypertextual Web Search Engine](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/334.pdf)
* [Extracting Patterns and Relations from the World Wide Web](http://ilpubs.stanford.edu:8090/421/1/1999-65.pdf)


* The web is growing
* The issue was scaling
    * Hardware scales up
    * Disk seek time is still slow
* Google tries to solve that
* How


Back in the 1990's, the internet was growing both in size and popularity. This
called the need for efficient page retrieving systems that could search trough
millions of web pages and answers millions of queries.

<img src="assets/images/Ancientlibraryalex.jpg"
     alt="Library of Alexandria"
     style=""/>

First came Archie.

In 1990 [Archie](https://www.youtube.com/watch?v=TdM3MNq9YL0) (short for file
archive) was a search engine that worked by going  through a list of public
File Transfer Protocol sites.  It then creates an index of the FTP files.  We
used a client to enter keywords and the server would send back a list of
filenames that we then had to download to see the content of. Archie used
_grep_ to look for the keywords in the filenames.  This technique was of course
very slow and required to databases to be updated by hand. Moreover the quality
of the search was far from good, if the title was not a perfect match it was
near impossible to find anything, because using matching keywords in title does
not give us anything about the quality of the page or even its content.

The paradigm of keyword matching to find web pages continued until around 1995.
With various other search engine, (LIST SEARCH ENGINES).

Then came Google.

> History about Sergey and Lawrence.

Google had the goal to be fast, efficient, reliable in its search results and
above all, scalable.

So how did they achieve this?
