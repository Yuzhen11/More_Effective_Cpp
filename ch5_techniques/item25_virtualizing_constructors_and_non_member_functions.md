# Item 25: Virtualizing constructors and non-member functions

```c++
class NLComponent {
};
class TextBlock: public NLComponent {
};
class Graphic: public NLComponent {
};

// A newsletter object consists of a list of NLComponent object
class NewsLetter {
public:
    NewsLetter(isteam& str);

    // read the data for the next NLComponent from str
    // create the component and return a poiner to it
    static NLComponent* readComponent(istream& str);
private:
    list<NLComponent*> components;
};

NewsLetter::NewsLetter(istream& str) {
    while(str) {
        components.push_back(readComponent(str));
    }
}
```
`readComponent` creates a new object depending on the data it reads. Because it creates new objects, it acts much like a
constructor, but because it can create different types of objects, we call it a **virtual constructor**.

**virtual copy constructor**
```c++
class NLComponent {
public:
    // declaration of virtual copy constructor
    virual NLComponent* clone() const = 0;
};
class TextBlock: public NLComponent {
public:
    virual TextBlock* clone() const {  // virtual copy constructor
        return new TextBlock(*this);
    }
};
class Graphic: public NLComponent {
public:
    virual Graphic* clone() const {  // virtual copy constructor
        return new Graphic(*this);
    }
};

```
The above implementation takes advantage of a relaxation in the rules for virtual function return types. No longer must a 
derived class's redefinition of a base class's virtual function declare the same return type. Instead, if the function's 
return type is a pointer to a base class, the derived class's function may return a pointer to a class derived from that base class.

Then, the normal copy ctor of NewsLetter just call the virtual copy ctor for each component.
```c++
NewsLetter::NewsLetter(const NewsLetter& rhs) {
    for (auto it = rhs.components.begin(), it != rhs.components.end(); ++ it) {
        components.push_back((*it)->clone());
    }
}
```
**Making Non-Member Functions Act Virtual**

```c++
class NLComponent {
public:
    virtual ostream& print(ostream& s) const = 0;
};
class TextBlock: public NLComponent {
public:
    virtual ostream& print(ostream& s) const;
};
class Graphic: public NLComponent {
public:
    virtual ostream& print(ostream& s) const;
};

inline
ostream& operator<<(ostream& s, const NLComponent& c) {
    return c.print(s);
}
```

Let the non-member function call the virtual function.
