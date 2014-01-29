---
layout: default
title: An Obscure Error in Go
author: Gunnar Aasen
date: January 28, 2014
categories: programming, golang
---
## The Error
I was making a quick update to my simple HTTP server ([shs](https://github.com/gunnaraasen/shs)) to display the directory being served to stdout. Other than importing the `path/filepath` package, the only line added was

<?prettify lang=go?>

```
	fmt.Println("Serving files from" + filepath.Abs(string(directory)))
```
where `directory` is of type `http.Dir`. Simple enough, except the compiler gave this error.

<?prettify lang=go?>

```
	./shs.go:25: multiple-value filepath.Abs() in single-value context
```
## Huh...
I quickly pull up the golang docs. The function signatures for [fmt.Println()](http://golang.org/pkg/fmt/#Println) and [filepath.Abs()](http://golang.org/pkg/path/filepath/#Abs) are:

<?prettify lang=go?>

```
	func Println(a ...interface{}) (n int, err error)
```
and

<?prettify lang=go?>

```
	func Abs(path string) (string, error)
```
According to the [spec](http://golang.org/ref/spec), the return values of `filepath.Abs()` should be [passed](http://golang.org/ref/spec#Passing_arguments_to_..._parameters) to `fmt.Println()` and [assigned](http://golang.org/ref/spec#Assignability) to empty interfaces. Therefore, `filepath.Abs()` should be executing and, along with the first string, pass `string, string, err` as arguements to be assigned to a slice of empty interfaces `[]interface{}` for `fmt.Println()` to process. This should work since every type implements an empty interface in Go.

Puzzled and still failing to grasp the meaning of the compiler error for what seems to be valid code, I did a quick search on the internet and imediately came up with this [bug report](https://code.google.com/p/go/issues/detail?id=973) and thread on [golang-nuts](https://groups.google.com/forum/#!topic/golang-nuts/rXmSkyINIzs). The bug report explains that, according to the [spec](http://golang.org/ref/spec#Calls):
> Given an expression f of function type F, '''f(a1, a2, … an)'''calls f with arguments a1, a2, … an. Except for one special case, arguments must be single-valued expressions assignable to the parameter types of F and are evaluated before the function is called. The type of the expression is the result type of F. A method invocation is similar but the method itself is specified as a selector upon a value of the receiver type for the method."

And here are the key lines...
> As a special case, if the return values of a function or method g are __equal in number and individually assignable__ to the parameters of another function or method f, then the call f(g(parameters_of_g)) will invoke f after binding the return values of g to the parameters of f in order. The call of f __must contain no parameters other than the call of g__, and g must have at least one return value. __If f has a final ... parameter, it is assigned the return values of g that remain after assignment of regular parameters__.

Strictly speaking, this means that 'fmt.Println()' works without the string at the beginning and fails with any other arguements.

<?prettify lang=go?>

```
Works -> fmt.Println(filepath.Abs(string(directory)))
Fails -> fmt.Println(any_arguement, filepath.Abs(string(directory)))
```

As Rob Pike states in the [bug report](https://code.google.com/p/go/issues/detail?id=973), 

1. As mentioned in the [golang-nuts thread](https://groups.google.com/forum/#!topic/golang-nuts/rXmSkyINIzs), it's a deliberate design decision to prevent people from crafting expressions like f(g(x), h(x)), which would be prone to breaking when arguements change. 
2. Alternatively, using accepting multiple return values from 
