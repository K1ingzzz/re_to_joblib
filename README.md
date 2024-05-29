# re_to_joblib

## the point of sec issue

In my opinion,the focus of discussing security issues is to help programmers comply with security norms and guidelines, not whether it will cause harm or whether it is easy to reach.If all programmers can write code safely, then the security problems we face will be reduced a lot.

## how to fix

As for fix, i think make that class private is necessary, but the main reason is that using pickle to handle python objects without limiting method calls will bring security risks. Why does golang not have a deserialization vulnerability but python does? One of the reasons is that golang only assigns values to objects without calling any methods. However, due to the __reduce__ in python, attacker can control deserialization specification, so this problem cannot be avoided.I know many packages use it this way, but common doesn't mean 100% correct.

Python docs has already provided a safer way to handle python objects by pickle, but maybe no one care. https://docs.python.org/release/3.8.1/library/pickle.html?highlight=pickle#pickle.Unpickler.find_class.

Here is an example

```
import pickle
import builtins
import os

class A:
    def __reduce__(self):
        return (os.system,('whoami',))
    
a=A()
with open('test.pkl','wb') as file:
    pickle.dump(a,file)

class RestrictedUnpickler(pickle.Unpickler):
    def find_class(self, module, name):
        #put the modules and names you need to the whitelist
        safe_classes = {"int", "str", "list"}
        safe_modules = {"builtins"}

        if module in safe_modules and name in safe_classes:
            return getattr(builtins, name)
        
        raise pickle.UnpicklingError(f"Forbidden class: {module}.{name}")

with open('test.pkl','rb') as file:
    unpickler = RestrictedUnpickler(file)
    try:
        obj = unpickler.load()
        print(f"Unpickled object: {obj}")
    except pickle.UnpicklingError as e:
        print(f"Error during unpickling: {e}")
```

It's not 100% safe, whether it is suitable also depends on the different projects themselves, but at least it filters out many common dangerous calls such as 'os.system'.

I hope this help.
