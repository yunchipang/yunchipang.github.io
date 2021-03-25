---
layout: post
title: 【CS50】week3 - algorithms | plurality, runoff/tideman 
permalink: /cs50-week3-algorithms.html
date: 2021-02-16
author: yunchipang
tags: [CS50, C]
---

# **basics**
如同標題所寫，這堂課David教的是最基本的演算法，和Brian合作用淺顯易懂的圖像理解法，利用道具簡單介紹`linear search`, `binary search`, `selection sort`, `bubble sort`以及`merge sort`這幾項入門的演算法基礎，也解釋了各個演算法時間複雜度的upper bound及lower bound。以下是David課堂中提到的各個演算法的pseudo code筆記。

### **selection sort**
```
for i from 0 to n-1
	find smallest item between i'th item and last item
	save smallest item with i'th item
```

### **bubble sort**
```
repeat until sorted
	for i from 0 to n-2
		if i'th and i+1'th elements out of order
			swap them
	if no swaps
		quit
```

### **merge sort**
```
if only one number
	quit
else
	sort the left half of numbers
	sort the right half of numbers
	merge sorted halves
```
透過介紹`merge sort`，David也簡單帶出了「遞迴」(recursion)這個在函式中呼叫自身的概念。他在`merge sort`裡面發揮的功能就是將所有資料切成一半、再各自一半、再各自一半，如此下去最小的組合即是兩個數字互相比較，並將兩數按照順序排列，如此執行函式直到全部數字被成功排列。如果看文字難懂的話，去看Brian的手動排列示範影片，保證立刻清楚到不行！

<br>

# **pset3/plurality**

# **pset3/runoff**

# **pset3/tideman**