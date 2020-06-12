---
title: golang slice注意事项
date: 2020-06-12 17:40:55
tags:
- golang
- slice
categories:
- golang
---
# 声明
```
//newSlice和oldSlice指向同一个底层数组
newSlice := oldSlice[0:cap(oldSlice)]
if fmt.Sprintf("%p",&newSlice[0]) == fmt.Sprintf("%p",&oldSlice[0]){
    fmt.Printf("这一句必然会执行")
}
```
# 扩容
## 方法1
> 采用append实现,当容量不够时会自动扩容,自动扩容的尺寸为append之前容量的2倍,不推荐此方法.
```
oldSlice := []int{0,1,2}
fmt.Print(cap(oldSlice))//3
newSlice := append(oldSlice,3)
fmt.Print(cap(newSlice))//6
if fmt.Sprintf("%p",&newSlice[0]) == fmt.Sprintf("%p",&oldSlice[0]){
    fmt.Printf("这一句不会执行")
}
```
## 方法2
> 采用copy实现,内存占用少,推荐此方法.
```
oldSlice := []int{0,1,2}
newSlice := make([]int,3,4)
copy(newSlice,oldSlice)
newSlice[3] = 3// or newSlice = append(newSlice,3)
fmt.Print(cap(newSlice))//4
if fmt.Sprintf("%p",&newSlice[0]) == fmt.Sprintf("%p",&oldSlice[0]){
    fmt.Printf("这一句不会执行")
}
```
# 注意事项
- 因为重新切分的切片和原切片引用同一个数组,所以当指需要原切片一小部分数据时,最好复制这一小部分数据,否则内存中将保存大量的数据.
```
var digitRegexp = regexp.MustCompile("[0-9]+")
func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    //只需要一小部分数据,但是却保存了一个文件的切片,
    //因为需要的数据是重新切分的切片,和原数据应用的相同的底层数组.
    return digitRegexp.Find(b)
}
func CopyDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    c := make([]byte, len(b))
    //用新数据代替了原数据,函数返回后原数据就会被销毁,不占用内存.
    copy(c, b)
    return c
}
```