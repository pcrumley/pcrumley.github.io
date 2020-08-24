---
title: "Plotting Magnetic Field Lines in 2D? Here's what you need to know."
categories:
  - Blog
tags:
  - DataViz
  - Tips & Tricks
---

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/katex.min.css" integrity="sha384-yFRtMMDnQtDRO8rLpMIKrtPCD5jdktao2TV19YiZYWMDkUR5GQZR/NOVTdquEx1j" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/katex.min.js" integrity="sha384-9Nhn55MVVN0/4OFx7EE5kpFBPsEMZxKTCnA+4fqDmg12eCTqGi6+BB2LjY8brQxJ" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/contrib/auto-render.min.js" integrity="sha384-kWPLUVMOks5AQFrykwIup5lo0m3iMkkHrD0uJ4H5cjeGihAutqP0yW0J6dpFiVkI" crossorigin="anonymous" onload="renderMathInElement(document.body);"></script>


One of the major difficulties of understanding plasmas is just getting a handle on
what the magnetic field is doing. If you're lucky enough to be running
simulations in 2D, you'll have the value of the magnetic field at every point on
your grid, but it still can be hard to understand the geometry of it all.
That's where [field lines](https://en.wikipedia.org/wiki/Magnetic_field) come
in.

There's this trick that is passed around by word of mouth amongst theoretical
plasma physicists, but I can't find it online or in print anywhere. Hopefully
the solid use of SEO in my blog title will help it become more widely known
(Don't @ me, [Michael Barbaro](https://www.nytimes.com/column/the-daily)).
The trick makes calculating field lines in 2D very fast, but as you'll
see, it only works for magnetic or other divergence-free 2D vector fields.

In 2D, the field lines in the \\(xy\\) plane can be calculated as lines of
equipotential of \\(A_z\\). The trick is: **you can calculate \\(A_z\\) at any
point in the grid by integrating \\(-B_y\\) along \\(x\\), and \\(B_x\\) along
\\(y\\).**

If you want to just [believe me](https://en.wikipedia.org/wiki/Argument_from_authority)
and skip the vector calculus, scroll to the  bottom for a simple python snippet
showing how to do use this trick efficiently. Otherwise, let's take a little
detour to prove we've made a valid construction of \\(A_z\\).


### Simple calculation of the of the magnetic vector potential, \\(A_z\\).

Recall Maxwell's EQ's.
\\[ \mathbf{E} = \nabla V - \frac{\partial \mathbf{A}}{\partial t} \\]
\\[ \mathbf{B} = \nabla \times \mathbf{A} \\]

We are free to choose any \\(A\\) that satisfies the above equation, due to gauge
freedom. Calculate curl of \\(A\\)
\\[ \nabla \times \mathbf{A} =
  \hat{i}\left[\frac{\partial A_z}{\partial y} - \frac{\partial A_y}{\partial z}\right]
  -\hat{j}\left[\frac{\partial A_z}{\partial x} - \frac{\partial A_x}{\partial z}\right]
  +\hat{k}\left[\frac{\partial A_y}{\partial x} - \frac{\partial A_x}{\partial y}\right]
\\]
Now because the simulation is \\(2D\\), we know that
\\( \frac{\partial}{\partial z} \\) is zero for all quantities. Furthermore,
we only need to know \\(A_z\\), so we just need to construct any continuous and
infinitely differentiable \\(A_z\\) that is a valid solution to
the following set of partial differential equations.
\\[ B_x = \frac{\partial A_z}{\partial y} \\]
\\[ B_y = -\frac{\partial A_z}{\partial x} \\]
One choice of \\(A_z\\) that satisfies the above equations is the following:
\\[ A_z(x,y) = - \int_0^{x}{B_y(x',0)\ dx'}  + \int_0^{y}{B_x(x,y')\ dy'}.\\]

Hopefully you can
[convince yourself](https://en.wikipedia.org/wiki/Fundamental_theorem_of_calculus)
that \\(\frac{\partial A_z}{\partial y} = B_x \\), So we'll just show the algebra for
\\[B_y = -\frac{\partial A_z}{\partial x} = B_y(x,0) - \int_0^y{ \frac{\partial B_x}{\partial x}\ dy'}. \\]

Because \\(\nabla \cdot B = 0\\) and fields are 2D;
\\(\frac{\partial B_x}{\partial x} = -\frac{\partial B_y}{\partial y}.\\)
Substituting this in...
\\[-\frac{\partial A_z}{\partial x} = B_y(x,0) + \int_0^y{ \frac{\partial B_y}{\partial y}\ dy'} = B_y(x,0)  + B_y(x, y) - B_y(x,0) \\]
or
\\[B_y = -\frac{\partial A_z}{\partial x}.\\]

Which is precisely what we needed to show, so we have proven our choice of
\\(A_z\\) is valid. However, you explicitly use the fact the field is
solenoidal to calculate the field lines, so this trick will not work for
electric fields. You'll need to do the full Helmholtz decomposition or trace
them out as streamlines for that.

Let's say you have two 2D numpy arrays, ```bx``` and ```by```, corresponding
to the \\(x\\) and \\(y\\) components of the field respectively. With some
Numpy magic and disregarding units, \\(A_z\\) can be calculated and field lines
plotted with:

```python
  import numpy as np
  import matplotlib.pyplot as plt
  bydx = -np.cumsum(by[0, :])
  bxdy = np.cumsum(bx[:, :], axis=0)
  Az = bydx[np.newaxis, :] + bxdy
  plt.contour(Az)
```

Isn't it beautiful to program with multidimensional arrays? Thank you,
[Kenneth E. Iverson](https://en.wikipedia.org/wiki/APL_(programming_language))
