# Item 26: Limiting the number of objects of a class

## Allowing zero or one Objects

Declare the ctors private.

Create a static object using non-member function or static member function or function in some namespace.

```c++
class Printer {
public:
    static Printer& thePrinter();
private:
    Printer();
    Printer(const Printer& rhs);
};
Printer& Printer::thePrinter() {
    static Printer p;
    return p;
}
```

static variable in function is preferred to static variable in class:

1. Create only when needed vs Create beforehand
2. Can controal the creation order.

In 90s, inline function has only internal linkage. So each compilation unit would have one static member... So more than one copy
of the static object may be created.

We can also use a counter to keep track how many objects we have created.

## Contexts for Object Construction

```c++
class ColorPrinter: public Printer {
    ...
};
Printer p;
ColorPrinter cp;
```
If we only allow one printer, then we cannot create a color printer after a printer.

An object can exist in three different contexts:

1. On their own
2. As a base class parts of more derived objects
3. Embedded inside larger objects

Prevent derivation: 
you can' derive from classes with private constructors.
```c++
class FSA {
public:
    static FSA* makeFSA() {
        return new FSA();
    }
    static FSA* makeFSA(const FSA& rhs) {
        return new FSA(rhs);
    }
private:
    FSA();
    FSA(const FSA& rhs);
};
```

## Allowing objects to come and go

Allow the object to create and destroy and then create again.

Combine the object-counting code and pseudo-constructors
```c++
class Printer {
public:
    class TooManyObjects {};  // for exception
    // pseudo-constructors
    static Printer* makePrinter() {
        return new Printer;
    }
    ~Print() {
        --numObjects;
    }
private:
    static unsigned int numObjects = 0;
    Printer() {
        if (numObjects >= 1) {
            throw TooManyObjects();
        }
        ++numObjects;
    }
    Printer(const Printer& rhs);
};
```
This technique is easily generalized to any number of objects.

## An object-counting base class

Move the logic of object-counting to a base class

```c++
template<class BeingCounted>
class Counted {
public:
    class TooManyObjects();  // for exception
    static int objectCount() { return numObjects; }
protected:
    Counted() {
        init();
    }
    Counted(const Counted&) {
        init();
    }
    ~Counted() {
        --numObjects;
    }
private:
    static int numObjects;
    static int maxObjects;
    void init() {
        if (numObjects >= maxObjects) throw TooManyObjects();
        ++numObjects;
    }
};
```
The template in the base class is used to create the static instance for every subclass type!
```c++
class Printer: private Counted<Printer> {
public:
    // pseudo-constructors
    static Printer* makePrinter();
    static Printer* makePrinter(const Printer& rhs);
    ~Printer();
    using Counted<Printed>::objectCount;
    using Counted<Printed>::TooManyObjects;
private:
    Printer();
    Printer(const Printer& rhs);
};
```

1. private inheritance: Means we won't use Counted<Printer>\* to point to a Printer and no need to give a virtual destructor
2. the using statement: make the function available in private inheritance

```c++
// it's easy to take care of numObjects
template<class BeingCounted>
int Counted<BeingCounted>::numObjects;  // defines numObjects and automatically initialized to 0

// Require the clients of the class to provide the appropriate initialization.
int Counted<Printer>::maxObjects = 10;
int Counted<FileDescriptor>::maxObjects = 16;
```
