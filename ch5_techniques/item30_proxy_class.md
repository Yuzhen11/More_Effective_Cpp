# Item 30: Proxy Classes

## Implementing a two-dimensional arrays

The built-in version need to know the diemensions in compile time.

```c++
template<class T>
class Array2D {
public:
    class Arrary1D {
    public:
        T& operetor[](int index);
        const T& operator[](int index) const;
        ...
    };
    Arrary1D operator[](int index);
    const Arrary1D operator[](int index) const;
    ...
};

Array2D<float> data(10,20);
cout << data[3][6];  // fine
```

Array1D is a proxy class. Its instances stand for one-dimensional arrays that, conceptually, do not exist.

## Distinguish reads from writes via operator[]

This is especially useful for reference-counted class, introduced in item 29. In item 29, we made conservative
assumption taht all calls to operator[] were for writes.

Now, we delay the decision after the operator[], we return a proxy class!

```c++
class String {
public:
    class CharProxy {
    public:
        CharProxy(String& str, int index);  // creation
        CharProxy& operator=(const CharProxy& rhs);  // lvalue uses
        CharProxy& operator=(char c);  // lvalue uses
        operator char() const;  // rvalue uses
    private:
        String& theString;  // string this proxy pertains to
        int charIndex;  // char within that string this proxy stands for
    };
    const CharProxy operator[](int index) const;  // for const Strings
    CharProxy operator[](int index);  // for non-const Strings
    friend class CharProxy;
private:
    RCPtr<StringValue> value;
};

String s1, s2;
cout << s1[5];  // legal, read
s2[5] = 'x';  // legal, write
s1[3] = s2[8];  // legal, write/read
```

```c++
const String::CharProxy String::operator[](int index) const {
    return CharProxy(const_cast<String&>(*this), index);
}
String::CharProxy String::operator[](int index) const {
    return CharProxy(*this, index);
}

String::CharProxy::operator char() const {
    return theString.value->data[charIndex];
}

String::CharProxy& String::CharProxy::operator=(const CharProxy& rhs) {
    if (theString.value->isShared()) {
        theString.value = new StringValue(theString.value->data);
    }

    theString.value->data[charIndex] = rhs.theString.value->data[rhs.charIndex];
    return *this;
}
```

## Limitations

```c++
String s1 = "Hello";
char* p = &s1[1];  // error
```
To eliminate this difficulty, you'll need to overload the address-of operator.

```c++
template<class T>
class Array {
public:
    class Proxy {
    public:
        Proxy(Array<T>& array, int index);
        Proxy& operator=(const T& rhs);
        operator T() const;
    };
    const Proxy operator[](int index) const;
    Proxy operator[](int index);
};

Array<int> intArray;
intArray[5] = 22;  // fine
intArray[5] += 5;  // error
++intArray[5];  // error
```

You must define each of these functions for the Array<T>::Proxy.

```c++
class Rational {
public:
    Rational(int numerator = 0; int denominator = 1);
    int numerator() const;
    int denominator() const;
    ...
};
Array<Rational> array;

cout << array[4].numerator();  // error!
```

```c++
void swap(char& a, char& b);
String s = "+C+";
swap(s[0], s[1]);
```

## Evaluation

Proxy classes allow you to achieve some types of behavior that are otherwise difficult or impossible to implement.

1. Multidimensional arrays
2. lvalue/rvalue differentiation
3. Suppression of implicit conversion (Item 5)

As function return values, proxy objects are temporaries.

proxy objects usually exhibit behavior that is subtly different from that of the real objects they represent.

std::vector<bool>'s operator[] will return a proxy class, and it cannot bind to bool& or auto&. Instead, we can use auto&&
