---
title: "Goroutine 切片参数问题"
date: 2022-07-26T16:44:04+08:00
tags: ["go","待解决"]
---


``` go
package main

import (
	"fmt"
	"runtime"
	"sync"

	"github.com/panjf2000/ants/v2"
)

type TargetPeople struct {
	ID int64
}

type WorkDataItem struct {
	Name int64
}

type Entity struct {
	PID      string
	TName    string
	WorkData []WorkDataItem
}

func main() {

	var wg sync.WaitGroup
	p, _ := ants.NewPool(runtime.NumCPU())
	defer p.Release()
	var params Entity
	params.PID = "pid"
	params.TName = "tname"

	var ids []int64

	for _, vP := range preparePeopleList() {
		params.WorkData = append(params.WorkData, WorkDataItem{
			Name: int64(vP.ID),
		})
		// wd = append(wd, WorkDataItem{
		// 	Name: int64(vP.ID),
		// })
		ids = append(ids, vP.ID)
		if len(params.WorkData) == 3 {

			wd := make([]WorkDataItem, len(params.WorkData))
			copy(wd, params.WorkData)

			wg.Add(1)
			p.Submit(wrap(&wg, ids, params, wd))

			params.WorkData = params.WorkData[0:0]
			ids = ids[0:0]
		}

	}
	wg.Wait()
}

func wrap(wg *sync.WaitGroup, ids []int64, vp Entity, wd []WorkDataItem) func() {
	return func() {
		defer wg.Done()
		// time.Sleep(3 * time.Second)
		fmt.Println(ids, vp.WorkData, wd)
	}
}

func preparePeopleList() []*TargetPeople {
	var list []*TargetPeople
	for i := 1; i < 20000; i++ {
		list = append(list, &TargetPeople{ID: int64(i)})
	}
	return list
}
```

假设people这个切片可以更大，对于这种分组后传递给协程，在保证内存大小可以预估且不乱的情况下有哪些方案？

目前我只能有三种想法：

- copy 深拷贝
- channel 每一个协程一个channel
- sync.Pool 这个有没有可能
