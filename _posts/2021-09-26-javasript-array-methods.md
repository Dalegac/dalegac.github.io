---
title: JavaScript数组方法
author: Dale
date: 2021-09-26 11:16:00 +0800
categories: [Blogging, JavaScript]
tags: [JavaScript, Array]
math: true
mermaid: true
---

## 1. 数组的声明

### 1.1 数组文本

使用数组文本是创建 JavaScript 数组最简单的方法。

```
var array-name = [item1, item2, ...];
```

空格和折行并不重要，声明可横跨多行。
请不要最后一个元素之后写逗号，比如：

```
var guns = ["AWM","98Kar","Uzi",]
```

可能存在跨浏览器兼容性问题。

### 1.2 使用 JavaScript 关键词 new

使用 new Array()创建数组

```
//1
var cars = new Array("Saab", "Volvo", "BMW");
//2
var arr = new Array(3)
arr[0] = "George"
arr[1] = "John"
arr[2] = "Thomas"
```

**出于简洁、可读性和执行速度的考虑，请使用第一种方法（数组文本方法）。**

## 2. 静态方法

### 2.1 Array.of()

Array.of()方法创建一个具有可变数量参数的新数组实例，而不考虑参数的数量或类型。

```
//创建一个长度为50的空数组，元素undefined
let arr = new Array(50);
//创建一个数组：[50]
let arr = Array.of(50)
```

```
let arr = Array.of(1,2,3)
console.log(arr)    // [1,2,3]
```

一般用于将一组值，转换为数组。

### 2.2 Array.from()

Array.from() 方法从一个类似数组或可迭代对象中创建一个新的，浅拷贝的数组实例。

Array.from() 可以把类数组对象转变成数组。

- 将类数组对象转换为数组

```
let object = {
    '0': '声明方法',
    '1': '静态方法',
    '2': '实力方法',
    length: 3
};
let arr = Array.from(object);  // ["声明方法", "静态方法", "实例方法"]
```

- arguments 是一个类数组对象

```
function makeArray() {
  return Array.from(arguments);
}
let arr = makeArray('a', 'b', 'c');
console.log(arr);    // ["a", "b", "c"]
```

```
function cube() {
  return Array.from(arguments, value => value ** 3);
}
let arr = cube(1, 3, 5);    //  [1, 27, 125]
```

- dom 节点是一个类数组对象

```
Array.from(document.querySelectorAll('div'))
```

### 2.3 Array.isArray

Array.isArray() 用于确定传递的值是否是一个 Array。

```
// es5
Object.prototype.toString.call(arg) === '[object Array]';

//es6
let isArr = Array.isArray([1, 2, 3]);
console.log(isArr)  // true
```

## 3. 实例方法

