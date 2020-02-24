---
title: "Gradient"
date: 2020-02-22T13:26:17-08:00
draft: false
---

# Introduction

Gradient descent is widely used in Machine Learning. If you have heard of gradient many times, but still wonder what it is and why it is so useful, then you are in the right place. This blog will go through:

* The mathmatics fundation of gradient. 
* Application of gradient in Machine Learning

# What is Gradient 

Let's be straightforward and start from official mathmatical gradient definition from Wikipedia: 

> The **gradient** of a scalar-valued differentiable function *f* of several variables, \\({\displaystyle f\colon \mathbf {R} ^{n}\to \mathbf {R} }\\), is the vector field, or more simply a vector-valued function \\({\displaystyle \nabla f\colon \mathbf {R} ^{n}\to \mathbf {R} ^{n}}\\), whose value at a point \\({\displaystyle p}\\) is the vector whose components are the partial derivatives of \\({\displaystyle f}\\) at \\({\displaystyle p}\\).
>
> $$\nabla f(p)={\begin{bmatrix}{\frac {\partial f}{\partial x_{1}}}(p)\\\\\\vdots \\\\{\frac {\partial f}{\partial x_{n}}}(p)\end{bmatrix}}. $$

Wow, so what does it mean? Let's paraphrase the definition a little. 

1. The gradient is attached to a differentiable function \\(f\\). 
2. The gradient is a function. It transforms a vector to another vector.
3. The gradient transformation depends on partial derivatives. 

If we treat input vector as a point and the output vector as a directional vector, then we can get some important proterties of gradient: 

1. The gradient at the point \\(p\\) (input),  points to the direction (output) where function \\(f\\) increases fastestly. 
2. The length of gradient at point \\(p\\) is the instantaneous increase rat of function \\(f\\).

Now let's use some examples to show how it works. 

## One-dimensional Independent Variable

Considering function: \\(y = f(x) = x^2\\). Assuming we are at point (\\(x=2\\)), so in which direction we can increase \\(f(x)\\) fastestly? 

Without loss of generality, let's use unit vector to represent the direction. For this one-dimensional space, it's very simple. We can only move \\(x\\) in two directions - \\(\vec u_{left} = [-1]\\) and \\(\vec u_{right} = [1])\\). From the diagram below, it's clear that \\(\vec u_{right} \\) will increase \\(f\\) and \\(\vec u_{left} \\) will decrease \\(f\\). But what if we don't have the diagram at hand, how to know which direction to go?

![x^2](/ml/math/gradient/image1.svg "\\(f(x) = x^2\\)")

To answer above question, we need to use Directional Derivatives, which represents the instantaneous rate of change of \\(f\\) in some direction.

$$D_\vec u^{right} = \lim \limits_{h \to 0} \frac{f(x+h) - f(x)}{h} =  \lim \limits_{h \to 0} \frac{x^2 + 2xh + h^2 - x^2}{h}  = 2x$$  

$$D_\vec u^{left} = \lim \limits_{h \to 0} \frac{f(x-h) - f(x)}{h} =  \lim \limits_{h \to 0} \frac{x^2 - 2xh + h^2 - x^2}{h}  = -2x$$  

Given \\(\frac{\partial f(x)}{\partial x}=2x\\) and \\(\nabla f(x) = {\begin{bmatrix} {\frac{\partial f(x)}{\partial x}} \end{bmatrix}} \\),  we can merge above two formulas to: 

$$D_\vec u = [\frac{\partial f(x)}{\partial x}] \bullet \vec u = \nabla f(x) \bullet \vec u = \begin{cases} -\lVert \nabla f(x)  \rVert \bullet \lVert \vec u  \rVert,\\ if\\ \nabla\\ f(x)\\ and\\ \vec u\\ are\\ in\\ opposite\\ direction \\\\ \lVert \nabla f(x)  \rVert \bullet \lVert \vec u  \rVert,\\   if\\ \nabla f(x)\\ and\\ \vec u\\ are\\ in\\ same\\ direction \\\\\end{cases}$$

So gradient \\(\nabla f(x)\\) shows the direction and rate of fastest increase of \\(f(x)\\).

## Two-dimensional Independent Variables

Let's try another function \\(f(x,y)=x^2+y^2\\), now we have two independent variables and the independent variables space is extended to a plane of \\((x, y)\\). Assuming we are at \\((x=7,y=7)\\), which direction we can move to increase the value of \\(f(x,y)\\) fastestly?

It's not as obvious as first example, there are too many options there. For example, we can move toward point \\((x=5, y=7)\\), but the valuse of \\(f(x,y)\\) will drop from 98 to 74, so is there any good way to find the direction we want?

![x^2+y^2](/ml/math/gradient/image2.svg "\\(f(x,y) = x^2+y^2\\)")

Assuming \\(\vec u = (x_d, y_d)\\) is an unit vector in \\((x,y)\\) plane. We can define the rate of change of \\(f(x,y)\\) along \\(\vec u\\) direction at point \\((x_0, y_0)\\) by

$$D_\vec u = \lim \limits_{h \to 0}  \frac {f(x_0+h \bullet x_d, y_0+ h \bullet y_d)-f(x_0,y_0)}{h}$$

To calculate above limitation, we can take two steps to move from \\((x_0,y_0)\\) to \\((x_0+h \bullet x_d, y_0+ h \bullet y_d)\\).

1. Move from \\((x_0,\\ y_0)\\) to \\((x_0+h \bullet x_d,\\ y_0)\\).
2. Move from \\((x_0+h \bullet x_d,\\ y_0)\\) to \\((x_0+h \bullet x_d,\\ y_0+ h \bullet  y_d)\\).

![vector_move](/ml/math/gradient/vector_move.svg "vector_move")

Assuming \\(f(x,y)\\) is continuous, then we can rewite the directional derivative to

$$D_\vec u = \lim \limits_{h \to 0}  \frac {f(x_0+h \bullet x_d, y_0+ h \bullet y_d) - f(x_0+h \bullet x_d, y_0) + f(x_0+h \bullet x_d, y_0) -f(x_0,y_0)}{h} =\frac{\partial f(x_0,y_0)}{\partial x} \bullet x_d + \frac{\partial f(x_0,y_0)}{\partial y} \bullet y_d=\nabla f(x_0,y_0) \bullet \vec u$$

Provided \\(\theta\\) is the angle between \\(nabla f(x_0,y_0) \\) and \\(\vec u\\), then

$$\nabla f(x_0,y_0) \bullet \vec u = \lVert \nabla f(x_0,y_0) \rVert\\  \lVert \vec u \rVert\\ cos \theta \leq \lVert \nabla f(x_0,y_0) \rVert\\  \lVert \vec u \rVert$$

When and only when \\(\nabla f(x_0,y_0)\\) and \\(\vec u\\), in the same direction, we can have 

$$\lVert \nabla f(x_0,y_0) \rVert\\  \lVert \vec u \rVert\\ cos \theta = \lVert \nabla f(x_0,y_0) \rVert\\  \lVert \vec u \rVert=\lVert \nabla f(x_0,y_0) \rVert$$

### Multi-dimensional Independent Variables

We can continue the reasoning to the multi-dimentional independent variables. With the independent variables grow, the process of argument may will become more complicated, but the backend logic is the same. 

# Gradient in Machine Learning

## Gradient Descent
