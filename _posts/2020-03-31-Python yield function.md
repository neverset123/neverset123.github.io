---
layout:     post
title:      python yield function
subtitle:   
date:       2020-03-31
author:     neverset
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - python
---

the yield is able to execute an expression and return a value in time sequence rather than calculate them all and return in list; mostly used as generator

## yield example
### return multivalue in sequence

    def func():
        yield 1
        yield 2
        yield 3

### iterator generator

    def iterator_gen():
        i=1
        while True:
            yield i*i
            i+=1

    for a in iterator_gen():
        if a<100:
            break
        print(a)

### yield from

    def get_content(entry):
    for block in entry.get_blocks():
        yield block
    #can be refactored with yield from to accelerate 15%
    def get_content(entry):
        yield from entry.get_blocks()
