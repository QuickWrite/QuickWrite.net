---
title: "My biggest gripe with the Go programming language"
meta_title: "My biggest gripe with the Go programming language"
description: "The main problem that Go faces in my unprofessional opinion"
date: 2025-03-09T15:02:47+01:00
image: "/images/header-images/my-biggest-gripe-with-go.png"
categories: ["Go", "Programming Language", "Critique"]
author: "QuickWrite"
tags: ["Go", "Programming Language", "Critique", "Assignment Operator"]
draft: false
---

> There are only two kinds of languages: the ones people complain about and the ones nobody uses.
>
> Probably not [Bjarne Stroustrup](https://www.stroustrup.com/quotes.html)

The programming language Go is a language that shines in simplicity and ease of use. 
It manages to bridge the gap between low level code and high level concepts quite nicely. 
Things like the garbage collector allow the programmer to **not** think about memory management, but it also allows for "more complex" things like references to an object.
To me, Go feels more or less like a language that has the benefits of a low level compiled language, but also has ergonomics like a high level language without all the stuff you normally have to think about.

This also means that this language is not made for real low level stuff like Operating Systems, Drivers, Browsers, etc. But it is a language in which most projects can be made. And with in-built language features like Goroutines it also has some features that make it quite easy to write parallel code. But let's stop with the praises as this blog post isn't titled "Go is a perfect language". This article is a critque of the Go programming language.

**So: What don't I like?** <br>
The assignment operator.

## The main problem
Go has different ways of creating a variable. There are:
- The most verbose version:
  ```go
  var name string
  name = "John"
  ```
- which can be written as:
  ```go
  var name string = "John"
  ```
- which can be written as:
  ```go
  var name = "John"
  ```
- which can be written as:
  ```go
  name := "John"
  ```

This is quite a lot. And even though it clashes with the idea of having *simple* syntax, this isn't my main problem. My main problem lies in the last listed way of creating a variable as it is quite similar to the initialization syntax:
```go
name = "John"
```

### So, what's the problem?
The main problem lies in the fact that this can easily lead to completely different outcomes of the program, if you are not attentive enough. Here for example two programs that do something completely different whilst looking strikingly similar:
{{< tabs >}} {{< tab "With colon" >}}
Code with declaration and assignment operator:
```go
func main() {
	test := "Hello"
	{
		test := "World"
		fmt.Println(test)
	}

	fmt.Println(test)
}
```
which prints out:
```
World
Hello
```
{{< /tab >}}
{{< tab "Without colon" >}}
Code with just the assignment operator:
```go
func main() {
	test := "Hello"
	{
		test = "World"
		fmt.Println(test)
	}

	fmt.Println(test)
}
```
which prints out:
```
World
World
```
{{< /tab >}}
{{< /tabs >}}

Even though this is quite obvious in a scenario like this, something like this is quite complicated to find if it comes more complicated situations.
This can lead to just weird behaviour or even security vulnerabilities.

### But you can just use the other syntax
Yeah, I could. But the `:=` operator exists for a specific reason in the language and as long as it exists, there will be programmers that use this syntax.
I myself use this syntax as it is the main way things are being written in Go and in most cases it is just simpler to use than other things.

And even if I would not use that syntax, others will. It won't fix the core issue of that syntax. It would just shift the problem somewhere else. <br>
We want programming languages to be good and not just think that if there is an issue, we can just ignore it.

### I wouldn't do something like that
We make mistakes all the time. Everyone. No matter how experienced you are, you will make some stupid mistake where you have to debug way too long and at the end you think that the solution should have been obvious. This blog post is mostly a problem analysis where the language itself can lead to such moments as the language makes specific syntax not obvious enough so that some things can easily be overlooked.

To be honest. I had so many situations where I made an absolutely stupid mistake where the compiler did not warn me at all and I had to debug the program for hours only to find out that I should have written `<=` instead of `<`. It happens.

### Other situations
This operator is also not really an operator that declares and initializes the left side in every single case:

Let's say we have some code like this? Would it compile?
```go
func main() {
	test := 1

	test, err := strconv.Atoi("-42")

	_ = test
	_ = err
}
```
Any seasoned Go programmer would now instantly state: "Yes. This will compile." <br>
But would a person that didn't program in Go already know that?

On first glance a person would think that this is shadowing the variable. But Go does not support shadowing in the same scope. As this wouldn't compile:
```go
func main() {
	var test string = "A string"

	test, err := strconv.Atoi("-42")

	_ = test
	_ = err
}
```

This can obviously explained in different ways why this was implemented like this, but this is a situation where it isn't obvious what the compiler will do if you didn't have any experience in the language already.

## An example of a security vulnerability.
Where this can easily lead to issues is when you think you created a new variable in this scope, but didn't and you use parallelism.
In this case we would think that we can just use this variable, but we have created a race condition (a video about this exact situation can be found [here](https://www.youtube.com/watch?v=wVknDjTgQoo) and the CTF [here](https://2024.ctf.link/internal/challenge/fb03748d-7e94-4ca2-8998-a5e0ffcbd761/)).

In this example we have some code that uses the error variable in the outer scope for different things, but it is also used in the handler function for an HTTP request (which is asynchronous):
```go
func main() {
    // some code ...

    // The error variable is declared in the outer scope
    err := // something

    // some code ...

    http.HandleFunc("/get", func(w http.ResponseWriter, r *http.Request) {
        name := r.URL.Query().Get("name")

        //     v - Here is the problem as it uses the variable, instead of creating a new one.
		if err = checkPath(name); err != nil {
			http.Error(w, "checkPath :(", http.StatusInternalServerError)
			return
		}
    })

    // some code
}
```

In this case an attacker could abuse the fact that the `err` variable is "global". 
If they time the attack correctly, the `err` variable will be rewritten before it is being checked.

## Conclusion
I think that Go is quite a nice language to work with. It has nice syntax, is quite simple and you can create things easily. 
It is also a typed language, which is also a plus.

But this issue is, even though it is quite a small one, an issue that sometimes still bothers me. Maybe it is just a me-problem, but I think that this issue has some security implications as well. It is some syntax that is easily disregarded and overlooked, which makes it dangerous.
