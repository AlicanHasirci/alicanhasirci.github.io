---
layout: post
title: "Matrices Pt.1"
date: 2016-3-13 21:25:28
image: '/assets/img/'
description:
tags:
- matrices
- matrix
- computer
- graphics
categories:
- computergraphics
twitter_text:
---

Matrices are frequently used in coputer graphics for a veriety of purposes including representation of spatial transforms. Reason for this is the characteristics of a matrix but we will get to that. Lets start with a simple question first. What is a matrix? A matrix is an array of numeric elements that follow certain arithmetic rules. Here is an example.
$$\begin{bmatrix}a & b\\c & d\end{bmatrix}$$
To understand why matrices are used in computer graphics first we must see what they are capable of doing.

Matrix Arithmetics
==================
A matrix times a constant results in a matrix where each element has been multiplied with the constant. For example:
$$2\begin{bmatrix}a & b\\c & d\end{bmatrix}=\begin{bmatrix}2a & 2b \\2c & 2d\end{bmatrix}$$
As for multiplying two matrices we multiply rows of the first matrix with columns of the second:
$$\begin{bmatrix}a&b\\c&d\end{bmatrix}\begin{bmatrix}x&y\\z&w\end{bmatrix}=\begin{bmatrix}a.x+b.z& a.y+b.w \\c.x+d.z &c.y+d.w\end{bmatrix}$$
Therefore taking a product of two matrices is only possible if the number of columns of the left matrix is the same as the number of rows of the right matrix. Since we represent the matrix size like this:
$$m \times n$$
A valid multiplication between two matrices should look like this:
$$m\times n \; n\times p \rightarrow m \times p$$
One important thing you should notice at this point is that matrix multiplications are not commutative. Not only they do not produce the same result, in some cases it may not be valid. Therefore:
$$A\times B\neq B \times A$$
Although commutativity can be achieved by multiplying the transpose matrices in which case:
$$A\times B= B^T \times A^T$$
And to get the transpose of a matrix all you have to do is switch the rows for columns. Like this:
$$A=\begin{bmatrix}a&b\\c&d\end{bmatrix}$$
$$A^T=\begin{bmatrix}a&c\\b&d\end{bmatrix}$$
This information will be useful when we will talk about matrix forms of vectors since there are two different ways a vector can be represented(column-form, row-form).

There is a special case for matrices where they can have inverses. Just like multiplying *x* with *1/x* would give you *1*, multiply a matrix with it's inverse gives you "matrix one" which is called an identity matrix. This exists only for the square matrices and it consists of ones down the diagonal and zeroes elsewhere. For example the four by four indentity matrix is:
$$I=\begin{bmatrix}1&0&0&0\\0&1&0&0\\0&0&1&0\\0&0&0&1\end{bmatrix}$$
So:
$$A^{-1}A=I$$
