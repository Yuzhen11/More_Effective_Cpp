# Ch3: Exceptions

In C, Error codes have sufficed. But user can ignore it.

Use setjmp and longjmp? destructors for local objects won't be called.

So, if you need a way of signaling exceptional conditions that cannot be ignored, and if you must ensure
that local destructors are called when searching the stack for code that can handle exceptional conditions,
you need C++ exceptions. 

## Item 9: Use destructors to prevent resource leaks

Use smart pointer objects instead of raw pointers, and you won't have to worry about heap objects not being deleted,
not even when exceptions are thrown.

```c++
void processAdoptions(istream& dataSource) {
    while (dataSource) {
        ALA* pa = readALA(dataSource);
        pa->processAdoption();
        delete pa;
    }
}

void processAdoptions(istream& dataSource) {
    while (dataSource) {
        auto_ptr<ALA> pa = readALA(dataSource);
        pa->processAdoption();
    }
}
```

Create a class whose constructor and destructor acquire and release the resource.

```c++
class WindowHandle {
public:
    WindowHandle(WINDOW_HANDLE handle): w(handle) {}
    ~WindowHandle() { destroyWindow(w); }
    operator WindowHandle() {return w;}  // implicit conversion
private:
    WINDOW_HANDLE w;
    WindowHandle(const WindowHandle&);  // prevent copy
    WindowHandle& operator=(const WindowHandle&);  // prevent copy

};
```
What happens if an exception happen in constructor or destructor?

## Item 10: Prevent resource leaks in constructors

If an exception is thrown during creation of an object, destructor will never be called. Never.

C++ destroys only fully constructed objects.

We need to handle it by ourselves.
```c++
BookEntry::BookEntry(const string& name, const string& address, const string& imageFileName, const string& audioClipFileName)
: theName(name), theAddress(address), theImage(0), theAudioClip(0) {
    try {
        if (imageFileName != "") {
            theImage = new Image(imageFileName);
        }
        if (audioClipFileName != "") {
            theAudioClip = new AudioClip(audioClipFileName);
        }
    }
    catch(...) {
        delete theImage;
        delete theAudioClip;
        throw;
    }
}
```

What if the pointer is const?
```c++
class BookEntry {
public:
private:
    Image* const theImage;  // pointers are now cosnt
    AudioClip* const theAudioClip;
};

BookEntry::BookEntry(const string& name, const string& address, const string& imageFileName, const string& audioClipFileName)
: theName(name), theAddress(address), 
theImage(imageFileName != "" 
         ? new Image(imageFileName)
         : 0), 
theAudioClip(audioClipFileName != ""
         ? new AudioClip(audioClipFileName)
         : 0) 
         {}
```
Previous solution cannot work.

The best way is to use smart pointer! Again!

```c++
class BookEntry {
public:
private:
    const auto_ptr<Image> theImage;
    const auto_ptr<AudioClip> theAudioClip;
};
```

If an exception is thrown during the initialization of theAudioClip, theImage is already a fully constructed object, so it 
will automatically be destroyed, just like theName, theAddress.

No need to write destructor.

Use of smart pointer is easy and robust in the face of exceptions!

## Item 11: Prevent exceptions from leaving destructors

Two situations a constructor is called.

1. when an object is destroyed under "normal" conditions, e.g., when it goes out of scope or is explicitly deleted.
2. when an object is destroyed by the exception-handling mechanism during the stack-unwinding part of exception propagation.

If contol leaves a destructor due to an exception while another exception is active, C++ calls the terminate function.

We need to prevents exceptions thrown from a destructor.

```c++
Session::~Session() {
    try {
        logDestruction(this);
    }
    catch(...) {}
}
```

Why we should keep exceptions from propagating out of destructors?

1. Prevents terminate from being called during the stack unwinding part of exception propagation.
2. Ensure that destructors always accomplish everything they are supposed to accomplish.

## Item 12: Understand how throwing an exception differs from passing a parameter or calling a virtual function

```c++
catch(Widget& w) {
    ...
    throw;  // prefer this one. No copy is made. if the exception originally 
            // thrown was of type SpecialWidget, it will proprocate a SpecialWidget exception
}

catch(Widget& w) {
    ...
    throw w;  // throw a new exception which is type Widget.
}

```

Three primary differences:

1. Exception objects are always copied; when caught by value, they are copied twice.

    Need to copy even if we catch by reference.

2. Objects thrown as exceptions are subject to fewer form of type conversion than are objects passed to functions.

    Cannot catch int type if you write double.
    
3. catch clauses are examined in the order in which they appear in the source code.

    Should not put base class before derived class, otherwise, the derived class catch clause can never be executed.

## Item 13: Catch exceptions by reference

Why not catch by pointer?

The object being thrown should exist out of the scope. So, clients may pass the address of a global or static object,
others may pass the address of an exception on the heap. To delete or not?

Why not catch by value?

Slicing problem and more copy.

So, catch exceptions by reference!

## Item 14: Use exception specifications judiciously

If a function throws an exception not listed in its exception specification, the special function `unexpected` is automatically
invoked. The default behavior is to call terminate and then abort.

It's easy to write functions that make terrible thing occur. Compilers only partially check exception usage for consistency
with exception specifications.
```c++
extern void f1();
void f2() throw(int) {
    f1();  // f1 may throw something besides an int
}
```
1. A good way to start is to avoid putting exception specifications on templates that take type arguments. Because various functions
may be called.

2. Exception specifications on user defined callback functions.

3. Handle exceptions "the system" may throw. E.g. bad_alloc in operator new.

If preventing unexpected exceptions isn't practical, you can exploit the fact that C++ allows you to replace unexpected exceptions
with exceptions of a different type.

```c++
class UnexpectedException {};  // all unexpected exception objects will be replaced by objects of this type

void convertUnexpected() {  // function to call if an unexpeted exception is thrown
    throw UnexpectedException();
}
set_unexpected(convertUnexpected);
```
The unexpected exception is then replaced by a new exception of type UnexpectedException.

Another way to translate unexpected exception into a well known type: if the unexpected function's replacement rethrows the
current exception, that exception will be replaced by a new exception of the standard type bad_exception.
```c++
void convertUnexpected() {
    throw;
}
set_unexpected(convertUnexpected);
```
New exception will be propagated instead of the original one.

Exception specifications have another drawback: They result in unexpected being invoked even when a higher-level caller
is prepared to cope with the exception that's arisen.

```c++
class Session {
public:
    ~Session();
private:
    static void logDestruction(Session* objAddr) throw();  // throw nothing
};
Session::~Session() {
    try {
        logDestruction(this);
    }
    catch(...) {}
}
```

Now, suppose some function called by logDestruction throws an exception that logDestruction fails to catch. When this
unanticipated exception propagates through logDestruction, unexpected will be called, and by default, that will result in
termination of the program. The author try to catch all possible exceptions, but he fails.

Summary of exception specifications:
They provide excellent documentation on the kinds of exceptions a function is expected to throw, and for situations in which
violating an exception specification is so dire as to justify immediate program termination, they offer that behavior by default.
They are only partly checked by compliers and easy to violate inadvertently. They prevent high-level exception handlers from
dealing with unexpected exceptions, even when they know how to.

Exception specifications are a tool to be applied judiciously.

## Item 15: Understand the costs of exception handling

To handle exceptions at runtime, programs must do a fair amount of bookkeeping.

try blocks and exception specifications also incur cost.

No need to worry too much, because exception should be rare.

You can also compile without support for exceptions when that is feasible; limit your use of try blocks and exception spcifications
to those locations where you honestly need them; and throw exceptions only under conditions that are truly exceptional.
