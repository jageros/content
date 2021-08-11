---
type: "posts"
author: "jager"
title: "queue golang实现"
date: "2021-08-03"
tags: [
"go",
"queue",
]
---

> 本文介绍go语言``gopkg.in/eapache/queue``包的原理，代码解析以及使用方法

<!--more-->

## 说明
> 本文主要介绍golang queue 库：``gopkg.in/eapache/queue``的实现原理和使用。<br>
> 第三方开源库获取： ``go get gopkg.in/eapache/queue.v1`` <br>
> 使用时导入： ``import "gopkg.in/eapache/queue.v1"``

## 原理
> 队列的缓存区为环形，实际是一个数组，当队头元素取出后，队头标志会往后移动，空出的位置可存储队尾新加元素，当队尾增加元素时，队尾标志向后移动，当缓存区我末尾没有位置时，队尾标志从0开始，复用队头取出元素空出的位置，直到当缓存区满后，缓存区空间成倍增长【ps: resize中的``newBuf := make([]interface{}, q.count<<1)``】，队列元素复制到缓存区的开始区域，当队列元素数量减为缓存区1/4的时候，缓存区空间重置为元素个数的两倍长度，同时队列元素复制到缓存区的开始区域。

## 函数功能
    queue.New()返回一个长度为16的空队列
    queue.Add(elem)为入队，即向队列中添加元素
    queue.Remove()为出队列操作，返回一个队头元素
    queue.Peek()返回队头元素，但元素不出队
    queue.Get(i)返回队列中第i个元素，但不从队列中删除


## 源码解读
ps： 该库主要是这个文件： ``queue.go``，源码不多，开发者可根据需求修改后使用
```go
/*
Package queue provides a fast, ring-buffer queue based on the version suggested by Dariusz Górecki.
Using this instead of other, simpler, queue implementations (slice+append or linked list) provides
substantial memory and time benefits, and fewer GC pauses.

The queue implemented here is as fast as it is for an additional reason: it is *not* thread-safe.
*/
package queue

// minQueueLen is smallest capacity that queue may have.
// Must be power of 2 for bitwise modulus: x % n == x & (n - 1).
const minQueueLen = 16   // 队列缓存区最小长度

// Queue represents a single instance of the queue data structure.
type Queue struct {
	buf               []interface{}  // 缓存区
	head, tail, count int   // 队头下标，队尾下标，队列长度
}

// New constructs and returns a new Queue.
func New() *Queue {
	return &Queue{
		buf: make([]interface{}, minQueueLen),   // 返回最小长度缓存的空队列
	}
}

// Length returns the number of elements currently stored in the queue.
func (q *Queue) Length() int {
	return q.count   // 队列长度
}

// resizes the queue to fit exactly twice its current contents
// this can result in shrinking if the queue is less than half-full
func (q *Queue) resize() {
	newBuf := make([]interface{}, q.count<<1)  // 将队列缓存区长度重设为队列长度的两倍

    // 复制元素到缓存区开始区域
	if q.tail > q.head {
		copy(newBuf, q.buf[q.head:q.tail])
	} else {
	    // 因为是环形缓存区，需要先复制缓存区后半部分，即队列前半部分，再复制缓存区前半部分，即队尾部分
		n := copy(newBuf, q.buf[q.head:])
		copy(newBuf[n:], q.buf[:q.tail])
	}

	q.head = 0  // 队头下标置0
	q.tail = q.count  // 队尾下标为队列长度（此下标为下一个元素插入的下标）
	q.buf = newBuf  // 新队列
}

// Add puts an element on the end of the queue.
func (q *Queue) Add(elem interface{}) {
	if q.count == len(q.buf) {
		q.resize()  // 新元素入队之前，当队列长度等于缓存区长度时，缓存区长度重设为两个队列长度
	}

	q.buf[q.tail] = elem // 入队
	// bitwise modulus
	q.tail = (q.tail + 1) & (len(q.buf) - 1)  // 队尾下标在缓存环中移动一位
	q.count++  // 队列长度+1
}

// Peek returns the element at the head of the queue. This call panics
// if the queue is empty.
func (q *Queue) Peek() interface{} {
	if q.count <= 0 {
		panic("queue: Peek() called on empty queue")
	}
	return q.buf[q.head]   // 返回队头元素，不出队
}

// Get returns the element at index i in the queue. If the index is
// invalid, the call will panic. This method accepts both positive and
// negative index values. Index 0 refers to the first element, and
// index -1 refers to the last.
func (q *Queue) Get(i int) interface{} {
	// If indexing backwards, convert to positive index.
	if i < 0 {
		i += q.count  // 支持负数，从队列尾部开始
	}
	if i < 0 || i >= q.count {
		panic("queue: Get() called with index out of range")
	}
	// bitwise modulus
	return q.buf[(q.head+i)&(len(q.buf)-1)] // 返回队列的第i个元素
	// 因为数环形缓存区，队尾可能在缓存去前半部分，所以需要与一下队列最大下标
}

// Remove removes and returns the element from the front of the queue. If the
// queue is empty, the call will panic.
func (q *Queue) Remove() interface{} {
	if q.count <= 0 {
		panic("queue: Remove() called on empty queue")
	}
	ret := q.buf[q.head] // 出队
	q.buf[q.head] = nil
	// bitwise modulus
	q.head = (q.head + 1) & (len(q.buf) - 1) // 队头下标在环形缓存区后移一位
	q.count--   // 队列长度减一
	// Resize down if buffer 1/4 full.
	if len(q.buf) > minQueueLen && (q.count<<2) == len(q.buf) {
		q.resize()  // 重置缓存区大小，当队列长度减小到缓存区长度1/4的时候，但不可小于最小长度16
	}
	return ret
}
```

