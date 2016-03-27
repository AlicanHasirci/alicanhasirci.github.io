---
layout: post
title: "Matrices Pt.1"
date: 2016-3-27 21:25:28
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
<div>
$$\begin{bmatrix}a & b\\c & d\end{bmatrix}$$
</div>
To understand why matrices are used in computer graphics first we must see what they are capable of doing.

Matrix Arithmetics
------------------
A matrix times a constant results in a matrix where each element has been multiplied with the constant. For example:
<div>$$2\begin{bmatrix}a & b\\c & d\end{bmatrix}=\begin{bmatrix}2a & 2b \\2c & 2d\end{bmatrix}$$</div>
As for multiplying two matrices we multiply rows of the first matrix with columns of the second:
<div>$$\begin{bmatrix}a&b\\c&d\end{bmatrix}\begin{bmatrix}x&y\\z&w\end{bmatrix}=\begin{bmatrix}a.x+b.z& a.y+b.w \\c.x+d.z &c.y+d.w\end{bmatrix}$$</div>
Therefore taking a product of two matrices is only possible if the number of columns of the left matrix is the same as the number of rows of the right matrix. To represent the size of a matrix we use a notation as such:
<div>$$m \times n$$</div>
Since for left matrix column size must be same with right matrix row size, a valid multiplication between two matrices should look like this:
<div>$$m\times n \;\times\; n\times p$$</div>
Resulting matrix size depends on the left matrix's row, and right matrix's column size so:
<div>$$m\times n \;\times\; n\times p \rightarrow m \times p$$</div>
One important thing you should notice at this point is that matrix multiplications are not commutative. Not only they do not produce the same result, in some cases it may not be valid at all. Thus multiply matrix A with matrix B is not the same thing as multiply B with A: $$A\times B\neq B \times A$$

Although matrix multiplication is not commutative directly, it can be achieved by multiplying the transpose matrices. Considering transpose of $$A$$ is $$A^T$$ and transpose of $$B$$ is $$B^T$$: 
<div>$$A\times B= B^T \times A^T$$</div>
To calculate transpose of a matrix all you have to do is switch the rows for columns. Like so which makes in a way since doing this will swap the row and column count:
<div>$$A=\begin{bmatrix}a&b\\c&d\end{bmatrix}$$
$$A^T=\begin{bmatrix}a&c\\b&d\end{bmatrix}$$</div>
While this may look insignificent for now, it will be useful when we will talk about matrix forms of vectors since there are two different ways a vector can be represented(column-form, row-form).

Identity Matrix
---------------
There is a special case for matrices where they can have inverses. Just like multiplying $$x$$ with $$1/x$$ would give you $$1$$, multiply a matrix with it's inverse gives you "matrix one" which is called an identity matrix, this also means that multiplying a matrix with identity matrix gives the matrix itself. Identity matrix exists only as a square matrix and it consists of ones down the diagonal and zeroes elsewhere. For example the four by four indentity matrix is:
<div>$$I=\begin{bmatrix}1&0&0&0\\0&1&0&0\\0&0&1&0\\0&0&0&1\end{bmatrix}$$</div>
So while $$A^{-1}A=I$$ is true $$AI=A$$ is also true.