![js数组方法](https://github.com/Dalegac/static-pic/blob/master/images/js数组方法.jpg?raw=true)

### 3.1 添加、删除元素

改变数组的本身的长度及内容

#### 3.1.1 push()

push() 方法可把它的参数顺序添加到 arrayObject 的尾部。

```
arrayObject.push(newelement1,newelement2,....,newelementX)
```

它直接修改 arrayObject，而不是创建一个新的数组。push()方法和 pop()方法使用数组提供的先进后出栈的功能。

```
var cars = new Array("Saab", "Volvo", "BMW")
console.log(cars.push("Benz","Ford"))   // 5
console.log(cars)            // ["Saab", "Volvo", "BMW","Benz","Ford"]
```

#### 3.1.2 pop()

pop() 方法将删除 arrayObject 的最后一个元素，把数组长度减 1，并且返回它删除的元素的值。

```
arrayObject.pop()
```

如果数组已经为空，则 pop() 不改变数组，并返回 undefined 值。不接受传参，只删除一个。

```
var cars = new Array("Saab", "Volvo", "BMW");
var car = cars.pop()
console.log(car) //BMW
console.log(cars) //["Saab","Volvo"]
```

#### 3.1.3 unshift()

unshift() 方法可向数组的开头添加一个或更多元素，并返回新的长度。

```
arrayObject.unshift(newelement1,newelement2,....,newelementX)
```

unshift() 方法将把它的参数插入 arrayObject 的头部，并将已经存在的元素顺次地移到较高的下标处，以便留出空间。该方法的第一个参数将成为数组的新元素 0，如果还有第二个参数，它将成为新的元素 1，以此类推。

```
var cars = new Array("Saab", "Volvo", "BMW")
console.log(cars.unshift("Benz","Ford"))   // 5
console.log(cars) // ["Benz","Ford","Saab", "Volvo", "BMW"]
```

**unshift() 方法无法在 Internet Explorer 中正确地工作！**

#### 3.1.4 shift()

shift() 方法用于把数组的第一个元素从其中删除，并返回第一个元素的值。不接受传参，只删除一个。

```
arrayObject.shift()
```

如果数组是空的，那么 shift()方法将不进行任何操作，返回 undefined 值。请注意，该方法不创建新数组，而是直接修改原有的 arrayObject。

```
var cars = new Array("Saab", "Volvo", "BMW");
var car = cars.shift()
console.log(car) // Saab
console.log(cars) // ["Volvo", "BMW"]
```

### 3.2 截取元素+删除、插入、替换元素+覆盖

#### 3.2.1 slice()

slice() 方法可从已有的数组中返回选定的元素。

```
arrayObject.slice(start,end)
```

| 参数  | 描述                                                                                                                                                                                          |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| start | 必需。规定从何处开始选取。如果是负数，那么它规定从数组尾部开始算起的位置。也就是说，-1 指最后一个元素，-2 指倒数第二个元素，以此类推。                                                        |
| end   | 可选。规定从何处结束选取。该参数是数组片断结束处的数组下标。如果没有指定该参数，那么切分的数组包含从 start 到数组结束的所有元素。如果这个参数是负数，那么它规定的是从数组尾部开始算起的元素。 |

```
var arr=['a','b','c','d']

var newArr=arr.slice(0,3);          //不包含索引值为3对应的元素,同(0,-1)
console.log(newArr);                   //输出的是['a','b','c']

var newArr2=arr.slice(0);           //如果没有第二个参数，截取到的是最后一个元素
console.log(newArr2);                //输出的是['a','b','c','d']
```

该方法并不会修改数组，而是返回一个子数组。

#### 3.2.2 splice()

splice() 方法向/从数组中添加/删除项目，然后返回被删除的项目。该方法会改变原始数组。

```
arrayObject.splice(index,howmany,item1,.....,itemX)
```

| 参数              | 描述                                                                  |
| ----------------- | --------------------------------------------------------------------- |
| index             | 必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。 |
| howmany           | 必需。要删除的项目数量。如果设置为 0，则不会删除项目。                |
| item1,.....,itemX | 可选。向数组添加的新项目。                                            |

splice()方法可删除从 index 处开始的零个或多个元素，并且用参数列表中声明的一个或多个值来替换那些被删除的元素。如果从 arrayObject 中删除了元素，则返回的是含有被删除的元素的数组。

```
var arr = ['a','b','c','d','e']

//删除前三个元素
var arrDel = arr.splice(0,3)
console.log(arrDel) // ['a','b','c']
console.log(arr) // ['d','e']

//插入元素
var a = ['f','g','h']
//插入一个数组 a
arr.splice(1,0,a)
console.log(arr) // ['d', Array(3), 'e']
// 插入三个元素
arr.splice(1,0,'f','g','h')
console.log(arr) // ['d', 'f', 'g', 'h', Array(3), 'e']

//替换倒数第二个元素
var arrRep = arr.splice(-2,1,'hello')
console.log(arrRep) // Array(3)
console.log(arr) // ['d', 'f', 'g', 'h', 'e']

```

#### 3.2.3 copyWithin()

copyWithin() 方法用于从数组的指定位置拷贝元素到数组的另一个指定位置中。

```

array.copyWithin(target, start, end)

```

| 参数   | 描述                                                                   |
| ------ | ---------------------------------------------------------------------- |
| target | 必需。复制到指定目标索引位置。                                         |
| start  | 可选。元素复制的起始位置。                                             |
| end    | 可选。停止复制的索引位置 (默认为 array.length)。如果为负值，表示倒数。 |

```

//例子 1
const arr1 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
arr1.copyWithin(1, 3, 6)
console.log('%s', JSON.stringify(arr1))
// [1,4,5,6,5,6,7,8,9,10,11]

//例子 2
const arr2 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
arr2.copyWithin(3)
console.log('%s', JSON.stringify(arr2))
//[1,2,3,1,2,3,4,5,6,7,8]

//例子 3
const arr3 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
arr3.copyWithin(3, -3, -2)
console.log('%s', JSON.stringify(arr3))
// [1,2,3,9,5,6,7,8,9,10,11]

```

copyWithin()用于操作当前数组自身，用来把某个区间位置的元素复制并覆盖到其他位置上去。

### 3.3 数组转字符串

#### 3.3.1 join()

join() 方法用于把数组中的所有元素放入一个字符串。元素是通过指定的分隔符进行分隔的。

```

arrayObject.join(separator)

```

| 参数      | 描述                                                             |
| --------- | ---------------------------------------------------------------- |
| separator | 可选。指定要使用的分隔符。如果省略该参数，则使用逗号作为分隔符。 |

返回一个字符串。该字符串是通过把 arrayObject 的每个元素转换为字符串，然后把这些字符串连接起来，在两个元素之间插入 separator 字符串而生成的。

```

var arr = ['d', ['a','b','c'], 'e']
console.log(arr.join("?") // d?a,b,c?e
console.log(arr.join("") // da,b,ce
console.log(arr.join()) // d,a,b,c,e

```

#### 3.3.2 toString()

toString() 方法可把数组转换为字符串，并返回结果。

```

arrayObject.toString()

```

arrayObject 的字符串表示。返回值与没有参数的 join()方法返回的字符串相同。当数组用于字符串环境时，JavaScript 会调用这一方法将数组自动转换成字符串。但是在某些情况下，需要显式地调用该方法。

```

var arr = ['d', ['a','b','c'], 'e']
console.log(arr.toString()) // d,a,b,c,e

```

### 3.4 排序

#### 3.4.1 reverse()

reverse() 方法用于颠倒数组中元素的顺序。**数组在原数组上进行颠倒，不生成副本。**

```

arrayObject.reverse()

```

```

var arr = [1,2,3,4]
arr.reverse()
console.log(arr) // [4, 3, 2, 1]

```

#### 3.4.2 sort()

sort() 方法用于对数组的元素进行排序。**数组在原数组上进行排序，不生成副本。**

```

arrayObject.sort(sortby)

```

| 参数   |               描述               |
| :----- | :------------------------------: |
| sortby | 可选。规定排序顺序。必须是函数。 |

默认排序顺序为按字母升序。可以按照字母/数字进行排序

```

// 数字排序（数字和升序）
var points = [40,100,1,5,25,10];
points.sort((a,b) => a-b); //points.sort(function(a,b){return a-b});
console.log(points) // [1, 5, 10, 25, 40, 100]

// 数字排序（数字和降序）
var points = [40,100,1,5,25,10];
points.sort((a,b) => b-a); //points.sort(function(a,b){return b-a});
console.log(points) //[100, 40, 25, 10, 5, 1]

//字母排序（字母和降序）
var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.sort();
fruits.reverse();
console.log(fruits) // ["Orange", "Mango", "Banana", "Apple"]

```

### 3.5 遍历数组

```

var cars = [
{ id: '1', price: 40 },
{ id: '2', price: 110 },
{ id: '3', price: 60 },
{ id: '4', price: 80 },
{ id: '5', price: 50 },
{ id: '6', price: 130 }
]

```

#### 3.5.1 forEach()

forEach() 方法用于调用数组的每个元素，并将元素传递给回调函数。不会对空数组进行检测。

```

array.forEach(function(currentValue, index, arr), thisValue)

```

| 参数         | 描述                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| currentValue | 必须。当前元素的值                                                                                           |
| index        | 可选。当前元素的索引值                                                                                       |
| arr          | 可选。当前元素属于的数组对象                                                                                 |
| thisValue    | 可选。对象作为该执行回调时使用，传递给函数，用作"this"的值。如果省略了 thisValue ，"this" 的值为 "undefined" |

```

// forEach: 对数组每一个项进行操作，返回 undefined
cars.forEach(car => car.price += 10 ) //undefined
//[50, 120, 70, 90, 60, 140]
注意：forEach 不能 break 也不能 return！

```

#### 3.5.2 map()

map() 方法返回一个新数组，数组中的元素为原始数组元素调用函数处理后的值。map() 方法按照原始数组元素顺序依次处理元素。
**注意： map() 不会改变原始数组。**

```

array.map(function(currentValue, index, arr), thisValue)

```

| 参数         | 描述                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| currentValue | 必须。当前元素的值                                                                                           |
| index        | 可选。当前元素的索引值                                                                                       |
| arr          | 可选。当前元素属于的数组对象                                                                                 |
| thisValue    | 可选。对象作为该执行回调时使用，传递给函数，用作"this"的值。如果省略了 thisValue ，"this" 的值为 "undefined" |

```

// map: 对数组每一个项进行操作，并返回操作后的结果
var p = cars.map(car => { return car.price += 10 })
console.log(p) // [50, 120, 70, 90, 60, 140]

//从接口得到数据 res
let r = res.map(item => {
return {
title: item.name,
sex: item.sex === 1? '男':item.sex === 0?'女':'保密',
age: item.age,
avatar: item.img
}
})

```

#### 3.5.3 filter()

filter() 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素。如果没有符合条件的元素则返回空数组。filter() 不会对空数组进行检测。 filter() 不会改变原始数组。

```

array.filter(function(currentValue,index,arr), thisValue)

```

| 参数         | 描述                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| currentValue | 必须。当前元素的值                                                                                           |
| index        | 可选。当前元素的索引值                                                                                       |
| arr          | 可选。当前元素属于的数组对象                                                                                 |
| thisValue    | 可选。对象作为该执行回调时使用，传递给函数，用作"this"的值。如果省略了 thisValue ，"this" 的值为 "undefined" |

```

var bigOnes = cars.filter(car => { return car.price > 100 })
// [ { id: "2", price: 110 }, { id: "6", price: 130 } ]
// 返回满足条件的数组，并不改变原数组

```

#### 3.5.4 some()

some() 方法用于检测数组中的元素是否满足指定条件（函数提供）。
some() 方法会依次执行数组的每个元素：

- 如果有一个元素满足条件，则表达式返回 true , 剩余的元素不会再执行检测。
- 如果没有满足条件的元素，则返回 false。

注意： some() 不会对空数组进行检测。some() 不会改变原始数组。

```

arrayObject.some(function(currentValue,index,arr),thisValue)

```

| 参数         | 描述                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| currentValue | 必须。当前元素的值                                                                                           |
| index        | 可选。当前元素的索引值                                                                                       |
| arr          | 可选。当前元素属于的数组对象                                                                                 |
| thisValue    | 可选。对象作为该执行回调时使用，传递给函数，用作"this"的值。如果省略了 thisValue ，"this" 的值为 "undefined" |

```

var hasbig = cars.some(car => { return car.price > 100 }) // true

```

#### 3.5.5 every()

every() 方法用于检测数组所有元素是否都符合指定条件（通过函数提供）。

```

arrayObject.every (function (currentValue,index,arr), thisValue)

```

| 参数         | 描述                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| currentValue | 必须。当前元素的值                                                                                           |
| index        | 可选。当前元素的索引值                                                                                       |
| arr          | 可选。当前元素属于的数组对象                                                                                 |
| thisValue    | 可选。对象作为该执行回调时使用，传递给函数，用作"this"的值。如果省略了 thisValue ，"this" 的值为 "undefined" |

如果所有元素都满足条件，则返回 true。如果数组中检测到有一个元素不满足，则整个表达式返回 false ，且剩余的元素不会再进行检测。

```

cars.every( car => {return car.price<=130} //true
cars.every( car => {return car.price<=120} //false

```

#### 3.5.6 reduce()

reduce() 方法接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值。
reduce() 可以作为一个高阶函数，用于函数的 compose。
注意: reduce() 对于空数组是不会执行回调函数的。
reduceRight() 方法的功能和 reduce() 功能是一样的，不同的是 reduceRight() 从数组的末尾向前将数组中的数组项做累加。文档中就详细说明 reduce()，不对 reduceRight() 方法另外赘述。

```

arr.reduce(callback(accumulator, currentValue[,index[,array]])[, initialValue])

```

reduce() 方法有两个参数，第一个是回调函数，第二个是初始值。
参数 | 描述
---|---
accumulator | （累加器）——累加器累加回调函数的返回值。
currentValue |（当前值）——处理数组的当前元素。
currentIndex |（当前索引）——处理数组当前元素的索引。
array |（源数组）
initialValue | 初始值

```

// 初始化 accumulator = 0
const numbersArr = [67, 90, 100, 37, 60];

const total = numbersArr.reduce(function(accumulator, currentValue){
console.log("accumulator is " + accumulator + " current value is " + currentValue);
return accumulator + currentValue;
}, 0);

console.log("total : "+ total);

// 结果
accumulator is 0 current value is 67
accumulator is 67 current value is 90
accumulator is 157 current value is 100
accumulator is 257 current value is 37
accumulator is 294 current value is 60
total : 354

// JavaScript reduce 用例

// 1. 对数组的所有值求和
const studentResult = [67, 90, 100, 37, 60];
const total = studentResult.reduce((accumulator, currentValue) => accumulator +currentValue, 0);
console.log(total); // 354

// 2. 对象数组中的数值之和
const studentResult = [
{ subject: '数学', marks: 78 },
{ subject: '物理', marks: 80 },
{ subject: '化学', marks: 93 }
];

const total = studentResult.reduce((accumulator, currentValue) =>
accumulator + currentValue.marks, 0);

console.log(total); // 251

// 3. 展平数组
const twoDArr = [ [1,2], [3,4], [5,6], [7,8] , [9,10] ];

const oneDArr = twoDArr.reduce((accumulator, currentValue) => accumulator.concat(currentValue));

console.log(oneDArr);
// [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ]

// 4. 按属性分组对象
const result = [
{subject: '物理', marks: 41},
{subject: '化学', marks: 59},
{subject: '高等数学', marks: 36},
{subject: '应用数学', marks: 90},
{subject: '英语', marks: 64},
];

let initialValue = {
pass: [],
fail: []
}

const groupedResult = result.reduce((accumulator, current) => {
(current.marks >= 50) ? accumulator.pass.push(current) : accumulator.fail.push(current);
return accumulator;
}, initialValue);

console.log(groupedResult);
//结果
{
pass: [
{ subject: ‘化学’, marks: 59 },
{ subject: ‘应用数学’, marks: 90 },
{ subject: ‘英语’, marks: 64 }
],
fail: [
{ subject: ‘物理’, marks: 41 },
{ subject: ‘高等数学’, marks: 36 }
]
}

// 5. 删除数组中的重复项
const duplicatedsArr = [1, 5, 6, 5, 7, 1, 6, 8, 9, 7];

const removeDuplicatedArr = duplicatedsArr.reduce((accumulator, currentValue) =>
{
if(!accumulator.includes(currentValue)){
accumulator.push(currentValue);
}
return accumulator;
}, []);

console.log(removeDuplicatedArr);
// [ 1, 5, 6, 7, 8, 9 ]

```

### 3.6 查找

```

var cars = [
{ id: '1001', price: 40 },
{ id: '1002', price: 110 },
{ id: '1003', price: 60 },
{ id: '1004', price: 80 },
{ id: '1005', price: 50 },
{ id: '1006', price: 130 }
]

```

#### 3.6.1 find()

数组实例的 find 方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为 true 的成员，然后返回该成员。如果没有符合条件的成员，则返回 undefined。

```

arrayObject.find(callback(value,index,array))

```

| 参数  | 描述       |
| ----- | ---------- |
| value | 当前的值   |
| index | 当前的位置 |
| array | 原数组     |

```

[1, 4, -5, 10].find((n) => n < 0)
// -5

[1, 5, 10, 15].find(function(value, index, arr) {
return value > 9;
}) // 10

```

#### 3.6.2 indexOf()

搜索数组中的元素，并返回它所在的位置。

```

arrayObject.indexOf(item,start)

```

| 参数  | 描述                                                                                                                                  |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------- |
| item  | 必须。查找的元素。                                                                                                                    |
| start | 可选的整数参数。规定在数组中开始检索的位置。它的合法取值是 0 到 stringObject.length - 1。如省略该参数，则将从字符串的首字符开始检索。 |

```

var a = cars.indexOf({ id: '1004', price: 80 }); //a = -1(无法比较对象，地址不同)

var fruits=["Banana","Orange","Apple","Mango","Banana","Orange","Apple"];
var t = fruits.indexOf("Apple",4);// t = 6

```

#### 3.6.3 lastIndexOf()

lastIndexOf() 方法可返回一个指定的元素在数组中最后出现的位置，从数组的后面向前查找。
如果要检索的元素没有出现，则该方法返回 -1。

```

array.lastIndexOf(item,start)

```

| 参数  | 描述                                                                                                                                            |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| item  | 必须。查找的元素。                                                                                                                              |
| start | 可选的整数参数。规定在数组中开始检索的位置。它的合法取值是 0 到 stringObject.length - 1。如省略该参数，则将从字符串的最后一个字符处开始检索。。 |

```

var a = cars.lastIndexOf({ id: '1004', price: 80 });//a=-1(无法比较对象，地址不同)

var fruits=["Banana","Orange","Apple","Mango","Banana","Orange","Apple"];
var t = fruits.lastIndexOf("Apple");// t = 2

```

#### 3.6.4 findIndex()

findIndex() 方法返回传入一个测试条件（函数）符合条件的数组第一个元素位置。

findIndex() 方法为数组中的每个元素都调用一次函数执行：

- 当数组中的元素在测试条件时返回 true 时,findIndex()返回符合条件的元素的索引位置，之后的值不会再调用执行函数。
- 如果没有符合条件的元素返回 -1

注意: findIndex() 对于空数组，函数是不会执行的。findIndex()并没有改变数组的原始值。

```

arrayObject.findIndex(function(currentValue, index, arr), thisValue)

```

| 参数         | 描述                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------ |
| currentValue | 必须。当前元素的值                                                                                           |
| index        | 可选。当前元素的索引值                                                                                       |
| arr          | 可选。当前元素属于的数组对象                                                                                 |
| thisValue    | 可选。对象作为该执行回调时使用，传递给函数，用作"this"的值。如果省略了 thisValue ，"this" 的值为 "undefined" |

```

var i = cars.findIndex(car=>{ return car.price > 100 }) // i=1

```

#### 3.6.5 includes()

ncludes() 方法用来判断一个数组是否包含一个指定的值，如果是返回 true，否则 false。

```

arr.includes(searchElement, fromIndex)

```

| 参数          | 描述                                                                                                               |
| ------------- | ------------------------------------------------------------------------------------------------------------------ |
| searchElement | 必须。需要查找的元素值。                                                                                           |
| fromIndex     | 可选。从该索引处开始查找 searchElement。如果为负值，则按升序从 array.length + fromIndex 的索引开始搜索。默认为 0。 |

```

[1, 2, 3].includes(2); // true
[1, 2, 3].includes(4); // false
[1, 2, 3].includes(3, 3); // false
[1, 2, 3].includes(3, -1); // true
[1, 2, NaN].includes(NaN); // true

```

### 3.7 其他

#### 3.7.1 concat()

concat() 方法用于连接两个或多个数组。该方法不会改变现有的数组，而仅仅会返回被连接数组的一个副本。

```

arrayObject.concat(array1,array2,...,arrayX)

```

| 参数                     | 描述                                                           |
| ------------------------ | -------------------------------------------------------------- |
| array1,array2,...,arrayX | 必需。该参数可以是具体的值，也可以是数组对象。可以是任意多个。 |

```

var hege = ["Cecilie", "Lone"];
var stale = ["Emil", "Tobias", "Linus"];
var kai = ["Robin"];
var children = hege.concat(stale,kai,'Hello','World')
console.log(children)
// ["Cecilie", "Lone", "Emil", "Tobias", "Linus", "Robin", "hello", "world"]

```

如果要进行 concat() 操作的参数是数组，那么添加的是数组中的元素，而不是数组。**(只会解开第一层数组)**

#### 3.7.2 entries()

entries() 方法返回一个数组的迭代对象，该对象包含数组的键值对 (key/value)。

```

var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.entries();

[0, "Banana"]
[1, "Orange"]
[2, "Apple"]
[3, "Mango"]

```

迭代对象中数组的索引值作为 key， 数组元素作为 value。

#### 3.7.3 keys()

keys() 方法用于从数组创建一个包含数组键的可迭代对象。

如果对象是数组返回 true，否则返回 false。

```

var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.keys();

[0, "Banana"]
[1, "Orange"]
[2, "Apple"]
[3, "Mango"]

```

## 4. 数组的补充

### 4.1 ES6 中数组的扩展

#### 4.1.1 扩展运算符

扩展运算符（spread）是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。

### 4.2 数组技巧

#### 4.2.1 去重

使用 new Set()以及 Array.from()或者展开运算符(...)

```

var fruits = [“banana”, “apple”, “orange”, “watermelon”, “apple”, “orange”,“grape”, “apple”];

// 方法一
var uniqueFruits = Array.from(new Set(fruits));
console.log(uniqueFruits); // returns [“banana”, “apple”, “orange”, “watermelon”, “grape”]
// 方法二
var uniqueFruits2 = […new Set(fruits)];
console.log(uniqueFruits2); // returns [“banana”, “apple”, “orange”, “watermelon”, “grape”]

```

#### 4.2.2 清空数组

直接将之 length 设置成 0

```

var fruits = [“banana”, “apple”, “orange”, “watermelon”, “apple”, “orange”, “grape”, “apple”];

fruits.length = 0;
console.log(fruits); // returns []

```

#### 4.2.3 数组转换成对象

有时候需要将数组转换成对象的形式，使用.map()一类的迭代方法能达到目的，这里还有个更快的方法，前提是正好希望对象的 key 就是数组的索引

```

var fruits = [“banana”, “apple”, “orange”, “watermelon”];
var fruitsObj = { …fruits };
console.log(fruitsObj); // returns {0: “banana”, 1: “apple”, 2: “orange”, 3: “watermelon”, 4: “apple”, 5: “orange”, 6: “grape”, 7: “apple”}

```

#### 4.2.4 遍历数组

平时我们使用最多的就是数组的.map 方法，其实还有一个方法也能达到一样的目的，用法比较冷门，所以我们总是忽视，那就是 Array.from

```

var friends = [
{ name: ‘John’, age: 22 },
{ name: ‘Peter’, age: 23 },
{ name: ‘Mark’, age: 24 },
{ name: ‘Maria’, age: 22 },
{ name: ‘Monica’, age: 21 },
{ name: ‘Martha’, age: 19 },
]

var friendsNames = Array.from(friends, ({name}) => name);
console.log(friendsNames); // returns [“John”, “Peter”, “Mark”, “Maria”, “Monica”, “Martha”]

```

#### 4.2.5 填充数组

创建数组的时候，你有没有遇到过需要填充上默认值的场景，你肯定首先想到的就是循环这个数组。ES6 提供了更便捷的.fill 方法

```

var newArray = new Array(10).fill(“1”);
console.log(newArray); // returns [“1”, “1”, “1”, “1”, “1”, “1”, “1”, “1”, “1”, “1”, “1”]

```

#### 4.2.6 合并数组

浅拷贝

```

var fruits = [“apple”, “banana”, “orange”];
var meat = [“poultry”, “beef”, “fish”];
var vegetables = [“potato”, “tomato”, “cucumber”];
var food = […fruits, …meat, …vegetables];
console.log(food); // [“apple”, “banana”, “orange”, “poultry”, “beef”, “fish”, “potato”, “tomato”, “cucumber”]

```

#### 4.2.7 两个数组的交集

```

var numOne = [0, 2, 4, 6, 8, 8];
var numTwo = [1, 2, 3, 4, 5, 6];
var duplicatedValues = […new Set(numOne)].filter(item => numTwo.includes(item));
console.log(duplicatedValues); // returns [2, 4, 6]

```

#### 4.2.8 去除假值

首先，我们熟悉下假值(falsy values)是什么？在 JS 中，假值有：false、0、''、null、NaN、undefined。现在我们找到这些假值并将它们移除，这里使用的是.filter 方法

```

var mixedArr = [0, “blue”, “”, NaN, 9, true, undefined, “white”, false];
var trueArr = mixedArr.filter(Boolean);
console.log(trueArr); // returns [“blue”, 9, true, “white”]

```

#### 4.2.9 随机值

从数组中获取随机的一个值，的核心知识是随机生成一个值 x：x >= 0 并且 x < 数组的 length

```

var colors = [“blue”, “white”, “green”, “navy”, “pink”, “purple”, “orange”, “yellow”, “black”, “brown”];
var randomColor = colors[(Math.floor(Math.random() * (colors.length)))]

```

#### 4.2.10 倒序

```

var colors = [“blue”, “white”, “green”, “navy”, “pink”, “purple”, “orange”, “yellow”, “black”, “brown”];
var reversedColors = colors.reverse();
// 或者 colors.slice().reverse();
// 两者有啥区别？
console.log(reversedColors); // returns [“brown”, “black”, “yellow”, “orange”, “purple”, “pink”, “navy”, “green”, “white”, “blue”]

```

#### 4.2.11 求和

```

var nums = [1, 5, 2, 6];
var sum = nums.reduce((x, y) => x + y);
console.log(sum); // returns 14

```
