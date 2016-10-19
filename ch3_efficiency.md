# Ch3: Efficiency

## Item 24: Understand the costs of virtual functions, multiple inheritance, virtual base classes, and RTTI

virtual tables and virtual table pointers.

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
