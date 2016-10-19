# Ch1: Basics

## Item 1: Distinguish between pointers and references

First, recognize that there is no such thing as a null reference. A reference must always refer to some objects.

```c++
char* pc = 0;
char& rc = *pc;  // bad!
```

Because a reference must refer to an object, C++ requires that references be initialized.

The fact that there is no such thing as null reference implies that it can be more efficient to use references
than to use pointers. No need to test the validity of a reference before using it.

Another important difference between pointers and references is that pointers may be reassigned to refer to different
objects. A reference, however, always refers to the object with which it is initialized.

Use a pointer when it's possible that there's nothing to refer to.

Use a reference when you know there will always be an object to refer to and you never want to refer to anything else.

References are the feature of choice when you know you have something to refer to, when you'll never want to refer to
anything else and when implementing operators whose syntactic requirements make the use of pointers undesirable. In
all other cases, stick with pointers.

## Item 2: Prefer C++-style casts

C++ style casts are more specify and more easy to find.

static_cast has basically the same power and meaning as the general-purpose C-style cast. (But can not remove const)

cosnt_cast is used to cast away the constness or volatileness.

dynamic_cast is used to perform safe casts down or across an inheritance hierarchy. 
Failed casts are indicated by a null pointer (when casting pointers) or an exception (when casting reference).

dynamic_cast cannot be applied to type lacking virtual functions.

reinterpret_cast is used to perform type conversions whose result is nearly always implementation-defined.
One usage is to cast between function pointer types.

## Item 3: Never treat arrays polymorphically

C++ allows you to manipulate arrays of derived class objects through base class pointers and references. This is no 
feature at all, because it almost never works the way you want it to.

```c++
class BST {...};
class BalancedBST: public BST {...};

void printBSTArray(stream& s, const BST array[], int numElements) {
    for (int i = 0; i < numElements; ++i) {
        s << array[i];
    }
}

// work fine when you pass an array of BST objects
BST BSTArrary[10];
printBSTArray(cout, BSTArrary, 10);

// Bad if you pass derived class objects
BalancedBST bBSTArray[10];
printBSTArray(cout, bBSTArray, 10);
```
array[i] stands for \*(array+i).

In the for loop, the code walks through the array. Each time pointer moved by sizeof(BST). 

The problem is similar if you try to delete an array of derived class objects through a base class pointer.

## Item 4: Avoid gratuitou default constructors

The lacking default constructors may be problematic in three contexts:

1. The creation of arrays.
2. Ineligible for use with many template-based container classes
3. Virtual base classes lacking default constructors are a pain to work with

Some workaround for the first context:

1. Provide the necessary arguments at the point where the array is defined. (Only work for stack)
2. Use array of pointers
3. Use placement new. But be careful wehn deleting them.
