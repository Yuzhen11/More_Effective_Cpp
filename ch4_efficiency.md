# Ch3: Efficiency

Two types of efficiency:
Language-independent and C++-related

## Item 16: Remember the 80-20 rule

The overall performance of your software is almost alwasy determined by a small part of its constituent code.

Use Profilers

## Item 17: Consider using lazy evaluation

Four examples:

1. To avoid unnecessary copying of objects
    
    reference counting

2. To distinguish reads from writes using operator[]

3. To avoid unnecessary reads from databases

    Lazy fetching, fetch the field on demand

    mutable keywords may be used: Update a field pointer in const member function

4. To avoid unnecessary numerical computations

Lazy evalution is only useful when there's a resonable chance your software will be asked to perform
computations that can be avoided.

## Item 18: Amortize the cost of expected computations

Over-eager evaluation: cache min, max, avg

Caching: cache data from database

Prefetching: allocate more memroy than needed

Over-eager evaluation is a technique for improving the efficiency of programs when you must support operations
whose results are almost always needed or whose results are often needed more than once.

## Item 19: Understand the origin of temporary objects

Unnamed objects usually arise in one of two situations: 

1. When implicity type conversions are applied to  make function calls succeed.
2. When functions return objects.

```c++
int countChar(const string& str, char ch);

char buffer[MAX_STRING_LEN];
countChar(buffer, ch);
```
Temporary object will be created and destroyed!

It's convenient but has unnecessary expense.

Solutions:

1. Redesign your code so conversions like these can't take place. Item 5
2. Modify your software so that the conversions are unnecessary. Item 21

These conversions occur only when passing objects by value or reference-to-const. They do not occur
when passing an object to a reference-to-non-const parameter.

An error will be reported.

In this case, no temporary is created. Pass by reference means it's ok to modify it, but you are modifying a temporary.

For return value, there are return value optimization.

Unnamed objects offer compilers more flexibility in optimization than named objects.

Train yourself to look for places where temporary objects may be created. Anytime you see a reference-to-const parameter,
the possiblity exists that a temporary will be created to bind to that parameter.

## Item 20: Facilitate the return value optimization

Old topic, also in Effective C++.

Don't try to return pointer or reference when you need to return an object.

Just return the value!!! Return value optimization will help.


## Item 21: Overload to avoid implicit type conversions
```c++
class UPInt {
public:
    UPInt();
    UPInt(int value);
    ...
};
const UPInt operator+(const UPInt& lhs, const UPInt& rhs);

UPInt upi1, upi2;
UPInt upi3 = upi1 + upi2;
upi3 = upi1 + 10;
upi3 = 10 + upi1;
```

They do so through the creation of temporary object ot convert the integer 10 into UPInts.

If you want implicit type conversions without incurring any cost for temporaries?

Overload!

```c++
const UPInt operator+(const UPInt& lhs, int rhs);
const UPInt operator+(int lhs, const UPInt& rhs);

upi3 = upi1 + 10;  // find, no temporary for upi1 or 10
upi3 = 10 + upi1;  // find, no temporary for upi1 or 10
```

```c++
const UPInt operator+(int lhs, int rhs);  // error
```

Every overloaded opeartor must take at least one argument of a user-defined type.

## Item 22: Consider using op= instread of stand-alone op

```c++
class Rational {
public:
    Rational& operator+=(const Rational& rhs);
};
// operator+ implemented in terms of operator+=
const Rational operator+(const Rational& lrs, const Rational& rhs) {
    return Rational(lhs) += rhs;
}
```

1. Assignment versions of operators are more efficient than stand-alone versions. (no need to retuen a new object)
2. Allow clients of your classes to make the difficult trade-off between efficiency and convenience.
```c++
result = a+b+c+d;  // 3 temporary objects

result = a;
result += b;  // no temporary needed
result += c;
result += d;
```

Assignment versions of operators (such as operator+=) tend to be more efficient than stand-alone versions of those
operators(e.g. operator+).

Interesting to know that long time ago, return value optimization is not that powerful and can only work for unnamed objects.

As a library designer, you should offer both. As an application developer, you should consider using assignment versions.

## Item 23: Consider alternative libraries

Different libaraies offering similar functionality often feature different performance trade-offs.

Different libraries embody different desgin decisions regarding efficiency, extensibiltiy, portability, type safety, and other issues.

Different designers assign different priorities to these criteria. They thus sacrifice different things in their designs.

## Item 24: Understand the costs of virtual functions, multiple inheritance, virtual base classes, and RTTI

**virtual tables and virtual table pointers**

A vtbl is usually an array of pointers to functions. Each class that declares or inherits a virtual functions has its
own vtbl. The entries in a class's vtbl are pointers to the implementations of the virtual functions for that class.

Where to put the vtbl?

Most programs and libraries are created by linking together many object files.

Which object file should contain the vtbl for any given class?

1. Generate a copy of the vtbl in each object file. The linker strips out duplicate copies.
2. A heuristic to determine which object file should contain. A class's vtbl is generated in the objec file containing the 
first non-inline non-pure virtual function in that class.

Each object whose class declares virtual functions has a vptr points to the virtual table for that class.

virtual function calls:

1. Follow the object's vptr to vtbl.
2. Find the pointer in the vtbl that correspons to the function being called.
3. Invoke the function pointed by the pointer.

So, it seems still efficient.

virtual can not work with inline. runtime vs compilation.

**virtual base class**

Multiple inhertance often leads to the need for virtual base classes.

Implementations of virtual base classes generally use pointers to virtual base class parts as the means for 
avoiding the replication, and those pointers may be stored inside your objects.

Three vptrs and two pointers to virtual base class!

**RTTI**

RTTI let us discover information about objects and classes at runtime.

There only needs to be a single copy of the RTTI information for each class.

RTTI was designed to be implementable in terms of a class's vtbl.

Additional entry in each class vtbl.

Don't try to implement this functionalities by yourself.

If you need these feature, you need to pay for it.
