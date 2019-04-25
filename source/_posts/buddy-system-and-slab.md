---
title: 伙伴系统、slab
date: 2019-04-09 22:10:56
tags: os
---
内核内存管理的一项重要工作就是如何在频繁申请释放内存的情况下，避免碎片的产生。这就要求内核采取灵活而恰当的内存分配策略。通常，内存分配一般有两种情况：大对象（大的连续空间分配）、小对象（小的空间分配）。针对不同的需求，Linux分别采取了伙伴系统算法和SLAB进行内存分配。  
### 伙伴系统：
把所有的空闲页框分为11个块链表，每个块链表中的结点分别是大小为1，2，4，8，16，32，64，128，256，512和1024个连续页框的页框块。最大的页框块包含1024个连续页框，对应4MB大小的连续内存。假设要申请一个256个页框的块，则先从结点为256个连续页框块的链表中查找空闲块，如果没有，就去512个页框的链表中找，找到了则将页框块分为2个256个页框的块，一个分配给应用，另外一个移到256个页框的链表中。如果512个页框的链表中仍没有空闲块，继续向1024个页框的链表查找—分割—分配和转移。如果仍然没有，则返回错误。使用过的页框块在释放时，会主动将两个连续的页框块合并为一个较大的页框块，然后作为结点插入相应规格的链表中。  
伙伴系统很好地解决了外部碎片（页框之间的碎片）问题。  
<!-- more -->
### SLAB：
伙伴系统分配内存时是基于页框为单位的，比较大。如果是几十个字节的小内存分配怎么办呢？此时就需要用SLAB机制。slab分配器是基于对象进行管理的，所谓的对象就是内核中的数据结构（例如：task_struct,file_struct 等）。相同类型的对象归为一类，每当要申请这样一个对象时，slab分配器就从一个slab列表中分配一个这样大小的单元出去，而当要释放时，将其重新保存在该列表中，而不是直接返回给伙伴系统，从而避免内部碎片。slab分配器并不丢弃已经分配的对象，而是释放并把它们保存在内存中。slab分配对象时，会使用最近释放的对象的内存块，因此其驻留在cpu高速缓存中的概率会大大提高。也就是说：在内存中维护一个slab列表，列表项对应这各种数据结构大小的内存块；当有小对象申请内存时，直接从slab列表中找到对象类型的列表项，把相应大小的内存分配出去；对象用完后，释放掉对象并把对象所占的内存块归还到slab列表以供下一个同类型的对象使用。  
同理，由于SLAB是对于小对象的内存分配，很好地解决了页框内部的内存分配产生的碎片（内部碎片）问题。  
