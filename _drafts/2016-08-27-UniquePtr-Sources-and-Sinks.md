---
layout: post
title: UniquePtr Sources and Sinks
---
One of the biggest challenges of designing robust applications in C++ is
careful memory management.
C++ gives developers great control over memory allocation, layout, and 
deallocation.
The language is also notorious for memory leaks and segmentation faults,
putting a heavy burden on all developers to design and code carefully.
The C++11/14 feature, `std::unique_ptr`, provides a simple but powerful
mechanism to encode object ownership rules in your software design.

## The problem

In C++98, the [factory method pattern][FactoryMethod] is used to
encapsulate the logic of object creation.

~~~ cpp
class Factory
{
  /** This factory method creates a Widget object */
  virtual Widget* MakeWidget() const;
}
~~~

The class `Factory` or its subclasses encapsulate the logic of object
creation, which is executed by the `MakeWidget` method.
While this is a common pattern, returning a bare pointer alerts 
experienced C++ developers to several common issues:

1. Through design convention or explicit documentation comments, users
   of this factory need to know that they are responsible for calling
   `delete` on the returned `Widget` object.
2. User must make sure they do not let the returned pointer value
   go out of scope before calling delete. They also must make sure they
   call delete exactly once. Structured coding can help pairing up
   the `MakeWidget` call and the corresponding delete, but in practice
   this is a common source of memory leaks.
3. Users must be careful to delete the `Widget` object only once.
   Often, the pointer value is copied to another variable, and it is critical
   to keep track of the owner that is responsible for the object lifetime.

In C++11, [unique pointers][UniquePtr] offer an elegant solution to these three
problems. By returning a `std::unique_ptr<Widget>`, the transfer of 
ownership is explicit, the `Widget` object is deleted when its pointer
goes out of scope, and unique ownership is enforced.

In this blog post, I describe the mechanics of using `std::unique_ptr`
with factory methods. Two concepts are important: (1) The notion 
of object sources and sinks, for methods that create or receive ownership
of objects, and (2) `std:move` semantics for this pattern.

## Object sources

The `MakeWidget` method is an example of an object source,
a function that creates a new object and transfers ownership
to the calling code.

## Object sinks

Unique pointers are also convenient for defining object sinks,
functions at accept and take ownership of allocated objects.

## Conclusion

[FactoryMethod]: https://en.wikipedia.org/wiki/Factory_method_pattern
[UniquePtr]: http://en.cppreference.com/w/cpp/memory/unique_ptr
