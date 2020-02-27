---
title: "Gradient"
date: 2020-02-22T13:26:17-08:00
draft: false
---

# Introduction

Gradient descent is widely used in machine learning for parameters optimization. If you have heard of gradient many times, but still wonder what is behind it, then you are in the right place. This page will go through:

* The mathematic foundation of gradient. 
* Application of gradient in Machine Learning

# What is Gradient 

Let's be straightforward and start from official mathematic definition for gradient ([Wikipedia][https://en.wikipedia.org/wiki/Gradient]): 

> The **gradient** of a scalar-valued differentiable function *f* of several variables, \\({\displaystyle f\colon \mathbf {R} ^{n}\to \mathbf {R} }\\), is the vector field, or more simply a vector-valued function \\({\displaystyle \nabla f\colon \mathbf {R} ^{n}\to \mathbf {R} ^{n}}\\), whose value at a point \\({\displaystyle p}\\) is the vector whose components are the partial derivatives of \\({\displaystyle f}\\) at \\({\displaystyle p}\\).
>
> $$\nabla f(p)={\begin{bmatrix}{\frac {\partial f}{\partial x_{1}}}(p)\\\\\\vdots \\\\{\frac {\partial f}{\partial x_{n}}}(p)\end{bmatrix}}. $$

Well, what does it mean? Let's paraphrase it. 

1. The gradient is attached to a differentiable function \\(f\\). 
2. The gradient is a function. It transforms a vector to another vector.
3. The gradient transformation depends on partial derivatives. 

If we treat input vector as a point and output vector as a directional vector, then we can get some important proterties of gradient: 

1. The gradient value at the point \\(p\\) (input),  points to the direction (output) where function \\(f\\) increases fastestly. 
2. The length of gradient value at point \\(p\\) is the instantaneous increase rate of function \\(f\\).

Now let's use some examples to show how it works. 

## One-dimensional Independent Variable

Considering function: \\(y = f(x) = x^2\\). Assuming we are at point (\\(x_0=2\\)), now question is in which direction we can increase \\(f(x)\\) fastestly? 

Without loss of generality, let's use unit vector to represent the direction. For this one-dimensional space, it's very simple. We can only move \\(x\\) in two directions - \\(\vec u_{left} = [-1]\\) and \\(\vec u_{right} = [1])\\). From the diagram below, it's clear that \\(\vec u_{right} \\) will increase \\(f\\) and \\(\vec u_{left} \\) will decrease \\(f\\). But what if we don't have the diagram at hand, how to know which direction to go?

![x^2](/ml/math/gradient/image1.svg "\\(f(x) = x^2\\)")

To answer it, we need to use Directional Derivatives, which represents the instantaneous rate of change of \\(f\\) in a direction. Below are Directional Derivatives for right and left direction:

$$D_\vec u^{right} = \lim \limits_{h \to 0} \frac{f(x+h) - f(x)}{h} =  \lim \limits_{h \to 0} \frac{x^2 + 2xh + h^2 - x^2}{h}  = 2x$$  

$$D_\vec u^{left} = \lim \limits_{h \to 0} \frac{f(x-h) - f(x)}{h} =  \lim \limits_{h \to 0} \frac{x^2 - 2xh + h^2 - x^2}{h}  = -2x$$  

Given \\(\nabla f(x) = {\begin{bmatrix} {\frac{\partial f(x)}{\partial x}} \end{bmatrix}} =[2x]\\) and \\(\nabla f(x_0) =[4]\\),  we can merge above two formulas to: 

$$D_\vec u = [\frac{\partial f(x)}{\partial x}] \bullet \vec u = \nabla f(x) \bullet \vec u = \begin{cases} -\lVert \nabla f(x)  \rVert \bullet \lVert \vec u  \rVert,\\ when\ \vec u\\ =[-1] \\\\ \lVert \nabla f(x)  \rVert \bullet \lVert \vec u  \rVert,\\  when\  \vec u\\ =[1] \\\\\end{cases}$$

So gradient \\(\nabla f(x)\\) shows the direction and rate of fastest increase of \\(f(x)\\).

## Two-dimensional Independent Variables

Let's try another function \\(f(x,y)=x^2+y^2\\), now we have two independent variables and the independent variables space is extended to the plane of \\((x, y)\\). Assuming we are at \\((x_0=7,y_0=7)\\), which direction we can move to increase the value of \\(f(x,y)\\) fastestly?

It's not as obvious as first example, there are too many options there. For example, we can move toward point \\((x=5, y=7)\\), but \\\(f(x,y)\\) will decrease from 98 to 74. Is there any good way to find the direction we want?

![x^2+y^2](/ml/math/gradient/image2.svg "\\(f(x,y) = x^2+y^2\\)")

Assuming \\(\vec u = (x_d, y_d)\\) is an unit vector in \\((x,y)\\) plane. We can define the rate of change of \\(f(x,y)\\) along \\(\vec u\\) direction at point \\((x_0, y_0)\\) by

