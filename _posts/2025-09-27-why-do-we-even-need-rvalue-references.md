---
layout: post
title: "Why Do We Even Need Rvalue References"
author: "WhalePark"
tags: [c++,programming]
---


### Distinguishing lvalues and rvalues
One of the big barriers that frustrates people learning C++ is expression classification between `rvalue` and `lvalue`. Strictly speaking, the division is between `glvalue` and `rvalue`. `glvalue` comprises `lvalue` and `xvalue`, and `rvalue` consists of `prvalue` and `xvalue`. So if you want a proper MECE-style classification, every C++ expression is either an `lvalue`, `xvalue`, or `prvalue`. For now, let’s ignore the fine details and think of it simply as just `lvalues` and `rvalues`. `xvalue` and `prvalue` can mostly be treated as the same category.

The first explanation people usually get is that an `lvalue` is something that can appear on the left-hand side of an assignment, and an `rvalue` is something that appears on the right-hand side. That’s correct, but it doesn’t explain why this classification is so crucial in C++. For example:

```
int x = 3; // x: lvalue, 3: rvalue
```

Okay, we know that, but so what? After all, other programming languages have the concept of immediate values as well. Why do we talk about `rvalue` so often only in C++ context?

### Reference Types in C++
C++ underwent a major shift around C++11, and one of the biggest changes was the introduction of rvalue references. (Note: this does not mean the introduction of rvalues. The concept of rvalues exists in other languages as well. For instance, in Python, when you write `x: int = 3`, the 3 is an `rvalue`.)

Even before C++11, there were references, for example:
```
int x = 3;
int &y = x;
int &z = y;
printf("%d %d %d\n", x, y, z);
// 3 3 3
```

But as you know, rvalue references are written as `T&&`. For example:
```
int &&x = 3;
printf("%d\n", x); // prints 3

//// int &&y = x; // Not valid: you cannot bind an lvalue to an rvalue reference
int &&y = std::move(x);
printf("%d\n", x); // prints 3
```
That works, but at first glance, it doesn’t seem particularly useful. It’s syntactically valid, but rvalue references weren’t invented to make integer initialization more convenient. In older C++, creating or passing objects often involved mandatory copies. For instance:
```
MyObject o1 = MyObject(x, y, z, w, ...);
```
Here, MyObject is created on the caller’s stack, then (without compiler optimizations like NRVO) copied to create another replica for the callee’s stack.
```
MyObject(const MyObject &o) {}
// In old C++, only this was possible.
// A deep copy of resources is performed inside.
MyObject(MyObject &&o) {}
// Since rvalue references, this code is also possible.
// Resources can just be taken and moved instead.
```

Another example:
```
someFunction(std::string("hello")); // In a case like this:

void someFunction(std::string &str);
// Not allowed. Cannot bind a temporary to a non-const lvalue reference.
// But if it were 'void someFunction(const std::string &str)', it would be allowed.

void someFunction(std::string str);
// Allowed, but 'std::string("hello")' is constructed twice
// (since C++17, only constructed once due to guaranteed copy elision).

void someFunction(std::string &&str);
// Allowed, and the temporary is constructed once and bound as an rvalue reference inside the function.
```

In fact, even in C++98, the two snippets below produce the same output, but one involves value copying while the other involves reference binding, or aliasing.
```
int x = 3; // An rvalue is assigned to an lvalue
int y = x; // The rvalue x is implicitly converted to an lvalue,
           // then copy-initialization is performed
printf("%d\n", y); // 3

int x = 3;     // An rvalue is assigned to an lvalue
int &y = x;    // The memory space of the lvalue x is bound to the lvalue reference y
printf("%d\n", y); // 3
```

Actually, the following code is also valid and makes sense.
```
int x = 3;
int y = std::move(x);
// Even if you cast to an rvalue with std::move, the integer is still copied,
// just like an implicit rvalue conversion, without any special operator behavior.
```
Of course, since this is just a primitive type without any internal resources, it doesn’t really mean much. `std::move` is simply an operator/syntax that casts an `lvalue` into an `rvalue`. It is not fundamentally different from other kinds of type casting.

Forwarding references and reference collapsing rules are also useful and interesting. The distinction between xvalues and prvalues is also somewhat amusing, but since both can bind to rvalue references, in practice there isn’t much difference in how they’re used. At most, knowing these fine-grained rules gives you the small satisfaction of mastering C++’s quirks.

I’m not a C++ PhD (not that such a thing even exists), and this post isn’t meant to be a formal technical document, so there may well be mistakes here and there. Please don’t quote this directly as an authoritative source. I picked up this knowledge piece by piece while using libraries and searching online, so my understanding may be looser or stricter than what the C++ standard actually specifies.
