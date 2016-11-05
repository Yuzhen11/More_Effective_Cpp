# Ch2: Operators

## Item 5: Be wary of user-defined conversion functions

Two kinds of functions allow compilers to perform implicit type conversions:
single-argument constructors and implict type conversions operators.

```c++
class Name {
public:
    Name(const string& s);
    ...
};

class Rational {
public:
    operator double() const;
};
```

Why implicit type conversion operators are not cool?
```c++
Rational r(1,2);
cout << r;
```
If operator<< is not overloaded for Rational, r will be convert to double...

A better way (not to provide the implicit conversion operators):
```c
class Rational {
public:
    double asDouble() const;
};
```

Just as std::string cannot implicitly convert to char\*, need to use c_str().

To prevent single-argument constructors to perform implicit type conversions, use `explicit` keyword.

Or you can provide another class to represent the single-arugment type. Make use of the feature that no sequence of
conversions is allowed to contain more than one user-defined conversion.

```c++
template<class T>
class Array {
public:
    class ArraySize {
    public:
        ArraySize(int numElements): theSize(numElements) {}
        int size() const {return theSize;}
    private:
        int theSize;
    };
    Array(ArraySize size);
};
Array<int> a(10);  // ok
if (a == 4) // not ok
```

Classes like ArraySize are often called proxy class.

## Item 6: Distinguish between perfix and postfix forms of increment and decrement operators

```c++
class UPInt {
public:
    UPInt& operator++(); // prefix ++
    const UPInt operator++(int);  // postfix ++

    UPInt& operator+=(int);  // += operator
};

// prefix form: increment and fetch
UPInt& UPInt::operator++() {
    *this += 1;
    return *this;
}
// postfix form: fetch and return
const UPInt UPInt::operator++(int) {
    UPInt oldValue = *this;
    ++(*this);
    return oldValue;
}
```

Easy to see why postfix ++ should return const:
```c++
UPInt i;
i++++;
```

1. Be careful about the return type.

2. postfix operators should be implemented in terms of the prefix operators.


## Item 7: Never overload &&, |, or , 

## Item 8: Understand the differnt meanings of new and delete

The difference between the new operator and operator new.

```c++
string* ps = new string("abc");
```
The new you are using is the new operator.

It calls operator new to allocate memory.

```c++
void* operator new(size_t size);

// call it
void* rawMemory = operator new(sizeof(string));
```
The return type is void\*, because this function returns a pointer to raw uninitialized memory.

Like malloc, operator new's only reponsiblity is to allocate memory.

Placement new: you have some raw memory that's already been allocated, and you need to constuct an object in the 
memory you have.
```c++
Widget* constructWidgetInBuffer(void* buffer, int widgetSize) {
    return new (buffer) Widget(widgetSize);
}
```

If you want to create an object on the heap, use the new operator. It both allocates memory and calls a constructor for
the object.

If you only want to allocate memory, call operator new; no constructor will be called. 

If you want to customize the memory allocation that takes place when heap objects are created, write you own version of
operator new and use the new operator; it will automatically invoke your custom version of operator new.

If you want to construct an object in memory you've already got a pointer to, use placement new.

operator new and operator delete is the C++ equivalent of calling malloc and free.
