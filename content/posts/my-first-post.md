---
title: "organize the worlds information"
date: 2022-08-02T18:55:47+01:00
draft: false
---

Back in the 1990s, the internet was growing both in size and popularity. This
called the need for efficient page retrieving systems that could search trough
millions of web pages and answers millions of queries. From 1993 to 1995, the
web roughly double every 3 months, going from 130 websites crawled by the
_World Wide Web Wanderer_ in June 1993 to 100,000 pages in January 1996, @MGray.

![Library of Alexandria](https://upload.wikimedia.org/wikipedia/commons/6/64/Ancientlibraryalex.jpg)

First came Archie.

## The keyword matching era

In 1990 [Archie](https://www.youtube.com/watch?v=TdM3MNq9YL0) (short for file
archive) was a search engine that worked by going  through a list of public
File Transfer Protocol sites.  It then created an index of the FTP files.  We
used a client to enter keywords and the server would then send back a list of
filenames that we then had to download to see the content of. Archie used
_grep_ to look for the keywords in the filenames.  This technique was of course
very slow and required databases to be updated by hand. Moreover, the quality
of the search was far from good, if the title was not a perfect match it was
near impossible to find anything, because using matching keywords in title does
not give us anything about the quality of the page or even its content.

The paradigm of keyword matching to find web pages continued until around 1995.
With various other search engine: YAHOO!, LYCOS.

Then came Google.

## The Google era

In 1996, two PhD students at Stanford University (Larry Page and Sergey Brin)
began doing research on search engines. They had two main complaints concerning
the search engines that were widely used at that time:
First, they worked by counting how many times a keyword appeared on a page and
ranking result accordingly, second, research in the field was underdeveloped
and therefore commercial search engine were very difficult to do experiments
with.
Their work eventually led to the creation of Google, and they published their
results for the first time in _The anatomy of a large-scale hypertextual web
search engine_, @brin1998anatomy.

Google had to be fast, efficient, reliable in its search results and
above all, scalable.

So how did they achieve this?

### PageRank

The biggest issue with current search engines was the quality of the results.
To address this, Page and Brin came up with the idea of using a probability
distribution. Each page had a probability that represented how likely a user is
to end up on a page if he were put on a random web page and kept clicking
random links, @brin1998anatomy @wilf2001searching.

That means that the rank of a website is proportional to the rank of the sum of
websites that links to it. To represent the relationship between $n$ websites
lets use a $n \times n$ matrix, $A$, where the column $j$ of the $i^{th}$ row is $1$
if the website $j$ has a link to $i$ and $0$ otherwise.
The importance of a page $i$ is then $x_i$ and can then be written as:

$$
x_i = \sum_{j=1}^{n}{a_{i,j}x_j}
$$

Where $a_{i,j}$ is the value of $A$ at the index $i,j$.

That problem is an eigenvalue and eigenvector problem (that is, it is
equivalent to finding the eigenvectors and eigenvalues of the matrix $A$). All you
have to do is take the eigenvector corresponding to the highest eigenvalue of
the matrix $A$. Then, the most important website is the one with the largest
entry in that vector.

That is the very basic explanation, PageRank uses a slightly modified version:

For example, it uses a damping factor $d \in [0,1]$ that represents the
probability that a user gets bored and requests a new random page (remember our
user clicking on random links), this damping factor is found by trial and
error. Users would rate the quality of their search and hyperparameter
(including the damping factor would be updated accordingly). Also, the rank of
the pages that links to our page are normalized by the number of links leaving
that page.

$$
pageRank(P) = (1-d) + d\left(\sum_{j=1}^k{\frac{pageRank(T_j)}{C(T_j)}}\right)
$$

Where $T_1 ... T_k$ are the web pages that link to $P$ and $C(T_j)$ is the number of links
going out of $T_j$.


That is pretty much it for the pageRank algorithm however, Google uses many
more technics to evaluate a search query and rank the results.

### Indexing the web


To be able to return excellent results with very low latency you have to find
an optimal way to store and evaluate the data you are dealing with.
Here how Google does it:

First they send web crawlers to downloads all the web pages from URL found in
a URL file list.

Then these pages are compressed into a repository.

Every page is then parsed and, for every word in the page we make a list of all
the occurrences of that word, (a hit list). The hit list contains for each
occurrence, the position of the word, the capitalization, the font and size of
the word.

An anchor-file is also made, it consists  of a list of all the URLs on the
page, the anchor file is then given to the crawlers for download.

The anchor is also used to compute the matrix that we will use for the pageRank
algorithm.

![Simplified search engine diagram](https://raw.githubusercontent.com/NathanHB/NathanHB.github.io/gh-pages/assets/images/search_engine.svg)

Now that we have for each page a list of words and information about their
position and function in the page, we can use it to evaluate a search query.

### Answering a search query

Answering a query is pretty simple, first let's look at the case of
a one word query:

To rank a page, Google uses the hit list for that word. The hit list
information are weighted with arbitrary weights, (the process of determining
the right weight is empirical, which means that they were updated depending on
user's feedback), the score given by this method is called the IR score.

For example is the query is: __banana__, Google will look __banana__'s hit list
for each web page. If for one web page the word appears more often but with
a very small font and in a footer, it will score less than a page with fewer
occurrences of the word but with the word appearing the title of the page.

For multiple word queries, the idea is the same, each word are evaluated
separately, but more weights are given if the words appear closer to each other.

Finally, the IR score is combined with the pageRank to give the final page rank
of the page.


## What about today?

What I explained was the way Google used to work, in the early 2000s, since
then Google has made significant changes to its algorithm, from the addition of
machine learning, to _NoFollow_ links.

Today Google's algorithm is the most efficient and complicated in the world, as
it serves $3.5$ billions search queries per day with great accuracy.

## References


