---
layout: post
categories: python
title: Game of Life
tags: [python]
---

I recently learned about the existence of Conway's [Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) and it fascinated me. It basically is a grid on which each cell can be either dead or alive and after one step or generation a simple set of rules apply:

* Any live cell with two or three live neighbors survives.
* Any dead cell with three live neighbors becomes a live cell.
* All other live cells die.

<!--more-->

These very basic rules can lead to a self evolving pattern which can be quite curious to watch. There are many websites which allow you to play around with it by setting the initial conditions and watching it evolve. In addition to this, the game of life is surprising Turing complete so by setting it carefully you would be able to compute anything with it.

This morning I decided to do a simple coding exercise and create a simple implementation of the game of life in Python: [Game of Life](https://github.com/samueldiethelm/life)

