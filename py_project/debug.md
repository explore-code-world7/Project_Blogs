# print recursive callback stack
```python
import traceback

def foo():
    print("Call stack:")
    print("".join(traceback.format_stack()))

def bar():
    foo()

bar()
````
