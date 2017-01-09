---
layout: post
title: C++11 Lambdas as Glue
---

Interfaces are a great way to decouple classes.
Sadly, interfaces in C++ are a pain.
Each class already needs a .h header file and an implementation .cpp file. 
Virtualizing methods and adding a pure base class is a lot of overhead
in a large code.
Further, rampant use of virtual functions encourages derived classes,
rather than composition.

Lambda functions, introduced in C++11, provide a direct and lightweight
way to make interfaces.

## The problem

As an example, consider an error logging class:

~~~ cpp
class ErrorLogger
{
public:
  LogError(const ErrorInfo& err_info);
  
  // More methods and fields we don't care about...
};
~~~ 

We could explicitly include this error logger class in every part of
out application that needs this functionality.
However, this exposes all the ErrorLogger interface, when we only
need a single function.

More critically, explicitly including ErrorLogger may lock in unwanted
side effects in other parts of our code. This is particularly troublesome
when our classes are used in other contexts, like unit testing or 
a reuse that would not need ErrorLogging.

## Old solution

The traditional solution to leave flexibility is to introduce 
a virtual base class:

~~~ cpp
class ErrorLoggerBase
{
public:
  virtual LogError(const ErrorInfo& err_info) = 0;
};

class ErrorLogger : public ErrorLoggerBase
{
public:
  virtual LogError(const ErrorInfo& err_info) override;
  
  // More methods and fields we don't care about...
};
~~~

This allows us the option of dependency injection.
We can create another derived class `ErrorLoggerBase` that ignores errors 
or records them for unit testing.
Each class that used `ErrorLogger` can have a pointer or reference to the
base class that can be set at compile time or run time.

~~~ cpp
class MyImportantClass
{
  // (other methods not shown) 
  
  void SetErrorLogger(ErrorLoggerBase* error_logger);
private:
  ErrorLoggerBase *m_error_logger;
};
~~~

This is a good programming practice, but a bit expensive.
We have introduced several new classes just to inject a different
behavior for the `ErrorLogger::LogError` function.

## New solution using a lambda function

With C++11, we should consider using functions as much as possible.
Java and many early C++ books put too much emphasis on objects.
Not everything is an object.
Moreover, even when you have objects, often other code should only know
about available functions, not details of other objects.

Here's an implementation of the error logging example that decouples these 
dependencies without introducing new classes.

~~~ cpp
class ErrorLogger
{
public:
  LogError(const ErrorInfo& err_info);

  // Get the error logging function for this object
  std::function<void(const ErrorInfo& err_info)> 
  GetErrorLoggingFunction()
  {
    return [this](const ErrorInfo& info) {LogError(info)}
  }

  // More methods and fields we don't care about...
};

class MyImportantClass
{
  // (other methods not shown) 
  
  std::function<void(const ErrorInfo&)> mLogError;
};
~~~ 

The elegance of this approach is that the classes that use ErrorLogger
do not even have to know about the ErrorLogger class.
All that is required is that a function with the signature 
`void(const ErrorInfo&)` exists.

The convenience method `ErrorLogger::GetErrorLoggingFunction()` returns
a lambda closure that encapsulates the method and the reference to
the ErrorLogger object.

A further convenience could be to include a null implementation of 
`mLogError` so that errors are ignored if the ErrorLogger is not included.
This mimics null-pointer behavior in the Swift language, which avoids
segmentation faults if mLogError was not set to an ErrorLogger object.

~~~ cpp
std::function<void(const ErrorInfo&)> mLogError 
    = [](const ErrorInfo&){};
~~~

## Summary

The important rule here is that functions are often simpler
than object interfaces.
This is an approach that is common in C, but with the type-checking and
machinery of C++ behind it.
I have found this use of lambdas particularly helpful when
refactoring C++ code, because it allows me to decouple classes and
inject dependencies without a lot of rewriting or introducing new classes
interfaces.
