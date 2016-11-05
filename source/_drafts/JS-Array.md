title: JS-Array
date: 2016-08-07 17:00:52
tags:
categories:
---



> Learning JavaScript **Array** object

1、遍历数组

```javascript
var array = ['hello', 'world'];
array.forEach((item, index, array) => {
    //...
})
```

2、增删元素:

- 数组尾部操作：`push` `pop`
- 数组头部操作：`unshift` `shift`

3、移除指定位置的元素，然后添加新元素

 `array.splice(start, deleteCount[, item1[, item2[, ...]]])` The splice() method changes the content of an array by removing existing elements and/or adding new elements.

```javascript
var array = ['hello', 'world']
array.splice(1, 1) // array: ["world"]
array.splice(1, 1, 'deng') // array: ["hello", "deng"]
```

4、数组拷贝

`arr.slice([begin[, end]])` The slice() method returns a shallow copy of a portion of an array into a new array object.

```	javascript
var array = ['hello', 'world']
var newArray = array.slice() // newArray: ["hello", "world"]
```

5、判断是否是数组 `Array.isArray()`

6、The ` fill() ` method fills all the elements of an array from a start index to an end index with a static value.



// TODO: http://web.jobbole.com/87355/



