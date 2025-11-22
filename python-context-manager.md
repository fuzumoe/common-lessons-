# 🟦 Python Context Managers  

Python **context managers** let you automatically run setup and cleanup
code using the `with` statement.

You already use them without realizing it:

``` python
with open("file.txt") as f:
    data = f.read()
```

------------------------------------------------------------------------

## 1. What the `with` Statement Really Does

Writing:

``` python
with something as value:
    ...
```

Internally becomes:

``` python
mgr = something
value = mgr.__enter__()

try:
    ...
except Exception as e:
    handled = mgr.__exit__(type(e), e, e.__traceback__)
    if not handled:
        raise
else:
    mgr.__exit__(None, None, None)
```

### ✔️ `__enter__()`

Runs at the start of the block and returns the object after `as`.

### ✔️ `__exit__(exc_type, exc_value, exc_tb)`

Runs at the end, even if there is an exception.

-   Return **True** → suppress exception\
-   Return **False / None** → re-raise exception

------------------------------------------------------------------------

## 2. Creating Custom Context Managers (Class-Based)

### 2.1 Simple Logging Context Manager

``` python
class LoggingContext:
    def __enter__(self):
        print("[ENTER] Starting block")
        return "You can use this value inside 'with'"

    def __exit__(self, exc_type, exc_value, exc_tb):
        if exc_type is None:
            print("[EXIT] Block finished successfully")
        else:
            print(f"[EXIT] Block raised: {exc_type.__name__}: {exc_value}")
        return False
```

------------------------------------------------------------------------

### 2.2 File Manager (Custom Version of `open`)

``` python
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None

    def __enter__(self):
        print(f"Opening file: {self.filename}")
        self.file = open(self.filename, self.mode, encoding="utf-8")
        return self.file

    def __exit__(self, exc_type, exc_value, exc_tb):
        print(f"Closing file: {self.filename}")
        if self.file and not self.file.closed:
            self.file.close()
        return False
```

------------------------------------------------------------------------

### 2.3 Temporary Directory Change

``` python
import os

class ChangeDirectory:
    def __init__(self, new_path):
        self.new_path = new_path
        self.old_path = None

    def __enter__(self):
        self.old_path = os.getcwd()
        os.chdir(self.new_path)
        print(f"Changed directory to {self.new_path}")
        return self.new_path

    def __exit__(self, exc_type, exc_value, exc_tb):
        os.chdir(self.old_path)
        print(f"Reverted directory to {self.old_path}")
        return False
```

------------------------------------------------------------------------

### 2.4 List Transaction (Rollback on Error)

``` python
class ListTransaction:
    def __init__(self, target_list):
        self.target_list = target_list
        self._backup = None

    def __enter__(self):
        self._backup = self.target_list.copy()
        return self.target_list

    def __exit__(self, exc_type, exc_value, exc_tb):
        if exc_type is not None:
            print("Error occurred, rolling back list changes")
            self.target_list.clear()
            self.target_list.extend(self._backup)
        else:
            print("No error, keeping list changes")
        return False
```

------------------------------------------------------------------------

### 2.5 Suppress Specific Exceptions

``` python
class Suppress:
    def __init__(self, *exception_types):
        self.exception_types = exception_types

    def __enter__(self):
        return None

    def __exit__(self, exc_type, exc_value, exc_tb):
        if exc_type is None:
            return False
        if issubclass(exc_type, self.exception_types):
            print(f"Suppressed {exc_type.__name__}: {exc_value}")
            return True
        return False
```

------------------------------------------------------------------------

## 3. Function-Based Context Managers (`contextlib`)

### 3.1 Basic Pattern

``` python
from contextlib import contextmanager

@contextmanager
def my_context():
    print("Entering")
    try:
        yield "Value inside with"
    finally:
        print("Exiting")
```

------------------------------------------------------------------------

### 3.2 Timer Context Manager

``` python
import time
from contextlib import contextmanager

@contextmanager
def timer(label="Block"):
    start = time.time()
    print(f"[{label}] Started")
    try:
        yield
    finally:
        end = time.time()
        print(f"[{label}] Finished in {end - start:.4f} seconds")
```

------------------------------------------------------------------------

### 3.3 Temporary Environment Variable

``` python
import os
from contextlib import contextmanager

@contextmanager
def temp_env_var(key, value):
    old_value = os.environ.get(key)
    os.environ[key] = value
    try:
        yield
    finally:
        if old_value is None:
            del os.environ[key]
        else:
            os.environ[key] = old_value
```

------------------------------------------------------------------------

## 4. Multiple and Nested Context Managers

### Multiple managers in one `with`:

``` python
with open("a.txt") as f1, open("b.txt") as f2:
    ...
```

### Nested:

``` python
with ChangeDirectory("/tmp"):
    with timer("inner"):
        ...
```

------------------------------------------------------------------------

## 5. How to Design a Context Manager

1.  Identify the resource/state.\
2.  Define setup (`__enter__` or code before `yield`).\
3.  Define cleanup (`__exit__` or code after `yield`).\
4.  Decide whether to suppress exceptions.

------------------------------------------------------------------------

## 6. Practice Ideas

-   Lock context manager\
-   Temporary logging level\
-   Database transaction\
-   Temporary file cleanup\
-   Mock timer returning elapsed milliseconds
