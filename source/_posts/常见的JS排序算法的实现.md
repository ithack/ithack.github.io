---
title: 常见的JS排序算法的实现
date: 2016-08-16 15:29:10
tags: [Javascript,排序]
---
今天就来说一说几种我们平时面试和实际应用中的JS排序的实现方法！下面的方法用了ES6中的一些新属性，但整体思路是不变的！

原文：[Sorting Algorithms in Javascript](http://h3manth.com/javascript-sorting/)

译文：排序算法的JavaScript实现

译者：dwqs

# 快速排序 #

>let quicksort = function(arr) {
>　if(arr.length <= 1)return arr;
>　let pivot = Math.floor((arr.length -1)/2);
>　let val = arr[pivot], less = [], more = [];
>　arr.splice(pivot, 1);
>　arr.forEach(function(e,i,a){
>　　e < val ? less.push(e) : more.push(e);
>　});
>　return (quicksort(less)).concat([val],quicksort(more));
>}


# 冒泡排序 #
<pre>
let compare = (n1, n2) => n1 - n2;

let bubbleSort = (arr, cmp = compare) => {
  for (let i = 0; i < arr.length; i++) {
    for (let j = i; j > 0; j--) {
      if (cmp(arr[j], arr[j - 1]) < 0) {
        [arr[j], arr[j - 1]] = [arr[j - 1], arr[j]];
      }
    }
  }
  return arr;
};
</pre>
<!--more-->
# 插入排序 #

<pre>
let insertionSort = (arr) => {
    for (let i = 0; i < a.length; i++) {
        let toCmp = arr[i];
        for (let j = i; j > 0 && toCmp < a[j - 1]; j--)
            arr[j] = a[j - 1];
        arr[j] = toCmp;
    }
    return arr;
}
</pre>

# 选择排序 #

<pre>
var selectionSort = function (arr) {
  let i,m,j;
  for (i = -1; ++i < a.length;) {
    for (m = j = i; ++j < a.length;) {
      if (arr[m] > arr[j]) m = j;
    }
    [arr[m], arr[i]] = [arr[i], arr[m]];
 }
 return arr;
}
</pre>

# 归并排序 #


>let mergeSort = (arr) => {
>　if (arr.length < 2) return arr;
>　let middle = parseInt(arr.length / 2),
>　left = arr.slice(0, middle),
>　right = arr.slice(middle);
>　return merge(mergeSort(left), mergeSort(right));
>}
>
>let merge = (left, right) => {
>　let result = [];
>　while (left.length && right.length) {
>　　left[0] <= right[0] ?result.push(left.shift()):result.push(right.shift());
>　}
>　while (left.length) result.push(left.shift());
>　while (right.length) result.push(right.shift());
>　return result;
>}

</pre>









