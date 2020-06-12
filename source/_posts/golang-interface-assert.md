---
title: golang接口类型断言
date: 2020-06-12 00:46:07
categories:
- golang
tags: 
- golang
- interface
---
# 前提
要弄清接口断言,必须先了解接口的静态类型和动态类型.
* 静态类型:是变量声明的时候的声明类型.
* 动态类型(接口特有):运行时赋值给这个变量的具体的值的类型,值为nil的时候没有动态类型.
```
type T string
var x interface{} // x为零值nil,静态类型为interface{}
var v *T // v为零值nil,静态类型为*T
x = 42 // x的值为42,动态类型为int,静态类型为interface{}
x = v // x的值为nil,静态类型为interface{},没有动态类型,因为指为nil
t := T("abc")
x = &t// x的值为abc,静态类型为interface{},动态类型为*T
```
# 断言
## 签名
```
n,ok := x.(T)
```
## 用法
* x必须是接口类型,如果不是可以用interface{}(x)来强转
* 如果T是接口且x已经实现了这个接口,那么n就是x相等的静态类型为T,动态类型为x同类型的值
* 如果T是结构体类型,那么n就会是T类型的变量,不会继承x的任何方法和属性
* 在需要调用动态类型的方法时,需要显示的做类型断言,不然编译器不知道运行时的类型,会报错(比如形参是空接口,实参是其它类型的时候)
* 当形参是空接口,实参是函数名的时候,是无法直接调用这个实参函数的,需要做如下调用
```
type FuncType func(s string) int
func PrintFunc(s string) int{
    fmt.Println(s)
    return 1
}
func JudgeFuncType(v interface{}){
    if _func, ok := v.(FuncType); ok{
        _func("hello")
    }
}
func main() {
    JudgeFuncType(FuncType(PrintFunc))
}
```
# 类型判断
## 签名
```
x.(type)
```
## 用法
* x必须是接口类型,如果不是可以用interface{}(x)来强转
* 只能和switch语句结合使用
* 返回实现的接口类型
```
type I1 interface {
    test()
}
type I2 interface {
	test()
}
type T struct {
	a int
}
func (i *T) test() {}
func main() {
	var i *T
	i = &T{a:1}
	switch interface{}(i).(type) {
	case I1:
		fmt.Println("a")
		break
	case I2:
		fmt.Println("b")
		break
	}
}
```
