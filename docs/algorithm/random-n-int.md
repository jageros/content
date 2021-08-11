---
type: "posts"
author: "jager"
title: "生成x个随机数"
date: "2021-08-03"
tags: [
"随机数",
"算法",
"go"
]
---

> 需求：生成x个随机数，要求这个x个随机数的和为y， 且随机数的最大值小于平均数的3倍，最小值大于0，例如：5个和为10的随机数避免出现6，1，1，1，1的情况。
> 【使用场景：游戏中卡牌包开包时随机出现卡牌质量的分布】<br>
> 本文展示go语言对该算法的实现 

<!--more-->

## 代码实现

```go
package main

import (
	"fmt"
	"math"
	"math/rand"
	"sort"
	"time"
)

func main() {
	rand.Seed(time.Now().Unix()) // 用时间初始化种子
	for i := 0; i < 10; i++ { // 求10次结果
		fmt.Println(randFewInt(10, 5, 6))
	}
}

/* 实现函数
 * f(sum, n) return []int
 * sum 表示总和，n表示个数， max表示生成的随机数小于max
 * []int 表示n个随机数
 */
func randFewInt(sum, n, max int) []int {
	var vals, numList []int
	valSet := map[int]struct{}{}

	// 最大值不能大于总和
	min := float64(sum) / float64(n)
	if float64(max) <= min {
		max = int(math.Floor(min)) + 1
	}

	if max < sum {
		max2 := max
		// 将0-sum分割成长度小于max的x段
		for i := 0; i < n-1; i++ {
			val := max2 - 1
			valSet[val] = struct{}{}
			max2 = val + max
			if max2 > sum {
				break
			}
		}
	}

	// 把筛选出还没被选中的数
	var ls []int
	for i := 1; i < sum; i++ {
		if _, ok := valSet[i]; !ok {
			ls = append(ls, i)
		} else {
			vals = append(vals, i)
		}
	}

	// 分割成n段需要n-1个随机数，从筛选出的数中随机出剩余个数的数
	//log.Printf(" ====== ls=%v  vals=%v", ls, vals)
	remainCnt := n - 1 - len(vals)
	vals = append(vals, RandIntSample(ls, remainCnt, false)...)

	// 在n-1个数中加上首端和末端， 即0和sum
	vals = append(vals, 0, sum)

	// 对n+1个数进行排序
	sort.Ints(vals)
	//log.Printf(" ====== raminCnt=%d   vals=%v", remainCnt, vals)

	// 求出每一段的长度， 即n个随机数
	for i := 1; i < len(vals); i++ {
		a := vals[i] - vals[i-1]
		numList = append(numList, a)
	}
	return numList
}

/* 从数组list中随机取n个数
 * canRepeat 表示是否可重复
 * 取出的数如果不可重复就不会出现0的随机数
 */
func RandIntSample(list []int, n int, canRepeat bool) []int {
	var args []interface{}
	for _, e := range list {
		args = append(args, e)
	}

	sampleList := RandSample(args, n, canRepeat)
	var ret []int
	for _, e := range sampleList {
		ret = append(ret, e.(int))
	}
	return ret
}

// 接口化实现， 即可用于任何类型的数组，不只是int型
/* 从数组list中随机取n个数
 * canRepeat 表示是否可重复
 * 取出的数如果不可重复就不会出现0的随机数
 */
func RandSample(list []interface{}, n int, canRepeat bool) []interface{} {
	if n <= 0 {
		return []interface{}{}
	}
	if !canRepeat && len(list) <= n {
		return list
	}

	var ret []interface{}
	var set map[int]struct{}
	if !canRepeat {
		set = make(map[int]struct{})
	}
	for len(ret) < n {
		i := rand.Intn(len(list))
		if !canRepeat {
			if _, ok := set[i]; ok {
				continue
			}
			set[i] = struct{}{}
		}
		ret = append(ret, list[i])
	}
	return ret
}
```