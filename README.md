# classes.lua

## What?

classes provides a simple solution to Object-Oriented Programming in Lua. It has the basic OOP features one might expect, such as inheritance, and covers up dealing with metatables and the like.  There are other great solutions out there, including ones that don't use metatables and ones that don't use libraries at all!  I wrote this mostly for fun, quite a while ago.

## Why?

classes was developed because when I started working with Corona, I had only a basic knowledge of Lua. Working on big projects, in any language, can usually benefit from some OOP functionality. There are a few libraries out there that bring this to Lua. This is a simple, lightweight, ~100 line implementation.

## How?

Import it like any other Lua module:

```Lua
local classes = require "classes"
```

### Creating a Class

Creating a Class is pretty straightforward. Heres why:

```Lua
-- The function classes.class() returns a new Class object.
-- In this case, our class is stored in the variable 'Animal'.
local Animal = classes.class()
```

Alternatively, you may want to create a class that has a Superclass. That's easy as well:

```Lua
-- Cat is now a Subclass of Animal.
local Cat = classes.class(Animal)
```

If no Superclass is provided to the function, __classes.Object__ is used as the Superclass. This basic Class has some predefined methods, but we will get to those later.

### Instance Methods

If you have ever looked into Corona's event system, which is inevitable, then you should be familiar with table listeners. classes uses the underlying concept of these for instance methods. When an instance method gets called, it automatically has access to the self variable - the variable for that specific instance of the Class.

Here is an example of creating a instance method:

```Lua
--- This is an instance method specific to the Cat class.
-- ':', not '.'!
function Cat:meow ()
	print("meow "..self.name.." meow")
end
```

### Class Methods

If you come from a language like Java or C#, a class method is simply a static method. These methods do not have access to the self variable, as they apply to the whole class, and not a specific instance.

The syntax is similar to an instance method, with a minor difference. A '.' is used instead of accessing the method via a ':'. This is because the method does not need to know what self is, it represents the whole class!

```Lua
--- This is an example of a Class method.  We can't access 'self', because it represents the whole Class, not an instance of this Class.
-- '.' not ':'!
--
-- @return Returns the total number of Animals.
function Animal.totalAnimals ()
	return numAnimals
end
```

### Constructors

Constructors are a special method that creates a new instance of a Class. In classes, that method is known to the Class as init.

```Lua
--- Constructor.  In addition to a name, it also takes in a cat breed.
--
-- @param breed The breed of the cat.
function Cat:init(name, breed)
	-- Make sure to call the super constructor!
	self.super:init(name)
	-- Next, do some custom instantiation.
	self.breed = breed
end
```

Notice the init method is being called on some self.super variable. This is just calling the constructor on the Class' Superclass. It is generally best practise, and sometimes required, to call the Superclass' constructor as the first thing you do in your constructor. More on the self and super variables later.

Constructors are optional. If you don't supply your own constructor, it will try to call the Superclass' constructor, and so on, until it finds one.

### Creating a Class Instance

To create an instance of a Class, simply utilize the new method on the class object. Notice how the new method is a class method and not instance method. This means it should be accessed via '.' and not a ':'.

```Lua
-- Create an instance of the Class Cat.  Pass the constructor a string called "mio".
local myCat = Cat.new("mio")
```

But this isn't the init function! Don't worry, this function calls your init function that you created after it takes care of some stuff. It passes the parameters you passed into the new function, too.

### Instance variables

Inside any instance method, instance variables can be set by declaring them in the self variable. These variables belong to the object, not their defining class.

This is a good time to explain the self variable. If you are not familiar with it, it refers to the actual instance of the class you are currently operating on. For instance, if you had two instances of the Cat class: myCat and yourCat, the self variable refers to the respective cat object when you invoke `myCat:meow()` or `yourCat:meow()`. When you call `myCat:meow()`, you can check that self equals myCat. When you call `yourCat:meow()`, self is equal to yourCat. This variable is only visible inside an instance method, not class methods or outside methods.

The super variable is similar to the self variable, except it refers to the Superclass of the current object you are operating on. You must access this method through the self variable. For instance, lets say we defined an init method in the Animal class.

```Lua
function Animal:init(name)
    self.name = name
end
 
-- ...
 
function Cat:init(name, breed)
    self.super:init(name)
    -- ...
end
```

Here, we first call our Superclass' constructor. Because Animal happens to the the Superclass, this takes the name argument we received and sets it as an instance variable. It is important to note that self refers to the same object when it is used in a super method.

### Class Variables
These, a Java developer would call them (maybe) are static variables. They belong to the class, not an instance of that class.

Here is an example for counting the number of objects you create of a certain type of Class.

```Lua
Animal.numAnimals = 0
 
Animal:init(name)
    self.name = name
    Animal.numAnimals = Animals.numAnimals + 1
    self.id = Animal.numAnimals
end
 
--- This method is available to the Animal class, and all Subclasses of Animal as well.
--
-- @return Returns the id of this Animal.
function Animal:getAnimalId ()
        return "Animal<"..self.id..">"
end
```

You access these variables via the name of the class, as seen above. These variables are public, anyone who has access to the class, has access to that variable.

To create a private static variable, use the local keyword and drop the class name as such:

```Lua
local numAnimals = 0
 
Animal:init(name)
    self.name = name
    numAnimals = numAnimals + 1
    self.id = numAnimals
end
```

*NOTE:* These variables can be accessed by any code in the same file as the class declaration. So if you want *true* private static variables, put all of the class definition in a separate file from anything else!

### Instance Type Checking

To check if an object is an instance of, or directly or indirectly a subclass a Class, use the __instanceOf__ method:

```Lua
-- Returns true,  a Cat is also an Animal.
myCat:instanceOf(Animal)
 
-- Returns false, an Animal is not also a Cat.
myAnimal:instanceOf(Cat)
```

Notice how we pass the Class object, not an instance of the object, to the method.

Anything else I should know?
Yea. Avoid using these method names in a class:

__new__, __alloc__

They are reserved for use in the classes library for instantiating objects.
Also, I wouldn't try changing these variables of an object/class value:

__super__, __class__

As you know super is a reference to the supertype, i.e. if mio is an instance of Cat, then `mio.super` is an instance of Animal, which gives the object the ability to call super methods, as seen before.
The class variable is a reference to the class object, i.e. `mio.class == Cat`
