# Item 28: Smart pointers

## Construction, assignment, and destruction of smart pointers

When copied and assigned, auto_ptr will transfer the ownership.

## Implementing the dereferencing operators

## Testing smart pointers for nullness

Implicit conversion to bool.

## Converting smart pointers to dumb pointers

Not good to have implicit conversion.

## Smart pointers and inheritance-based type conversions

Make use of member function template.

```c++
template<typename T>
class SmartPtr {
public:
    template<class newType>
    operator SmartPtr<newType>() {
        return SmartPtr<newType>(pointee);
    }
};
```

## Smart pointers and const
