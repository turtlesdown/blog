---
layout: post
title: Interesting Behavior in Go
author: Gunnar Aasen
date: January 28, 2014
categories: programming
tags:
  - golang
  - errors
  - quirks
---
_Overall, I've been happy with using [Go](http://golang.org/) as my primary language over the last year. It's well designed, easy to read, and has a great community. However, every once in awhile I run into some weird rough edges where the relative youthfulness of the language becomes evident. This blog post will be the first in a series that explore these quirks in golang._

### An odd error
A couple weeks ago, I set out to make a quick update to my simple HTTP server ([shs](https://github.com/gunnaraasen/shs)). I wanted shs to spit out a simple log of the directory it was serving on start-up. To do this, I added the `path/filepath` package to the imports and one other line:

```prettyprint
fmt.Println("Serving files from" + filepath.Abs(string(directory)))
```
where `directory` was type `http.Dir`. Simple enough, except the compiler spat out an error.

```prettyprint
./shs.go:25: multiple-value filepath.Abs() in single-value context
```
That's a weird error. I was pretty sure `fmt.Println()` was able to accept all values through the `...interface{}` receiver. Looking up [fmt.Println()](http://golang.org/pkg/fmt/#Println) and [filepath.Abs()](http://golang.org/pkg/path/filepath/#Abs) in the docs quickly confirmed their function signatures.

```prettyprint
func Println(a ...interface{}) (n int, err error)
func Abs(path string) (string, error)
```
I also took a look at the [spec](http://golang.org/ref/spec). Which seemed to confirm what I thought, that the return values of `filepath.Abs()` could be [passed](http://golang.org/ref/spec#Passing_arguments_to_..._parameters) to the empty interfaces `fmt.Println()` and [assigned](http://golang.org/ref/spec#Assignability). Since every type implements an empty interface in Go and `...interface{}` can take any number of return values, I was pretty confinced the compiler shouldn't have been throwing an error. Did a well-used function like `fmt.Println` really have a bug? I had to be missing something. 

### Digging deeper
Puzzled and still failing to grasp the meaning of the compiler error for what seemed to be valid code, I did a quick search on Google and immediately came up with this [bug report](https://code.google.com/p/go/issues/detail?id=973) and thread on [golang-nuts](https://groups.google.com/forum/#!topic/golang-nuts/rXmSkyINIzs).

Reading through those two threads, I realized that I had glossed over some important semantics. The bug report explains that the following lines in the [spec](http://golang.org/ref/spec#Calls) are particularly important.

> Given an expression `f` of function type `F`, `f(a1, a2, … an)` calls `f` with arguments `a1, a2, … an`. Except for one special case, arguments must be single-valued expressions assignable to the parameter types of `F` and are evaluated before the function is called. The type of the expression is the result type of `F`. A method invocation is similar but the method itself is specified as a selector upon a value of the receiver type for the method."

And here are the key lines...

> As a special case, if the return values of a function or method `g` are __equal in number and individually assignable__ to the parameters of another function or method `f`, then the call `f(g(parameters_of_g))` will invoke f after binding the return values of g to the parameters of `f` in order. The call of `f` __must contain no parameters other than the call of `g`__, and `g` must have at least one return value. __If `f` has a final `...` parameter, it is assigned the return values of `g` that remain after assignment of regular parameters__.

Strictly speaking, this means `fmt.Println()` works without the string at the beginning, but fails if there are any other arguements.

```prettyprint
Works -> fmt.Println(filepath.Abs(string(directory)))
Fails -> fmt.Println(any_arguement, filepath.Abs(string(directory)))
```
### An unsatisfying explanation
As Rob Pike states in the [bug report](https://code.google.com/p/go/issues/detail?id=973) mentioned earlier,

> It's debatable whether the spec admits `Printf(multipleReturnValues())` or `Println(multipleReturnValues)` but I think the answer is still no... In short, addressing this requires a change to the spec.  It should be clarified... the narrower case is explicitly permitted, but the general case is still not.

In short, the __only__ case where a `...` receiver will accept multiple return values is when it's the only arguement.

Why not allow `...` receivers to receive whatever they can accept? Well, as mentioned in the [golang-nuts thread](https://groups.google.com/forum/#!topic/golang-nuts/rXmSkyINIzs), it prevents people from crafting expressions which take multiple functions, like `f(g(x), h(x))`. Those kinds of expressions would be prone to silently breaking when arguements or return values changed. 

Why not ban all `...` receivers from accepting non-individually assignable multiple return values altogether? 

Why is that one case explicitly allowed? Probably because golang would be incredibly inconvenient for programmers to experiment on the fly without it. Adding friction to functions like `fmt.Println()` is a great way to kill the "almost dynamic" feeling of Go and put off the C++ programmers it's trying to attract.