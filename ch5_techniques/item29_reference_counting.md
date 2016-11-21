# Item 29: Reference counting

## Implementing Reference Counting

A reference-counting copy-on-write string class is implemented.

```c++
class String {
public:
    ...
    String(const String& rhs);
    String& operator=(const String& rhs);
private:
    struct StringValue {
        int refCount;
        char* data;

        StringValue(const char* initValue);
        ~StringValue();
    };
    StringValue* value;
};
```

### Copy-on-Write

```c++
class String {
public:
    char operator[](int index) const;  // for const Strings
    char& operator[](int index);  // for non-const Strings
};
```
There is no way for C++ compilers to tell us whether a particular use of operator[] is for a read or a write, so we 
must be pessimistic assume that all calls to the non-const operator[] are for writes.

Proxy classes can help us differentiate reads from writes!!! Item 30.

### Pointers, References, and Copy-on-Write

It's still possible that we can break the rule.

```c++
String s1 = "Hello";
char* p = &s1[1];  // a pointer to the value
String s2 = s1;
// Later when we modify the value pointed by p, the value of s2 will also be modified, since they are the same value
```

Solutions:

1. Ignore it.
2. Describe the limitation in docs and ignore it.
3. Add a variable `bool shareable;` in the StringValue to indicate whether there's a pointer pointing to it.

## A Reference-Counting Base Class

Design a base class `RCObject`, StringValue need to inherit from this class
```c++
class RCObject {
public:
    PCObject();
    RCObject(const RCObject& rhs);
    RCObject& operator=(const RCObject& rhs);
    virtual ~RCObject() = 0;

    void addReference();
    void removeReference();
    void markUnshareable();
    bool isShareable() const;
    bool isShared() const;
private:
    int refCount;
    bool shareable;
};
```
A `RCPtr` class to use the `RCObject` class's function to add or remove reference.
```c++
template<class T>
class RCPtr {
public:
    RCPtr(T* realPtr = 0);
    RCPtr(const RCPtr& rhs);
    ~RCPtr();
    RCPtr& operator=(const RCPtr& rhs);
    T* operator->() const;
    T& operator*() const;
private:
    T* pointee;
    void init();
};
```

```c++
class String {
public:
    ...
private:
    struct StringValue: public RCObject {
        char* data;
        StringValue(const char* initValue);
        StringValue(const StringValue& rhs);
        void init(const char* initValue);
        ~StringValue();
    };
    RCPtr<StringValue> value;
};
```
## Adding Reference Counting to existing classes
Apply the maxim that most problems in computer science can be solved with an additional level of indirection.
