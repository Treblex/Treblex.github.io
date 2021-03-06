---
title: 常用javascript方法整理之数字千分位格式化
date: 2019-08-20 16:21:57
---

## 场景

```
> 使用千分位格式化金额，更正式的显示
```

## 效果

![](https://i.loli.net/2019/08/21/z9wpms86QhHSbFr.png)

## 思路

```
 1.分割整数位和小数位，处理小数位补零，或者四舍五入
 2.获取整数位的长度 按3分割 插入 , 号
 3.拼接字符串 组合返回值
```

## 实现

```javascript
/**
 * 金额类数字格式化 千分位 小数点
 * @param {*} num   传入的数字
 * @param {*} f 	保留的小数位数
 */
const moneyFormat = (num, f) => {
  num = String(num); //转化为字符串

  let Old_f = null;
  if (num.search(/\./) > -1) {
    Old_f = num.toString().split(".")[1].length; //原始小数的位数
  }
  if (Old_f && Old_f !== "null" && Old_f < f) {
    //如果比目标位数小则补 “0”
    for (let i = 0; i < f - Old_f; i++) {
      num += "0";
    }
  }
  num = Math.floor(Number(num).toFixed(f) * Math.pow(10, f)) + ""; //tofixxed到值定位数后 转换成整数

  let integer = num.slice(0, num.length - f) + ""; //取转换后到字符串整数部分
  let decimal = num.slice(num.length - f, num.length) + ""; //小数部分

  let thou = Math.floor(integer.length / 3); //thousandCentimeter 取千分位到个数
  if (integer.length % 3 == 0) {
    thou = thou - 1; //如果完美整除则少一次循环
  }
  let new_integer = ""; //新的整数位
  let temp = 0; //临时坐标
  for (let i = 0; i < thou + 1; i++) {
    let target = integer.length - (thou - i) * 3; //thou-1 按照 54321 这样做循环 获取分号的坐标
    new_integer += integer.slice(temp, target); //拼接字符串
    if (i !== thou) {
      new_integer += ","; //过滤最后一个
    }
    temp = target;
  }
  integer = new_integer;

  return {
    full: integer + "." + decimal, //自动拼接字符串
    arr: [integer, decimal], //分别返回 整数部分 和 小数部分 用于自定义样式
  };
};
```
