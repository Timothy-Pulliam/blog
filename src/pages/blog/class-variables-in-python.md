---
layout: "../../layouts/BlogPost.astro"
title: "Class Variables in Python"
description: "Class Variables in Python"
pubDate: "Feb 3 2024"
heroImage: "/python-hero.jpg"
previewText: "A good guiding rule is, do not use class variables unless absolutely necessary. The reason is simple; they break the most basic principle of Object
Oriented Programming, which is that data should be encapsulated."
---

Consider the following python class

```python
class Cat(object):

    # class / static variables
    is_mammal = True
    genus = "felis"

    def __init__(self, name="daisy"):
        # instance variables
        self.set_name(name)

    # set / get methods
    def set_name(self, name):
        self.name = str(name)

    def get_name(self):
        return self.name
```

Class variables (also known as Static Variables) can be used without needing to instance the class

```python
print(Cat.is_mammal)
# True
```

## Why Class Variables are Bad

A good guiding rule is, do not use class variables unless absolutely necessary. The reason is simple; they break the most basic principle of Object
Oriented Programming, which is that data should be encapsulated. Other instances of a class should not have direct access to other instance's data members (variables). That would defeat the very purpose of creating separate instances with their own values to begin with.

You can change a class' static variable like so

```python
Cat.is_mammal = "yes"
print(Cat.is_mammal)
# yes
```

However, note that if you do, you will change the value for any existing instances, as well as future instances of the class.

```python
daisy = Cat()
print(daisy.is_mammal)
# True
Cat.is_mammal = "yes"
sophie = Cat()
print(daisy.is_mammal)
# yes
print(sophie.is_mammal)
# yes
```

## Only Use Class Variables For Constants

Class variables exist at the class scope. All instances of the class have access to these variables. In this regard they are similar to global variables and should only be used for storing global values that all instances of a class agree on (i.e. constants).

In the cat class example, having a reference to the genus of the cat is probably fine as a class variable, since that is constant and should never change. However, you should designate these values as constants by typing them in upper case.

The fact that class variables are similar to global variables should be enough of a deterrent to use them with extreme caution. Any variable that can be changed or read by many instances at any time creates a fundamentally unpredictable program.
