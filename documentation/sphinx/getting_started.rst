Getting Started
===============

Consider the situation where the entries of a matrix :math:`A \equiv A(x)\in \mathbb R^{M \times N}` is computed
by a computer program, where :math:`x \in \mathbb R^{N_x}`. To give an explicit example we consider

.. math::
   A(x) = \begin{pmatrix}
   \sin(x_1)^2 + x_2 & x_1 \\
   e^{x_1/x_2} & x_3 \\
   \log(x_1 + x_3*x_2) & 0 \\
   \end{pmatrix}

In a second step it is desired to compute

.. math::
    \Phi(x) = \max( \lambda( (A(x)^T A(x))^{-1}) \;,
    
where :math:`\lambda(C)` computes all eigenvalues of the matrix :math:`C` and
:math:`\max` returns the largest of the eigenvalues. The matrix inversion is not
really necessary since one could as well invert the smallest eigenvalue. It is 
used here simply to make the point that its easy to concatenate matrix functions
together.

We are interested in the numerical value of the gradient

.. math::
    \nabla_x \Phi(x)
    
at :math:`x=(3,5,7)^T`. At first we look at the forward mode of AD. E.g. we want to compute
:math:`\frac{\partial \Phi}{\partial x_1}`.

The corresponding code is::
    
    >>> import numpy
    >>> from algopy import UTPM, eigh, inv, dot
    >>> 
    >>> x = UTPM(numpy.zeros((2,1,3)))
    >>> x.data[0,0] = [3,5,7]
    >>> x.data[1,0] = [1,0,0]
    >>> 
    >>> A = UTPM(numpy.zeros((2,1,3,2)))
    >>> A[0,0] = numpy.sin(x[0])**2 + x[1]
    >>> A[0,1] = x[0]
    >>> A[1,0] = numpy.exp(x[0]/x[1])
    >>> A[1,1] = x[2]
    >>> A[2,0] = numpy.log(x[0] + x[2]*x[1])
    >>> 
    >>> print 'A =', A
    A = [[[[ 5.01991486  3.        ]
       [ 1.8221188   7.        ]
       [ 3.63758616  0.        ]]]
    
    
     [[[-0.2794155   1.        ]
       [ 0.36442376  0.        ]
       [ 0.02631579  0.        ]]]]
    >>> 
    >>> y = eigh(inv(dot(A.T, A)))[0][-1]
    >>> 
    >>> print 'Phi(x) = ', y.data[0]
    Phi(x) =  [ 0.04784897]
    >>> print 'd/dx_1 Phi(x) = ', y.data[1]
    d/dx_1 Phi(x) =  [ 0.01173805]
    
    
The output of `print 'A =', A` is the contents of `A.data` with shape
`A.data.shape = (2,1,3,2)`. The first block corresponds to `A.data[0]`
and is simply the normal function evaluation. In the block `A.data[1]`
one has the partial derivatives :math:`\frac{\partial A}{\partial x_1}`.
Similarly for `y.data[0]` which is the normal function evaluation and
`y.data[1]` is the partial derivative :math:`\frac{\partial \Phi}{\partial x_1}`.
To compute the complete gradient one could repeat the above procedure but setting::
    
    x.data[1] = [0,1,0]
    
resp::
    
    x.data[1] = [0,0,1]
    
To reduce overhead, ALGOPY offers the possibility to propagate `P` directions 
at once. It also allows to compute higher-order derivatives. I.e. compute not only
the zeroth and first Taylor coefficients but the first `D` coefficients.
Then the program would look like::
    
    >>> import numpy
    >>> from algopy import UTPM, eigh, inv, dot
    >>> 
    >>> D,P,Nx,M,N = 2,3,3,3,2
    >>> 
    >>> x = UTPM(numpy.zeros((D,P,Nx)))
    >>> x.data[0,:] = [3,5,7]
    >>> x.data[1,:] = numpy.eye(Nx)
    >>> 
    >>> A = UTPM(numpy.zeros((D,P,M,N)))
    >>> A[0,0] = numpy.sin(x[0])**2 + x[1]
    >>> A[0,1] = x[0]
    >>> A[1,0] = numpy.exp(x[0]/x[1])
    >>> A[1,1] = x[2]
    >>> A[2,0] = numpy.log(x[0] + x[2]*x[1])
    >>> 
    >>> y = eigh(inv(dot(A.T, A)))[0][-1]
    >>> 
    >>> print 'Phi(x) = ', y.data[0]
    Phi(x) =  [ 0.04784897  0.04784897  0.04784897]
    >>> print 'd/dx_1 Phi(x) = ', y.data[1]
    d/dx_1 Phi(x) =  [ 0.01173805 -0.01228258 -0.00893191] 
