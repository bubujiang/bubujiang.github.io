---
title: golang异常处理
date: 2020-06-11 20:00:43
categories:
- golang
tags: 
- golang
- panic
notoc: true
---
* 可以在不希望程序继续运行的前提下使用异常
* 异常通过panic关键字或者意料之外的空指针等错误触发
* 程序一旦出现异常,会沿着调用栈向上逐个执行defer函数,直到最顶层.如果异常被捕获,那么会继续执行后面的语句.
```
func funcA() error {
	defer func() {
		fmt.Println("ffffff\n")//执行
	}()
	return funcB()
}
func funcB() error {
	panic("foo")
	return errors.New("success")//这句不会执行
}
func test() {
	defer func() {
		if p := recover(); p != nil {
			//此处捕获了异常
			fmt.Printf("panic recover! p: %v\n", p)
		}
	}()
	err := funcA()
	//下面的语句不会执行
	fmt.Printf("err is %v\n", err)
}
func main() {
	test()
	fmt.Printf("ddddd\n")//异常被捕获,这句就会执行.如果没有捕获就不会执行.
}
```