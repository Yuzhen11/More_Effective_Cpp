# Item 31: Making functions virtual with respect to more than one object

Multiple Dispatch

```c++
class GameObject { ... };
class SpaceShip: public GameObject {};
class SpaceStation: public GameObject {};
class Asteroid: public GameObject {};

void checkForCollision(GameObject& object1, object2) {
    if (theyJustCollided(object1, object2)) {
        processCollision(object1, object2);
    }
    else {
        ...
    }
}
```

Ships, stations, asteroids collide with each other in different ways.

What you need is a kind of function whose behavior is somehow virutal on the types of more than one object. 
C++ offers no such function.

## Use Virtual Functions and RTTI

```c++
class GameObject {
public:
    virutal void collide(GameObject& otherObject) = 0;
    ...
};
class SpaceShip: public GameObject {
public:
    virtual void collide(GameObject& otherObject);
};

class CollisionWithUnkonwObject {
public:
    CollisionWithUnkonwObject(GameObject& whatWeHit);
    ...
};

void SpaceShip::collide(GameObject& otherObject) {
    const type_info& objectType = typeid(otherObject);
    if (objectType == typeid(SpaceShip)) {
        SpaceShip& ss = static_cast<SpaceShip&>(otherObject);
        // Process a spaceship-spaceship collision
    }
    else if (objectType == typeid(SpaceStation)) {
        SpaceStation& ss = static_cast<SpaceStation&>(otherObject);
        // Process a spaceship-spacestation collision
    }
    else if (objectType == typeid(Asteroid)) {
        Asteroid& ss = static_cast<Asteroid&>(otherObject);
        // Process a spaceship-Asteroid collision
    }
    else {
        throw CollisionWithUnkonwObject(otherObject);
    }
}
```

If a new type of object - a new class - is added to the game, we must update each RTTI-based if-then-else chain in the program.

## Using Virtual Functions Only

```c++
class GameObject {
public:
    virtual void collide(GameObject& otherObject) = 0;
    virtual void collide(SpaceShip& otherObject) = 0;
    virtual void collide(SpaceStation& otherObject) = 0;
    virtual void collide(Asteroid& otherObject) = 0;
};

class SpaceShip: public GameObject {
public:
    virtual void collide(GameObject& otherObject);
    virtual void collide(SpaceShip& otherObject);
    virtual void collide(SpaceStation& otherObject);
    virtual void collide(Asteroid& otherObject);
};

void SpaceShip::collide(GameObject& otherObject) {
    otherObject.collide(*this);
}

void SpaceShip::collide(SpaceShip& otherObject) {
    // Process a SpaceShip-SpaceShip collision
}
// ...
```

It's a smart way to do two virtual function calls.

Problems:

Each class must know about its siblings.

No if-then-else, but even worse, need to update the definition.

## Emulating Virtual Function Tables

```c++
class GameObject {
public:
    virutal void collide(GameObject& otherObject) = 0;
    ...
};
class SpaceShip: public GameObject {
public:
    virtual void collide(GameObject& otherObject);

    virtual void hitSpaceship(SpaceShip& otherObject);
    virtual void hitSpaceStation(SpaceStation& otherObject);
    virtual void hitAsteroid(Asteroid& otherObject);
private:
    typedef void(SpaceShip::*HitFunctionPtr)(GameObject&);
    typedef map<string, HitFunctionPtr> HitMap;
    static HitFunctionPtr Lookup(const GameObject& whatWeHit) {
        static HitMap collisionMap;

        // loop up in the map
        HitMap::iterator mapEntry = collisionMap.find(typeid(whatWeHit).name());

        if (mapEntry == collisionMap.end()) return 0;

        return (*mapEntry).second;
    }
};

void SpaceShip::collide(GameObject& otherObject) {
    HitFunctionPtr hfp = lookup(otherObject);
    if (hfp) {
        this->*hfp(otherObject);  // call it
    }
    else {
        throw CollisionWithUnkonwObject(otherObject);
    }
}
```

This method actually still use RTTI (typeid). Compared with the first method, this method eliminates the if-then-else
parts.

## Initializing Emulated Virtual Function Tables

Previously the hit functions are like these, with different signatures.

```c++
class SpaceShip: public GameObject {
    ...
    virtual void hitSpaceship(SpaceShip& otherObject);
    virtual void hitSpaceStation(SpaceStation& otherObject);
    virtual void hitAsteroid(Asteroid& otherObject);
    ...
};
```

One way is to use reinterpret_cast, which is a choice when converting between function pointer types.

A better way is to make these functions take GameObject& parameter and use dynamic_cast inside.

```c++
class SpaceShip: public GameObject {
    ...
    virtual void hitSpaceship(GameObject& spaceShip);
    virtual void hitSpaceStation(GameObject& spaceStation);
    virtual void hitAsteroid(GameObject& asteroid);
    ...
};

void SpaceShip::hitSpaceShip(GameObject& spaceShip) {
    SpaceShip& otherShip = dynamic_cast<SpaceShip&>(spaceShip);
}
...
```

## Using Non-Member Collision-Processing Functions

```c++
namespace {  // unnamed namespace

void shipAsteroid(GameObject& spaceShip, GameObject& asteroid);
void shipStation(GameObject& spaceShip, GameObject& spaceStation);
void asteroidStation(GameObject& asteroid, GameObject& spaceStation);

typedef void (*HitFunctionPtr)(GameObject&, GameObject&);
typedef map<pair<string, string>, HitFunctionPtr> HitMap;
HitMap* initilizeCollisionMap();
HitFunctionPtr lookup(const string& class1, const string& class2);
}
```

## Inheritance and Emulated Virtual Function Tables

## Initializing Emulated Virtual Function Tables

Use the creation of global objects to insert the entries into the map (local static variable).


