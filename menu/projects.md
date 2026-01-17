---
layout: page
title: My projects
permalink: /projects
---

# WIP

I'm currently working on improving [orienboard](https://github.com/maleksware/orienboard) and fixing two annoying printing bugs in [MeOS](https://www.melin.nu/meos/en/), the orienteering system used across Australia and, in particular, by Orienteering ACT.

# Completed/published

## [parsers](https://github.com/maleksware/parsers) (Haskell)

A couple of toy parsers for JSON, lambda expressions and others (WIP). Made with Haskell from scratch.

## orienboard (Flask, SocketIO)

A system for displaying live orienteering results on independent screens. Formerly [meshO Leaderboard](https://mesho.live) (my system was used until transition to a different stack).

The main priorities for this system are reliability in bush conditions and complete usability offline (critical for working in Australian outbacks, where orienteering events are normally held). The system is designed to be extendable to accept input from many platforms, but the only one currently supported is the MeOS information server.

_The code is currently being reworked to satisfy at least some professional codebase standards, and will be released soon._

## uc-labels (Python, Typst)

A small script for automatically converting CSV data about museum items into `.typ` files for University of Canberra Art Collection labels. Allows to generate new labels in big batches without typographical issues.

## [hash-segtree](https://github.com/maleksware/hash-segtree) (C++)

> Given a long string ($10^6$ characters) and queries of type `set all elements with indices between l and r to character c` and `compare substrings [l1, r1] and [l2, r2]`, what is a relatively simple and efficient way to do so? 

A short paper with the exposition of the segment tree that solves this problem, as well as a tested implementation.

## [upper-envelope](https://github.com/maleksware/cf-recommender) (C++, Python)

> Given many function graphs defined by values at points (assuming linearity between the points), what is their upper envelope?

Includes implementations in C++ and Python. I don't think these are particularly efficient, but on problems like this, where the size of the _answer_ is $\mathcal{O}(n^2)$, it's hard to come up with something that is not an order of magnitude less efficient.

## [codeforces-recommender](https://github.com/maleksware/cf-recommender) (JS)

A single page that recommends Codeforces problems based on account statistics. I know, everyone in the competitive programming world has made one of these.

