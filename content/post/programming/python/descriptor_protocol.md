---
title: Python Descriptor Protocol Inside-out 
date: 2020-03-22T13:29:17-08:00
authors: [Tony Zhang]
keywords: ["python", "descriptor"]
tags: ["programming", "python"]
image: /post/programming/python/descriptor/title.jpg
draft: false
emoji: true
---
<!-- TOC -->

- [Introduction](#introduction)
- [What is Descriptor](#what-is-descriptor)
    - [Descriptor Protocol](#descriptor-protocol)
    - [Attribute Lookup Chain](#attribute-lookup-chain)
- [Applications](#applications)
    - [Lazy Property](#lazy-property)
- [define lazy_property class](#define-lazy_property-class)
- [Descriptors in Python Internals](#descriptors-in-python-internals)
    - [Property](#property)
    - [Function & Method](#function--method)
    - [Class Method](#class-method)
    - [Static Method](#static-method)
- [Reference](#reference)

<!-- /TOC -->
# Introduction
Descriptor protocol is a important concept in Python, which is the basis of many Python internal implementation, such as properties, functions, class methods and static methods. This tutorial will elaborate descriptor:
1. **What is Descriptor** will introduce descriptor protocol, and how Python attribute lookup chain works. 
2. **Application** will discuss one use case (lazy property) of descriptor protocol. 
3. **Descriptors in Python Internals** shows how CPython leverage descriptor protocol to implement properties, functions, class & static methods. 

# What is Descriptor
First of all, let's define Descriptor. In short, a descriptor is an object meets following conditions: 
1. The object's class implements [Descriptor Protocol](#descriptor-protocol). 
2. The object is used as an attribute of another class. 

## Descriptor Protocol
When we said one class implements descriptor protocol, it means the class implements at least one of below methods. 
{{< highlight python "linenos=inline" >}}
__get__(self, instance, owner=None) -> value
__set__(self, instance, value) -> None
__delete__(self, instance) -> None
{{< / highlight >}}
> :information:: From Python 3.6, there is another method `__set_name__(self, owner, name)` added into descriptor protocol. 

In terms of which methods are implemented by class, descriptors are divided into below 2 categories: 
* When a class defines `__get__()`, and one or both `__set__()` and  `__delete()`, we call it **data descriptor**
* When a class only defines `__get__()`, we call it **non-data descriptor**.

The difference between data descriptor and non-data descriptor is important, because they are accessed in different priorities during attribute lookup. 

`__get__()` has three parameters: 
* `self`: descriptor object.
* `instance`: object which descriptor belongs to.
* `owner`: the class of instance. 

Parameters in `__set__()` and  `__delete()` are similar. You may notice that only `__get__()` has `owner` parameter. Why? Because `__get__()` could be accessed as either class attribute or instance/object attribute. Data descriptor, such as Python property, is normally accessed as instance/object attribute; while non-data descriptor, such as Python class method and static method, could be accessed from class directly. 

Below code creates a descriptor, and prints each parameter in `__get__()`:

{{< highlight python "linenos=inline" >}}
class Descriptor:
    def __init__(self):
        self.__value = 0
        
    def __get__(self, instance, owner=None):
        print("self: {}".format(self))
        print("instance: {}".format(instance))
        print("owner: {}".format(owner))
        return 1
        
class MyTest:
    # d is an object of Descriptor
    # d is defined as an attribute of MyTest class
    d = Descriptor()
    
print("======== Class Access ========")
print("MyTest.d: {}".format(MyTest.d))

print("======== Instance Access ========")
t = MyTest()
print("t.d: {}".format(t.d))
{{< / highlight >}}

Output: 
```
======== Class Access ========
self: <__main__.Descriptor object at 0x7f1cd213f0b8>
instance: None
owner: <class '__main__.MyTest'>
MyTest.d: 1
======== Instance Access ========
self: <__main__.Descriptor object at 0x7f1cd213f0b8>
instance: <__main__.MyTest object at 0x7f1cd213f710>
owner: <class '__main__.MyTest'>
t.d: 1
```
As you can see, when descriptor is accessed from class (`MyTest.d`), `instance` parameter is set to `None`. When descriptor is accessed through instance/object (`t.d`), `instance` is set to the object (`t` in our example). In both cases, `owner` is set to instance/object's class (`MyTest`). 

Below is another example code to show differences in data descriptors and non-data descriptors.
{{< highlight python "linenos=inline" >}}
class DataDescriptor:
    def __init__(self):
        self.__value = 0
        
    def __get__(self, instance, owner=None): 
        print("DataDescriptor __get__()")
        return self.__value

    # data descriptor need to define __set__ (or __delete__)    
    def __set__(self, instance, value):
        print("DataDescriptor __set__()")
        self.__value = value
        
class NonDataDescriptor():
    def __get__(self, instance, owner=None):
        print("NonDataDescriptor __get__()")
        return "NonDataDescriptor Value"
        
class MyTest:
    dd = DataDescriptor()
    ndd = NonDataDescriptor()
    
obj = MyTest()
print(obj.dd)
obj.dd = 5
print(obj.dd)
print(obj.ndd)
{{< / highlight >}}

The output is: 
```
DataDescriptor __get__()
0
DataDescriptor __set__()
DataDescriptor __get__()
5
NonDataDescriptor __get__()
NonDataDescriptor Value
```
## Attribute Lookup Chain
All of the magic in descriptor comes from attribute lookup chain. Every time you access an attribute, Python will follow a chain to find the attribute. In general, Python will look up below places in order. 
1. **Data Descriptor** - check whether attribute is a data description by following instance's [MRO](https://www.python.org/download/releases/2.3/mro/).
2. **Instance Attribute** - check whether attribute exists in instance's `__dict__`. 
3. **Non-Data Descriptor & Class Attribute** - check whether attribute is a non-data descriptor or exists in class's `__dict__`. 
4. `__getattr__()` method - if class implements `__getattr__()`, its result will be returned. 

Below is flow chart (got from [[2]](https://www.google.com/books/edition/Python_Descriptors/n6lxDwAAQBAJ?hl=en&gbpv=0)) summarizing above process. 
![Object Attribute Lookup Chain](/post/programming/python/descriptor/obj_attr_lookup_chain.png "Object Attribute Lookup Chain")

As you can see, data descriptor and non-data descriptor have different priorities, we will leverage this property to implement Lazy Property in next section.

# Applications
## Lazy Property
Lazy property works like normal Python property, but its value will be cached after the first access. Below is an example about how to define and use lazy property.

{{< highlight python "linenos=inline" >}}
# define lazy_property class
class lazy_property(object):
    def __init__(self, fget):
        self.fget = fget

    # only define __get__() to make it as non-data descriptor
    def __get__(self, instance, cls):
        value = self.fget(instance)
        setattr(instance, self.fget.__name__, value)
        return value   
        
class MyTest:
    def __init__(self):
        pass
    
    # use decorator, it's same as call 
    # lp = lazy_property(func_definition...)
    @lazy_property
    def lp(self):
        print("Initialize lp")
        return "I am lazy property"

obj = MyTest()
print("======== Before access lp ========")
print("instance __dict__: {}".format(obj.__dict__))
print("======== First time access lp  ========")
print(obj.lp)
print("instance __dict__: {}".format(obj.__dict__))
print("======== Second time access lp  ========")
print(obj.lp)
{{< / highlight >}}

Output: 
```
======== Before access lp ========
instance __dict__: {}
======== First time access lp  ========
Initialize lp
I am lazy property
instance __dict__: {'lp': 'I am lazy property'}
======== Second time access lp  ========
I am lazy property
```

As you can see, `__get__()` in `lazy_property` is only called at the first time. After that, `lp` attribute will be added to instance's `__dict__`, as instance's `__dict__` has higher priority than non-data descriptor, it hides future `__get__()` call. 
# Descriptors in Python Internals
As we mentioned before, many of python internal implementations rely on descriptor protocol. Here we analyze some of them in CPython. If you are not familiar with C, [[1]](https://docs.python.org/3/howto/descriptor.html) supplies corresponding Python emulation. 

## Property
`PyProperty_Type` represents Python property class in CPython, as you can see from definition, it defines two descriptor methods: `property_descr_get()` (map to `__get__()`) and `property_descr_set()` (map to `__set__()` and `__delete__`). Obviously, Python property is a data descriptor. 
{{< highlight c "linenos=inline, hl_lines=5-6" >}}
PyTypeObject PyProperty_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "property",                                 /* tp_name */
    ...
    property_descr_get,                         /* tp_descr_get */
    property_descr_set,                         /* tp_descr_set */
    ...
};
{{< / highlight >}}

`propertyobject` defines Python property object, `prop_get`, `prop_set` and `prop_del` map to Python property's `getter`, `setter` and `deleter`.
{{< highlight c "linenos=inline" >}}
typedef struct {
    PyObject_HEAD
    PyObject *prop_get;
    PyObject *prop_set;
    PyObject *prop_del;
    PyObject *prop_doc;
    int getter_doc;
} propertyobject;
{{< / highlight >}}

Below is the implementation of `property_descr_get()`. First, it checks the attribute is accessed through instance or class, if it's accessed from class, then property object itself will be returned. Otherwise, `property_descr_get()` will call `prop_get` to return the value. 
{{< highlight c "linenos=inline" >}}
static PyObject *
property_descr_get(PyObject *self, PyObject *obj, PyObject *type)
{
    if (obj == NULL || obj == Py_None) {
        Py_INCREF(self);
        return self;
    }

    propertyobject *gs = (propertyobject *)self;
    if (gs->prop_get == NULL) {
        PyErr_SetString(PyExc_AttributeError, "unreadable attribute");
        return NULL;
    }

    return PyObject_CallOneArg(gs->prop_get, obj);
}
{{< / highlight >}}

`property_descr_set` implements both `__set__()` and `__delete__()`. Based on whether `value` parameter is NULL, it will call `prop_del` or `prop_set` respectively (line 7~10).
{{< highlight c "linenos=inline" >}}
static int
property_descr_set(PyObject *self, PyObject *obj, PyObject *value)
{
    propertyobject *gs = (propertyobject *)self;
    PyObject *func, *res;

    if (value == NULL)
        func = gs->prop_del;
    else
        func = gs->prop_set;
    if (func == NULL) {
        PyErr_SetString(PyExc_AttributeError,
                        value == NULL ?
                        "can't delete attribute" :
                "can't set attribute");
        return -1;
    }
    if (value == NULL)
        res = PyObject_CallOneArg(func, obj);
    else
        res = PyObject_CallFunctionObjArgs(func, obj, value, NULL);
    if (res == NULL)
        return -1;
    Py_DECREF(res);
    return 0;
}
{{< / highlight >}}
## Function & Method
`PyFunction_Type` defines `func_descr_get()` (`__get__()`), but doesn't define `__set__()` and `__delete__()`, so Python function/method is a non-data descriptor. 
{{< highlight c "linenos=inline, hl_lines=5-6" >}}
PyTypeObject PyFunction_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "function",
    ...
    func_descr_get,                             /* tp_descr_get */
    0,                                          /* tp_descr_set */
    ...
};
{{< / highlight >}}

`func_descr_get()` shows how Python binds `self` when you call a method from an instance. when `obj` is not NULL, `func_descr_get()` returns a new function with `obj` bound to it. 
{{< highlight c "linenos=inline" >}}
static PyObject *
func_descr_get(PyObject *func, PyObject *obj, PyObject *type)
{
    if (obj == Py_None || obj == NULL) {
        Py_INCREF(func);
        return func;
    }
    return PyMethod_New(func, obj);
}
{{< / highlight >}}
## Class Method
Similar with normal function & method. Class method is also a non-data descriptor. 
{{< highlight c "linenos=inline, hl_lines=5-6" >}}
PyTypeObject PyClassMethod_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "classmethod",
    ...
    cm_descr_get,                               /* tp_descr_get */
    0,                                          /* tp_descr_set */
    ...
};
{{< / highlight >}}

Class method could be called both from class or instance. If the method is called from instance, line 12 will get class from instance. Line 14 and 16 binds the class (which is Python `cls` parameter) to returned method. 
{{< highlight c "linenos=inline" >}}
static PyObject *
cm_descr_get(PyObject *self, PyObject *obj, PyObject *type)
{
    classmethod *cm = (classmethod *)self;

    if (cm->cm_callable == NULL) {
        PyErr_SetString(PyExc_RuntimeError,
                        "uninitialized classmethod object");
        return NULL;
    }
    if (type == NULL)
        type = (PyObject *)(Py_TYPE(obj));
    if (Py_TYPE(cm->cm_callable)->tp_descr_get != NULL) {
        return Py_TYPE(cm->cm_callable)->tp_descr_get(cm->cm_callable, type, NULL);
    }
    return PyMethod_New(cm->cm_callable, type);
}
{{< / highlight >}}

## Static Method
Statc method is almost the same as class method, but it doesn't need to bind `cls`. So `sm_descr_get()` just returns `sm_callable` directly
{{< highlight c "linenos=inline, hl_lines=5-6" >}}
PyTypeObject PyStaticMethod_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "staticmethod",
    ...
    sm_descr_get,                               /* tp_descr_get */
    0,                                          /* tp_descr_set */
    ...
};
{{< / highlight >}}
{{< highlight c "linenos=inline" >}}
static PyObject *
sm_descr_get(PyObject *self, PyObject *obj, PyObject *type)
{
    staticmethod *sm = (staticmethod *)self;

    if (sm->sm_callable == NULL) {
        PyErr_SetString(PyExc_RuntimeError,
                        "uninitialized staticmethod object");
        return NULL;
    }
    Py_INCREF(sm->sm_callable);
    return sm->sm_callable;
}
{{< / highlight >}}

# Reference

[1] https://docs.python.org/3/howto/descriptor.html

[2] [Python Descriptors Understanding and Using the Descriptor Protocol](https://www.google.com/books/edition/Python_Descriptors/n6lxDwAAQBAJ?hl=en&gbpv=0)

[3] https://github.com/python/cpython
