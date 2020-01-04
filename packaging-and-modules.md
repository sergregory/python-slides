---
title: Python modules & dependencies system
tags: import, pip, poetry, module
description: a short intro into how to use python modules properly.
slideOptions:
    theme: white


---

# Python modules system

<!-- Put the link to this slide here so people can follow -->
slide: https://hackmd.io/W_rKVn7yQaiLSkZi0BKX3Q

---


## Rules & Restrictions

- Python 2 is dead
- All the information may be incomplete
- Documentation is the source of truth if you're in doubt:
    - [modules](https://docs.python.org/3/tutorial/modules.html?highlight=modules)
    - [import system](https://docs.python.org/3/reference/import.html#importsystem)
- If you have time, take a look at [David Beazley's extensive tutorial](http://www.dabeaz.com/modulepackage/) on this topic
    - I've taken a lot from that slides

---

## Basic "gold" path

----

### Modules

- Any Python source file is a module
```python3
#spam.py

def grok(x):
    ...
    
def blah(x):
    ...
```

- You use import to execute and access it

```python3
import spam

a = spam.grok('hello')

from spam import grok

a = grok('hello')
```

----

### Namespaces

- Each module is its own isolated world

```python3 
#spam.py
x = 42

def blah():
    print(x)
    
    
# eggs.py
x = 37

def foo():
    print(x)
```

- What happens in a module, stays in a module

----

### Global Variables

- Global variables bind inside the same module
```python3
# spam.py

x = 42
def blah():
    print(x)
```

- Functions record their definition environment

```
>>> from spam import blah
>>> blah.__module__'spam'
>>> blah.__globals__{ 'x': 42, ... }
```

----

### Module Execution
- When a module is imported, all of the statements in the module execute one after another until the end of the file is reached
- The contents of the module namespace are all of the global names that are still defined at the end of the execution process
- If there are scripting statements that carry out tasks in the global scope (printing, creating files, etc.), you will see them run on import
    - *This it why I insist on wrapping code into functions/classes*

----

### from module import
- Lifts selected symbols out of a module after importing it and makes them available locally
```python3
from math import sin, cos

def rectangular(r, theta):
    x = r * cos(theta)
    y = r * sin(theta)
    return x, y
```

- Allows parts of a module to be used without having to type the module prefix

----

### from module import *

- Takes all symbols from a module and places them into local scope

```python3
from math import *

def rectangular(r, theta):
    x = r * cos(theta)
    y = r * sin(theta)
    return x, y
```

- Sometimes useful
- Usually considered bad style (try to avoid)

----

### However, 

- variations on import do not change the way that modules work
```python3 
import math as m
from math import cos, sin
from math import *
...
```
- import always executes the entire file
- modules are still isolated environments
- these variations are just manipulating names

----

### Module Names
- File names have to follow the rules
    - Yes: `good.py`
    - No: `2bad.py`
- Must be a valid identifier name
- Also: avoid non-ASCII characters

----

### Naming Conventions
- It is standard practice for package and module names to be concise and lowercase
    - Yes: `foo.py`
    - No: `FooModule.py`
- Use a leading underscore for modules that are meant to be private or internal
    - `_private_foo.py`
- Don't use names that match common standard library modules (confusing)
    - `projectname/math.py`

----

### Module Search Path

- If a file isn't on the path, it won't import:
```
>>> import sys
>>> sys.path
['', '/usr/lib/python36.zip', '/usr/lib/python3.6',
 '/usr/lib/python3.6/lib-dynload',
 '/home/grigory/.local/lib/python3.6/site-packages',
 '/usr/local/lib/python3.6/dist-packages',
 '/usr/lib/python3/dist-packages']
```
- Sometimes you might hack it
```python3
import sys
sys.path.append("/project/foo/myfiles")
```
... although doing so feels "dirty"

----

### Module Cache
- Modules only get loaded once
```
>>> import spam
>>> import sys
>>> 'spam' in sys.modules
    True
>>> sys.modules['spam']
    <module 'spam' from 'spam.py'>
```
- There's a cache behind the scenes
- Consequence:  If you make a change to the source and repeat the import, nothing happens

----

### Module Reloading
- You can force-reload a module, but you're never supposed to do it
```
>>> from importlib import reload
>>> reload(spam)
    <module 'spam' from 'spam.py'>
```
- Modules reloading is dangerous! Lots of things can go wrong.

----

### Modules Reloading for Notebooks

- There is one special case when modules reloading is very useful and almost unescapable: coding in jupyter notebooks

```
% load_ext autoreload
% autoreload 2
```

- These two commands at the begging of the notebook will auto-reload your external modules on any change in them
    - Please read the documentation on autoreload module first!

----

### `__main__` check
- If a file might run as a main program, do this:
```python3
# spam.py

...

if __name__ == '__main__':    # Running as the main program
    ...
```
- Such code won't run on library import

```python3 
import spam  # Main code doesn't execute
```
```bash
python spam.py   # Main code executes
```

----

### Packages
- For larger collections of code, it is usually desirable to organize modules into a hierarchy

```
.
├── bar
│   └── grok.py
└── spam
    └── foo.py
```

- To do it, you just add `__init__.py` files
```
.
├── bar
│   ├── grok.py
│   └── __init__.py
└── spam
    ├── foo.py
    └── __init__.py
```

----

### Using a Package

- Import works the same way, multiple levels
```python3
import spam.foo
from spam.bar import grok
```
- The `__init__.py` files import at each level
- Apparently you can do things in those files
- These things should relate to the proper importing
- Placing logic in `__init__.py` files is considered as a bad practice.

----

### Summary

- It looks that there is not that much going on in the modules & packages, and in the import system
- Why it is considered to be complex?
- Let's continue beyound the "gold" path 

---

## What are the PYTHONPATH, sys.path and other guys?

----

### Populating path for imports: what we already know

- As I've said, if something is not in path, it cannot be imported.
- We may take a look a the path:
```python3
import sys
print(sys.path)
```
- We can manipulate the path in a hacky way:
```python3
import sys
sys.path.append('some_path')
```

----

### Anatomy of the `sys.path`

`sys.path` includes:
- minimal set from your python distribution
- virtual environment: `site-packages`
- no virtual environment: `site-packages` / `dist-packages`
- current working directory (directory of the script)
- directories from the `PYTHONPATH`

----

### Populating `sys.path`: the right way

1. (it does not work): import only from the current directory

1. add everything you need to the `PYTHONPATH`
    ```
    PYTHONPATH=`pwd` python scripts/script.py
    ```
1. use the `site` package and populate the path dynamically
1. write proper setup scripts and do the developer installs for the local use (theme for the next discussion)

---

## Imports done right

----

### Absolute imports

- Assuming the following structure
    ```
    .
    └── foo
        ├── grok.py
        ├── __init__.py
        └── spam.py

    ```
- Absolute import:
    ```python3
    # foo/spam.py
    from foo import grok
    ```
    - you should have parent dir for `foo` in path

----

### Relative imports

- Absolute import looks strange - `grok` is on the same level, maybe we can import it without absolute path? - Yes!
- Relative import:
    ```python3
    # foo/spam.py
    from . import grok
    ```
- Try to run it - and it will fail with the message:
    ```
    File "foo/grok.py", line 3, in <module>
    from . import spam
    ValueError: Attempted relative import in non-package

----

### How to avoid non-package import errors

```
.
├── bar.py
└── foo
    ├── grok.py
    ├── __init__.py
    └── spam.py
```

```python3
# bar.py
from foo import grok
```

```
python bar.py
```

- everything works smoothly
- but how to use relative imports without these restrictions?

----

### How to avoid non-package import errors 2

- Python assumes we have scripts on the top level of the project - so, we do not need relative imports
- But if we write script somewhere inside the project - we want to have relative imports
- Solution:
    ```
    python -m spam.grok
    ```

----

### Why and how it works?

- Python uses import machinery to locate the module and load its code
- Current directory is added to the `sys.path`
- We still may need to change `sys.path` if we have absolute imports that do not use current directory as root dir
- Note that the process of locating the module to be executed may require importing the containing package.



---

## Glossary for organizing python code

----

### Module 
- An object that serves as an organizational unit of Python code. Modules have a namespace containing arbitrary Python objects.
- Examples: a single `.py` file, a directory with python files / subdirectories

----

### Package

- A Python module which can contain submodules or recursively, subpackages. Technically, a package is a Python module with an `__path__` attribute.

----

### Regular package
- A traditional package, such as a directory containing an `__init__.py` file.

----

### Namespace package
- A PEP 420 package which serves only as a container for subpackages. Namespace packages may have no physical representation, and specifically are not like a regular package because they have no `__init__.py` file.

----

### Wait, what?!

- You mean that if I miss that stupid `__init__.py` file I'll get another type of package?
- Yes! And these two types of packages work differently!

---

## What is namespace package and how to use it

----

### Imports with namespace package

Image you have the following project structure:
```
├── bar
│   └── spam
│       └── bar.py
└── foo
    └── spam
        └── baz.py
```

Please note: there are two sub-directories with the same names (`spam`) and there are no `__init__.py` files

----

### Regular imports

If we do just regular import, everything works as expected:
```
>>> from bar.spam import bar
    Imported bar.py
    
>>> from foo.spam import baz
    Imported baz.py
```

----

### Imports using namespace packages

- So why are there so much fuss about namespace packages?
- Because it can do magic!

```
In [1]: import sys                                                         
In [2]: sys.path.extend(('foo', 'bar'))
In [3]: from spam import bar
Imported bar.py
In [4]: from spam import baz
Imported baz.py
In [6]: import spam
In [7]: spam
Out[7]: <module 'spam' (namespace)>
In [8]: spam.__path__
Out[8]: _NamespacePath(['foo/spam', 'bar/spam'])
```

----

### What documentation says

Namespace packages and regular packages are very similar. The differences are:
- Portions of namespace packages need not all come from the same directory <...> Regular packages are self-contained: all parts live in the same directory hierarchy.
- Namespace packages' `__path__` attribute is a read-only iterable of strings, which is automatically updated when the parent path is modified.

---- 

### How to use namespace packages?

- If you want to combine modules from different parts of your project into the same package
    - E.g. you have plugins system
    - Or just complex project structure (see catalyst for example)

---