---
layout: post
title: Variadic Templates and Recursion in C++11

---


C++11 is a fun language! I first learned about the language when I saw some
source code on [Garnet Chan‘s][ProfGarnetChan]
 research poster at an NSF SI2 function. He
explained that the “auto” keyword was one of many nice features in C++11 and I
should start using it. Since then I’ve had a blast using this much-improved C++
language. If you are a C++ programmer I would recommend Bjarne Stroustrup’s
[talk on  “C++11 Style” on YouTube][C++Style].

In this blog post I want to describe an advanced feature of C++11, variadic
templates, and how they work with recursive programming. 
While recursive techniques, especially [tail-call recursion][TailCall],
are very common in functional
languages, most C programmers are more comfortable with an imperative style.
Using C++11 variadic templates effectively often requires tail call recursion.

The problem we are going to solve is a simple one. Consider this handy templated
function, `ToString`:

~~~ cpp
template <class T1, class T2, class T3>
std::string ToString(const T1& arg1, const T2& arg2, const T3& arg3) {
   std::stringstream buffer;
   buffer << arg1 << arg2 << arg3; return buffer.str(); 
}
~~~

This function takes three objects of any type and concatenates them into a 
string using their `operator<<(stream&, OBJ)` functions. For example,

~~~ cpp
ToString("I give this a ", 3, "-star rating")
~~~

would return the string “I give this a 3-star rating”.

The problem: while this function is straightforward to read, it is hard-coded
for exactly three arguments. We could code this up for a dozen specialized
versions and hope the user never wanted more than 12 arguments. That’s ugly and
violates the DRY principle (don’t repeat yourself!). The variadic function from
C (most well known in printf syntax) does take a variable number of arguments,
but they cannot be objects.

C++11 provides a beautiful solution. A variadic template lets you define a
generic function with an arbitrary number of arguments, and the compiler
generates the necessary specializations at run time. For example, we could
write,

~~~ cpp
template<class... Args>
std::string ToString(const Args&... args)
{
   // Horrors!  This compiles, but we can't do anything  
   // with args because we don't know the types!a
   // This special case is only useful when args is empty. 
   return ""; 
 }
~~~

However, this general function is useless, because we can’t work directly with
the unknown args. In variadic templates, we usually just pass the variadic args
to another function! The trick in this case to specify the lead argument (known
as the “head”) explicitly, then leave the other arguments in the tail.

~~~ cpp
template<class T1, class... Args> 
std::string ToString(const T1& head, const Args&... tail) 
{
   std::stringstream buffer; return ToString(buffer << head, tail);
}
~~~

Note the recursion in the last line of the function.
Having used a stringstream buffer to convert the head object to a 
string, we call `ToString` again, this time the first argument is the 
stringstream buffer.
Now we need another specialization:

~~~ cpp
template<class T1, class... Args>
std::string ToString(
    const std::stringstream& buffer,
    const T1& head, const Args&... tail) {
  return ToString(buffer << head, tail);
}
~~~

This is beautiful, from an algorithmic point of view. Once we have the
buffer, we can grab the next element and append it to the buffer, then repeat
recursively. The last step is to close the recursion, when we only have one
argument that is the buffer,

~~~ cpp
std::string ToString(const std::stringstream& buffer) 
{
   return buffer.str();
}
~~~

This code should just work. However, the way C++ matches templates, it can be
important to put the specializations first. The final, working code is

~~~ cpp
template<class... Args> std::string
ToString(const Args&... args) {
  // This special case is only called when args is empty.
  return ""; 
}

std::string ToString(const std::stringstream& buffer)
{ 
  return buffer.str(); 
}

std::string ToString(
  const std::stringstream& buffer, const T1& head, 
  const Args&... tail)
{ 
  return ToString(buffer << head, tail); 
}

template<class T1, class... Args>
std::string ToString(const T1& head, const Args&... tail)
{ 
  std::stringstream buffer; 
  return ToString(buffer << head, tail);
}
~~~

This example is not the prettiest code, but it should clearly
demonstrate how variadic templates can be used in generic programming. The
ugliness comes from a template syntax, and is an artifact of the strong typing
in C++. The four short functions here are typical of functional languages, where
pattern matching and recursion are used more than in procedural languages.  Once
this is implemented, however, 
the real simplicity is that the `ToString` function
just works. For applications that require  C++, techniques like this can be used
to lean on the compiler to build general solutions that were not possible before
C++11.

[ProfGarnetChan]: http://chemists.princeton.edu/chan/
[C++Style]: http://youtu.be/0iWb_qi2-uI
[TailCall]: http://en.wikipedia.org/wiki/Tail_call