## 测试
> 打印缓存区数据，查看缓存区大小变化：<br>
先在queue.go中添加打印函数
>```
>func (q *Queue) LookData() {
>	fmt.Println(q.buf)
>}
>```

测试代码``main.go``
```go
package main

import (
	"test_code/queue"
)

func main() {
	que := queue.New()
	for i := 0; i < 16; i++ {
		que.Add(i)
		que.LookData()
	}

	for i := 0; i < 5; i++ {
		que.Remove()
		que.LookData()
	}

	for i := 0; i < 5; i++ {
		que.Add(i)
		que.LookData()
	}

	que.Add(100)
	que.LookData()

	for i := 0; i < 16; i++ {
		que.Remove()
		que.LookData()
	}
}
```
输出：
```
[0 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[0 1 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[0 1 2 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[0 1 2 3 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[0 1 2 3 4 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[0 1 2 3 4 5 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[0 1 2 3 4 5 6 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[0 1 2 3 4 5 6 7 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[0 1 2 3 4 5 6 7 8 <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[0 1 2 3 4 5 6 7 8 9 <nil> <nil> <nil> <nil> <nil> <nil>]
[0 1 2 3 4 5 6 7 8 9 10 <nil> <nil> <nil> <nil> <nil>]
[0 1 2 3 4 5 6 7 8 9 10 11 <nil> <nil> <nil> <nil>]
[0 1 2 3 4 5 6 7 8 9 10 11 12 <nil> <nil> <nil>]
[0 1 2 3 4 5 6 7 8 9 10 11 12 13 <nil> <nil>]
[0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 <nil>]
[0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15]
[<nil> 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15]
[<nil> <nil> 2 3 4 5 6 7 8 9 10 11 12 13 14 15]
[<nil> <nil> <nil> 3 4 5 6 7 8 9 10 11 12 13 14 15]
[<nil> <nil> <nil> <nil> 4 5 6 7 8 9 10 11 12 13 14 15]
[<nil> <nil> <nil> <nil> <nil> 5 6 7 8 9 10 11 12 13 14 15]
[0 <nil> <nil> <nil> <nil> 5 6 7 8 9 10 11 12 13 14 15]
[0 1 <nil> <nil> <nil> 5 6 7 8 9 10 11 12 13 14 15]
[0 1 2 <nil> <nil> 5 6 7 8 9 10 11 12 13 14 15]
[0 1 2 3 <nil> 5 6 7 8 9 10 11 12 13 14 15]
[0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15]
[5 6 7 8 9 10 11 12 13 14 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> 6 7 8 9 10 11 12 13 14 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> 7 8 9 10 11 12 13 14 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> 8 9 10 11 12 13 14 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> <nil> 9 10 11 12 13 14 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> <nil> <nil> 10 11 12 13 14 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> <nil> <nil> <nil> 11 12 13 14 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> <nil> <nil> <nil> <nil> 12 13 14 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> 13 14 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[14 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> 15 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> 0 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> 1 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> <nil> 2 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> <nil> <nil> 3 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> <nil> <nil> <nil> 4 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
[<nil> <nil> <nil> <nil> <nil> <nil> <nil> 100 <nil> <nil> <nil> <nil> <nil> <nil> <nil> <nil>]
```
