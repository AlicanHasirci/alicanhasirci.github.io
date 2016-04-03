---
layout: post
title: "Matrices Pt.2"
date: 2016-4-3 21:05:28
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

Now that we've talked about matrices a bit maybe it is time to talk about their use in computer graphics. As you know foundations of computer graphics is built on linear algebra, to be more specific points and vectors on 3D space.


Think about a wooden frame with a canvas stretched on it, the wooden frame is your 'mesh' and the canvas is your 'texture', the way you define this canvas on 3D space is by giving the coordinates of the frame corners(actually you you need to define a few more than that but for simplicity sake i'll skip that part). Now image a gallery full of these canvases with different paintings on them, and you want to move them around all at the same time. Say we have hundred paintings, this makes 400 corners(every 'corner' in this example is actually called a vertex in CG context), moving them all one by one can be easy after all 400 actions to take but lets say you want to first move them a bit, then rotate and move them again, you see that applying all these transformations to all corners one at a time can get out of hand quite easily and thats where matrices come in to play.


In graphics, we use square matrix to transform a vector represented as a matrix. For example if you have a 2D vector $$a=(x_a,y_a)$$ and want to rotate it by 90 degress about the origin to for ma vector $$a'=(-y_a,xa)$$, you can use a product of a $$2 \times 2$$ matrix and a $$2 \times 1 $$ matrix, called a 'column vector'. The operation in the matrix for is
<div>
$$\begin{bmatrix}0 & -1 \\ 1 & 0\end{bmatrix}\begin{bmatrix}x_a \\y_a\end{bmatrix} = \begin{bmatrix}-y_a \\x_a\end{bmatrix}$$
</div>
We can get the same result by using the transpose of this matrix and multiplying on the left ("premultiplying") with a row vector:
<div>
$$\begin{bmatrix}x_a&y_a\end{bmatrix}\begin{bmatrix}0 & 1 \\ -1 & 0\end{bmatrix} = \begin{bmatrix}-y_a &x_a\end{bmatrix}$$
</div>
These days postmultiplication is pretty much standard but some books on the subject still have examples with premultiplication.


Let's talk on an example for a better grasp on the subject. We'll start with a point $$A=(1,1)$$ or $$A=\begin{bmatrix}1\\1\\1\end{bmatrix}$$ in matrix form. Notice the $$1$$ as the third coordinate? It may strike you as a $$z$$ coordinate but actually it is used to first translation of points. Imagine you want to translate our point to $$A'=(3,3)$$, all you have to do is to use a translation matrix:
<div>$$T_{a,b}=\begin{bmatrix}1 & 0 & a \\ 0 & 1 & b \\ 0 & 0 & 1\end{bmatrix}$$</div>
And then you can multiply it with your point.
<div>
$$\begin{bmatrix}1 & 0 & 2 \\ 0 & 1 & 2 \\ 0 & 0 & 1\end{bmatrix}\begin{bmatrix}1\\1\\1\end{bmatrix} = \begin{bmatrix}3 \\3\\1\end{bmatrix}$$
</div>

Now lets step away from a point and talk about a square with coordinates as such $$(1,1) (1,-1) (-1,-1) (-1,1)$$. Applying this translation matrix to every vertex would shift our square 2 units up and right. This may not seem partical yet but bare with me. Lets talk about scaling and i promise the reason will get clearer.


Scaling is pretty straight-forward with matrices also, just multiply the coordinates with the given number,
<div>
$$S_{a,b}=\begin{bmatrix}a & 0 & 0 \\ 0 & b & 0 \\ 0 & 0 & 1\end{bmatrix}$$
</div> 
and there you have it but don't relax yet since there is satback about scaling. Lets talk about our cube again, by multiplying every point with $$S_{2,2}$$ would double it's size that is true but what happens when the center of is not on the origin? Lets say we translated our cube by $$T_{1,1}$$ now our coordinates are starting from lower left corner going clockwise respectively, $$(0,0) (0,2) (2,2) (2,0)$$, you see rather then growing to all directions homogenously, the square grows to the opposite direction of origin, actually that is always the case. This is why to get a homogenous scaling the square should be translated back to origin, scaled and translate back again. So it is in order $$T_{-1,-1} S_{2,2} T_{1,1}$$. Now apply all this transformations to every point maybe burdensome when you have a lot of points but here comes the forte of matrices. Multiplying all these matrices together to one handsome matrix, we can deduce all these translations in to one matrix and apply it to points one by one will give the same effect as applying them in turn.

So:
<div>
$$\begin{bmatrix}1&0&-1\\0&1&-1\\0&0&1\end{bmatrix}
\begin{bmatrix}2&0&0\\0&2&0\\0&0&1\end{bmatrix}
\begin{bmatrix}1&0&1\\0&1&1\\0&0&1\end{bmatrix}=
\begin{bmatrix}2&0&1\\0&2&1\\0&0&1\end{bmatrix}$$
</div>

As for rotations in 2D they are a lot simpler than their 3D version. A rotation for $$\theta$$ angles is annotated as so $$R_\theta$$ and here is the rotation matrix:
<div>
$$R_\theta=\begin{bmatrix}\cos\theta &\sin\theta&0\\-\sin\theta&\cos\theta&0\\0&0&1\end{bmatrix}$$
</div>