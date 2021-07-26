---
layout: blog
title: "Top 5 Favorite Python Tricks"
date: 2021-02-12
background: '/img/blog/post 3.jpg'
---

In this post, I will outline my top 5 favorite tricks that _can_ make your Python code more "pythonic." By "pythonic," we mean that your code corresponds to the principles laid out in [PEP](https://www.python.org/dev/peps/). [The Zen of Python](https://www.python.org/dev/peps/pep-0020/) succinctly explains some of the guiding principles for Python's design into a collection of aphorisms, below are the first seven:

```
Beautiful is better than ugly.

Explicit is better than implicit.

Simple is better than complex.

Complex is better than complicated.

Flat is better than nested.

Sparse is better than dense.

Readability counts.
```

Fun fact, you can see the aphorisms in Python with the following command: 

```
import this
```

Alright, let's get started.

## List comprehensions

Anyone coming out of a Python 101 class should have at least learned about lists and for loops, but let's discuss them very briefly before we talk about list comprehensions.

Lists are simple, ordered arrays that can contain any Python object, including integers, strings, and even whole machine learning models. The length and content of a list is limited in practice only by the memory of the system Python is running on.

For loops iterate over a sequence (often a list, but could also be something like a string or a dictionary). They are endlessly useful, though they are sometimes not the best way to work with certain kinds of sequences (such as manipulating data in pandas).

List comprehensions combine lists and for loops into a single-line statement.

## If not

3. 