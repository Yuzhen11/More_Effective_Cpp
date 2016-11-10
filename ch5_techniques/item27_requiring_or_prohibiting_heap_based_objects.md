# Item 27: Requiring or prohibiting heap-based objects

## Requiring heap-based objects

The straightforward way is to declare the constructors and destructor private. This is overkill.

Only need to declare the destructor to be private.

Introduce a privileged pseudo-destructor function that has access to the real constructor.

```c++
class UPNumber {
public:
    UPNumber();
    UPNumber(int initValue);

    // pseudo-destructor
    void destroy() const { delete this; }
private:
    ~UPNumber();
};

UPNumber n;  // error! (legal here, but illegal when n's dtor is later implicitly invokded)

UPNumber* p = new UPNumber;  // fine
delete p;  // error! attempt to call private destructor
p->destroy(); // fine
```

Restricting access to a class's destructor or its constructors prevents the creation of non-heap objects, it also
prevents both inheritance and containment as told in Item 26.

Easy to solve:
```c++
class UPNumber { ... };  // declare dtor protected
class NonNegativeUPNumber : public UPNumber { ... };  // now ok, derived classes have access to protected members

class Asset {
public:
    Asset(int initValue): value(new UPNumber(initValue)) {  // fine
    }
    ~Asset() {
        value->destroy();  // also fine
    }
private:
    UPNumber* value;  // using pointer instead
};
```

## Determine whether an object is on the heap

No way for constructor to detect that whether it is invoked on heap or not.

Three ways:

1. Overload the new operator
2. Use the memory organization
3. Use a table to track whether the memory is allocated via new.

Method 1: How about overload the new operator calls?

```c++
class UPNumber {
public: 
    class HeapConstraintViolation {};  // for exception 
    static void* operator new(size_t size) {
        onTheHeap = true;
        return ::operator new(size);
    }
    UPNumber() {
        if (!onTheHeap) {
            throw HeapConstraintViolation();
        }
        onTheHeap = false;  // clear flag for next obj
    }
private:
    static bool onTheHeap;  // flag to check whether the object is constructed on the heap
};
bool UPNumber::onTheHeap = false;
```

The problem is when you new an array, you call the new operator once, but the constructor many times.

```c++
UPNumber* numberArray = new UPNumber[100];
```

And the compiler may reorder the execution order or some statement.
```c++
UPNumber* pn = new UPNumber(*new UPNumber);
```

Method 2: Make use of the memory organization

Stack memory is in high address and heap memory is in low address.

```c++
bool onHeap(const void* address) {
    char onTheStack;
    return address < &onTheStack;
}
```
However, static objects are also in the low address. So we cannot tell.

```c++
char* pc = new char; // heap object: onHeap(pc) will return true
char c; // stack object: onHeap(pc) will return false
static char sc; // static object: onHeap(pc) will return true
```

Method 3: Use a table to track every memory allocated

A mixin("mix in") class is one that provides a single well-defined capability and is designed to be compatible with
any other capabilities an inheriting class might provide.

```c++
class HeapTracked {
public:
    class MissingAddress{};
    virtual ~HeapTracked() = 0;

    static void* operator new(size_t size);
    static void operator delete(void* ptr);

    bool isOnHeap() const;
private:
    typedef const void* RawAddress;
    static list<RawAddress> address;
};
```

## Prohibiting heap-based objects

Make it impossible to call the new operator:
```c++
class UPNumber {
private:
    static void* operator new(size_t size);
    static void operator delete(void* ptr);
};

UPNumber n1; // ok
static UPNumber n2; // also ok
UPNumber* p = new UPNumber; // error, attempt to call private opeator new
```
We could also declare operator new[] and operato delete [] private as well.

Declaring opeartor new private often also prevents UPNumber objects from being instantiated as base class parts
of heap-based derived class object. That's because operator new and operator delete are inherited, so if these
functions aren't declared public in a derived class, that class inherits the private versions declared in its base.

UPNumber's operator new is private has no effect on attempts to allocate objects containing UPNumber objects as members:
```c++
class Asset {
public:
    Asset(int initValue);
private:
    UPNumber value;
};
Asset* pa = new Asset(100);  // fine, calls Asset::operator new or ::operator new, not UPNumber::operator new
```