$$D_\vec u = \lim \limits_{h \to 0}  \frac {f(x_0+h \bullet x_d, y_0+ h \bullet y_d)-f(x_0,y_0)}{h}$$

To calculate above limitation, we can take two steps to move from \\((x_0,y_0)\\) to \\((x_0+h \bullet x_d, y_0+ h \bullet y_d)\\).

1. Move from \\((x_0,\\ y_0)\\) to \\((x_0+h \bullet x_d,\\ y_0)\\).
2. Move from \\((x_0+h \bullet x_d,\\ y_0)\\) to \\((x_0+h \bullet x_d,\\ y_0+ h \bullet  y_d)\\).

![vector_move](/ml/math/gradient/vector_move.svg "vector_move")

Assuming \\(f(x,y)\\) is continuous, then we can rewite the directional derivative to

$$D_\vec u = \lim \limits_{h \to 0}  \frac {f(x_0+h \bullet x_d, y_0+ h \bullet y_d) - f(x_0+h \bullet x_d, y_0) + f(x_0+h \bullet x_d, y_0) -f(x_0,y_0)}{h} =\frac{\partial f(x_0,y_0)}{\partial x} \bullet x_d + \frac{\partial f(x_0,y_0)}{\partial y} \bullet y_d=\nabla f(x_0,y_0) \bullet \vec u$$

Provided \\(\theta\\) is the angle between \\(\nabla f(x_0,y_0) \\) and \\(\vec u\\), then

$$\nabla f(x_0,y_0) \bullet \vec u = \lVert \nabla f(x_0,y_0) \rVert\\  \lVert \vec u \rVert\\ cos \theta \leq \lVert \nabla f(x_0,y_0) \rVert\\  \lVert \vec u \rVert$$

When and only when \\(\nabla f(x_0,y_0)\\) and \\(\vec u\\), in the same direction, we can have 

$$\lVert \nabla f(x_0,y_0) \rVert\\  \lVert \vec u \rVert\\ cos \theta = \lVert \nabla f(x_0,y_0) \rVert\\  \lVert \vec u \rVert=\lVert \nabla f(x_0,y_0) \rVert$$

### Multi-dimensional Independent Variables

We can continue the same analysis with multi-dimentional independent variables. With the dimension grows, the process of argument will become more complicated, but the backend logic is the same. 

# Gradient in Machine Learning

## Gradient Descent

Gradient descent is an optimization algorithm for finding the local minimum of a differentiable function. Let's go through the algorithm with an example. 

**Example**: Assuming we have a group of training data which represents people's height and weight: 

$$[h_i, w_i], 1 \leq i \leq n$$

Now we want to train a model to predict one's weight based on height. To keep simple, let's choose linear regression as model. e.g. we assume people's height and weight hold below relationship: 

$$wight = a \bullet height +b$$

Herein, \\(a\\) and \\(b\\) are two unknown parameters we will train later. How do we know which \\(a\\) and \\(b\\) are good for our prediction? Below funciton indicates the error between estimated weight and actual weight in training data. 

$$f(a,b) = \frac {1}{n} \sum_{i=1}^n (a \bullet h_i + b - w_i)^2$$ 

Obviously, We should find \\(a\\) and \\(b\\) to make above error function get minimum. To achieve that, we can start from a random point \\((a_0, b_0)\\), then keep moving the point to reduce \\(f(a,b)\\) gradually. As we have known, \\(\nabla f(a_0,b_0)\\) is the direction where \\(f(a,b)\\) increases fastestly, so the opposite direction will decrease \\(f(a,b)\\) fastestly. the gradient descent algorithm could be described as below: 

1. Select an initial point \\((a_0, b_0)\\), select a learning rate \\(\eta\\)
2. Set epoch indicator \\(i=0\\).
3. Check if \\(f(a_i, b_i)\\) is smaller than predefined threshold, if so, return \\(a_i, b_i\\) and exit.
4. Update \\(\begin{bmatrix} a_{i+1} \\\\b_{i+1} \end{bmatrix} =   \begin{bmatrix} a_{i} \\\\b_{i} \end{bmatrix} - \eta \bullet \nabla f(a_i,b_i)\\)
5. \\(i++\\), repeat step 3. 

\\(\eta\\) is the learning rate, it represents how far we want to go in each epoch. 

## Three Types of Gradient Descent

In above example, we use all the training data when updating parameters. When the training dataset becomes huge, the computation will be expensive. To get better performance, there are 3 different types of gradient descent used in the industry. 

1. **Batch Gradient Descent:** Parameters are updated after computing the gradient of error with respect to the entire training set (the same as our example).
2. **Stochastic Gradient Descent:** Parameters are updated after computing the gradient of error with respect to a single training example.
3. **Mini-Batch Gradient Descent:** Parameters are updated after computing the gradient of error with respect to a subset of the training set. The subset of trainning set is called **Mini-Batch**.

# Conclusion 

In this page, we reviewed mathematic definition of gradient and describe how to use gradient descent to optimize parameters in machine learning. It's always interesting to see how abstract mathematic principle been applied in real world. 
