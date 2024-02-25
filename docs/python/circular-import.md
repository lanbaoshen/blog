# Circular Import

In this document I will introduce how circular import commonly arise and what are the main ways to fix them.

## Modules Dependency

### Cause

```python title="main.py"
import module_a
```

```python title="module_a.py"
from module_b import func_b


def func_a():
    func_b()
```

```python title="module_b.py"
from module_a import func_a


def func_b():
    func_a()
```

At runtime, `main` tried to import something from `module_a`, so it start to initialize `module_a`. 
It starts to running the code from `module_a`, but the first line in `module_a` is to import something from `module_b`, 
so it stops initializing `module_a`, because `module_b` has to be initialized first.

And then, it hops over to `module_b` and starts to running that code instead. 
But then the first line in `module_b` is that it needs something from `module_a`, 
so it stops running `module_b` and goes over to `module_a` and it would like to just start initializing `module_a`.

However, it realizes that `module_a` is already in the process of initialization, so that's where the error occurs 

### Solution

We don't actually use `func_a` and `func_b` at import time, there is no true cyclic dependency.

If `func_a` is the only place that you need `func_b`, you can take the import and put it inside `func_a`:

```python hl_lines="2" title="module_a.py"
def func_a():
    from module_b import func_b
    func_b()
```

This could actually be even more efficient, because if you never call `func_a`, 
then it may be that you never even have to import `module_b` at all.

But if you have a batch of functions in `module_a` and a lot of different ones use the `func_b`. 
In that case, the better solution would be to just import the `module_b` directly:

```python hl_lines="1 5" title="module_a.py"
import module_b


def func_a():
    module_b.func_b()
```

```python hl_lines="1 5" title="module_b.py"
import module_a


def func_b():
    module_a.func_a()
```

First, `main` tries to import `module_a` cause `module_a` to start running. 
In `module_a` we get to the import `module_b` which triggers `module_b` to start running.

In `module_b` when we get to the import `module_a`, because the `module_a` has already started initializing, 
this module object technically exists. 

Since it exists, it doesn't start running this again. 
So the `module_b` will just continue on finished it import and then after that import is done, 
the `module_a` will finish importing.

## Type Hint

### Cause

```python title="main.py"
import module_a
```

```python title="module_a.py"
from module_b import B


class A:
    def __init__(self, b: B):
        self.b = b
```

```python title="module_b.py"
from module_a import A


class B:
    def __init__(self, a: A):
        self.a = a
```

Type annotations are defined on class definition, when the class is defined, not when it is run.

### Solution

If all we care about is static analysis, then we don't even need to do the import at runtime:

```python hl_lines="1-5" title="module_a.py"
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from module_b import B


class A:
    def __init__(self, b: B):
        self.b = b
```

```python hl_lines="1-5" title="module_b.py"
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from module_a import A


class B:
    def __init__(self, a: A):
        self.a = a
```

If you put all the import that you just need for the purpose of type-checking, 
then none of those will actually happen at runtime, completely avoiding the import loop.

The `from __future__ import annotations` changes the way that annotation work, 
instead of evaluating `A` and `B` as a name, all annotations are converted to string.
Even though `A` and `B` are not imported, the string `A` and `B` certainly exist.

## Subpackage Init Cycle  

### Cause

```python title="main.py"
from pkg.subpkg_b.module_b import B
```

```python title="pkg/__init__.py"
```

```python title="pkg/subpkg_a/__init__.py"
from .module_a import A
from .module_c import C
```

```python title="pkg/subpkg_a/module_a.py"
from ..subpkg_b.module_b import B


class A:
    ...
```

```python title="pkg/subpkg_a/module_c.py"
class C:
    ...
```

```python title="pkg/subpkg_b/__init__.py"
from .module_b import B
```

```python title="pkg/subpkg_b/module_b.py"
from ..subpkg_a.module_c import C


class B:
    ...
```

Our subpackage init files here are re-exporting some names in this case classes from some of the modules inside of them.
The problem is that every module in a subpackage depends on the init of the subpackage. 

So if the init of the subpackage depends on all the modules in the subpackage, 
then we have kind of made a fake dependency of every module in the subpackage on every other module in the subpackage.

That means even though `module_c` has no import at all, `module_c` now automatically also depends on `module_a` because it depends on the init.

`main` tries to import something from `module_b`, we first have to run the subpackage init, all `subpkg_b` does is import from `module_b`, so we start to import `module_b`.

`module_b` wants to import something from `module_c`, so we first have to initialize `subpkg_a`. 
But `subpkg_a` import from `module_a`, and `module_a` import from `module_b`, so now we have a cycle, 
because we haven't finished initializing `module_b`.

### Solution

The way to get rid of the cycle is to just make all of your init files blank.

If you still want to give your users a short way to import names, you can define an interface package:

```python title="pkg/interface_a/__init__.py"
from ..subpkg_a.module_a import A
from ..subpkg_a.module_c import C
from ..subpkg_b.module_b import B
```

```python title="main.py"
from pkg.interface_a import A, B, C
```
