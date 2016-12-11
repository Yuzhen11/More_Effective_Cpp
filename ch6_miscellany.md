# Miscellany

## Item 32: Program in the future tense

Good software adapts well to change.

One way to do this is to express design constraints in C++ instand of (or in addition to) comments or other documentation.

Given that things will change, write classes tha tcan withstand the rough-and-tumble word of software evolution.

Handle assignment and copy construction in every class, even if "nobody ever does those things".

Recognize that anyting somebody can do, they will do.

Strive for portable code.

Design your codee so that when changes are necessary, the impact is localized.

Future-tense thinking:

* Provide complete classes, even if some parts aren't currently used.

* Design your interfaces to facilitate common operations and prevent common errors.
Make the classes easy to use correctly, hard to use incorrectly.

* If there is no great penalty for generalizing your code, generalize it.

## Item 33: Make non-leaf classes abstract

```c++
class Animal {
public:
    Animal& operator=(const Animal& rhs);
}

class Lizard : public Animal {
public:
    Lizard& operator=(const Lizard& rhs);
};

class Chicken : public Animal {
public:
    Chicken& operator=(const Chicken& rhs);
};

Lizard liz1;
Lizard liz2;
Animal* pAnimal1 = &liz1;
Animal* pAnimal2 = &liz2;
*pAnimal1 = *pAnimal2;
```

Two problems:

1. The assignment operator invoked on the last line is that of the Animal class, even though the objects involved are of type Lizard.

2. The real programmers write code like this.

One approach to the problem is to make the assignment operators virtual.

```c++
class Animal {
public:
    virtual Animal& operator=(const Animal& rhs);
}

class Lizard : public Animal {
public:
    virtual Lizard& operator=(const Animal& rhs) {
        const Lizard& rhs_liz = dynamic_cast<const Lizard&>(rhs);
        // procees with a normal assignment of rhs_liz to *this
    }
};

class Chicken : public Animal {
public:
    virtual Chicken& operator=(const Animal& rhs);
};
```

The function assigns \*this only if rhs is really a Lizard. If it's not, the function propagates the bad_cast exception that dynamic_cast throws
when the cast fails.

For normal copy, we can add another overload operator=
```c++
class Lizard : public Animal {
public:
    virtual Lizard& operator=(const Animal& rhs) {
        return operator=(dynamic_cast<const Lizard&>(rhs));
    }
    Lizard& operator=(const Lizard& rhs)
};
```

Don't want to use dynamic_cast and Animal can also copy.

Make Animal virtual.

```c++
class AbstractAnimal {
protected:
    AbstractAnimal& operator=(const AbstractAnimal& rhs);
public:
    virtual ~AbstractAnimal() = 0;
};

class Animal: public AbstractAnimal {
public:
    Animal& operator=(const Animal& rhs);
};
class Lizard: public AbstractAnimal {
public:
    Animal& operator=(const Lizard& rhs);
};
```

## Item 34: Understand how to combine C++ and C in the same program

### Name Mangling

C++ compiler will do name mangling, but C compiler won't.

```c++
void drawLine(int x1, int y1, int x2, int y2);
// write this
drawLine(a, b, c, d);  // call to unmangled function name
// your object files contain:
xyzzy(a,b,c,d);  // call to mangled function name
```

If you want to invoke C function in C++ which is unmangled, suppress name mangling.

```c++
// declare a function called drawLine; don't mangle its name
extern "C"
void drawLine(int x1, int y1, int x2, int y2);
```

A function in assembler:
```c++
extern "C" void twiddleBits(unsigned char bits);
```

Let C++ code designed for use outside C++
```c++
// the following C++ function is designed for use outside C++ and should not have its name mangled
extern "C" void simulate(int iterations);
```

A slew of functions whose names you don't want mangled.
```c++
extern "C" {
    void drawLine(...);
    void simulate(...);
}
```

The use of extern "C" simplifies the maintenance of header files that must be used with both C++ and C.

```c++
#ifdef __cplusplus
extern "C"
#endif
void drawLine(...);
void simulate(...);
#ifdef __cplusplus
}
#endif
```

Different compiler mangles in different ways.

### Initialization of Statics

static initialization: some variables are constructed before main.

### Dynamic Memory Allocation

### Data Structure Compatibility

It is safe to pass data structures from C++ to C and from C to C++ provided the definition of those structures compiles in both C++ and C.

Adding non-virtual member functions to the C++ version of a struct that's otherwise compatible with C will probably not affect its compatibility.

### Summary

* Make suer that C++ and C compilers produce compatible object file.

* Declare functions to be used by both languages extern "C".

* If at all possible, write main in C++.

* Always use delete with memory from new; always use free with memory from malloc.

* Limit what you pass between the two languages to data structures that compile under C; 
the C++ versions of structs may contain non-virtual member functions.



