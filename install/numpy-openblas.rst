
==============================
Installing Numpy with OpenBLAS
==============================

These instructions are based on the very useful blog post that can be found on
`odsf's blog`_.

First, make sure you have these tools installed

.. sourcecode:: bash

  sudo apt-get install git python-dev gfortran

Also, it might make things easier to make sure that the `other` Fortran compiler,
``g77``, is not installed (though if you prefer to keep it around, you just have
to take an extra step in the Numpy install).


OpenBLAS
========

The first step is to install OpenBLAS. This can be done via ``apt-get``, but
installing from source has the advantages that OpenBLAS will be fully optimized
to your machine, and you can put it somewhere other than ``/usr/lib`` (which
should hopefully allow Octave to be installed simultaneously, though I haven't
tried this yet).

To install OpenBLAS, do the following commands

.. sourcecode:: bash

  cd ~/src
  git clone https://github.com/xianyi/OpenBLAS
  cd OpenBLAS
  make FC=gfortran
  sudo make PREFIX=/opt/openblas install

As you can see, I prefer to put my OpenBLAS install in ``/opt``, so it's out
of the way of things I install with ``apt-get``.

Finally, you have to let your system know about these new libraries. Add a
file to ``/etc/ld.so.conf.d/`` called ``openblas.conf``, containing the path
to your new libraries (``/opt/openblas/lib``). Then run ``sudo ldconfig``.

Numpy
=====

Next, we install Numpy

.. sourcecode:: bash

  cd ~/src
  git clone https://github.com/numpy/numpy
  cd ~/numpy

Add a file called ``site.cfg``, with the following lines

.. sourcecode:: linux-config

  [default]
  include_dirs = /opt/openblas/include
  library_dirs = /opt/openblas/lib

  [openblas]
  openblas_libs = openblas
  library_dirs = /opt/openblas/lib

  [lapack]
  lapack_libs = openblas
  library_dirs = /opt/openblas/lib

This file lets Numpy know where your OpenBLAS libraries are. Run
``python setup.py config`` to make sure everything is set up correctly.
You should see no mention of ATLAS. (TODO: I should try this with ATLAS
installed, to see if Numpy ignores it or tries to use it.)
If everything looks good, go ahead and call ``python setup.py build``.
If you have both Fortran compilers installed, call::

  python setup.py build --fcompiler=gnu95

to specify ``gfortran`` as the compiler (see `Numpy install notes`_).

One of the benefits of using OpenBLAS is that it can make Numpy's ``dot``
function for matrix-matrix multiplies really fast. For this to work, Numpy
needs the file ``core/_dotblas.so`` to exist. In your Numpy source directory
(where you should still be if you're following along), look under
``build/lib.linux-x86_64-2.7/numpy/core``, and make sure ``_dotblas.so`` is
there. You can also run ``ldd`` on it, to make sure that it's finding your
OpenBLAS library all right.

If everything seems good, call ``python setup.py install``. Installing to a
virtual environment is best. Otherwise, use the ``--user`` flag to install
to your home directory, or put the ``sudo`` command on front to install to
the ``/usr`` directory.

To make sure everything is working right, I run the following script

.. sourcecode:: python

  import numpy as np
  import numpy.random as npr
  import time

  ################################################################################
  ### Test 1
  N = 1
  n = 1000

  A = npr.randn(n,n)
  B = npr.randn(n,n)

  t = time.time()
  for i in range(N):
      C = np.dot(A, B)
  td = time.time() - t
  print "multiplied two (%d,%d) matrices in %0.1f ms" % (n, n, 1e3*td/N)

  ################################################################################
  ### Test 2
  N = 100
  n = 4000

  A = npr.randn(n)
  B = npr.randn(n)

  t = time.time()
  for i in range(N):
      C = np.dot(A, B)
  td = time.time() - t
  print "dotted two (%d) vectors in %0.2f us" % (n, 1e6*td/N)

  ################################################################################
  ### Test 3
  m,n = (2000,1000)

  A = npr.randn(m,n)

  t = time.time()
  [U,s,V] = np.linalg.svd(A, full_matrices=False)
  td = time.time() - t
  print "SVD of ({:d},{:d}) matrix in {:0.3f} s".format(m, n, td)

  ################################################################################
  ### Test 4
  n = 1500
  A = npr.randn(n,n)

  t = time.time()
  w, v = np.linalg.eig(A)
  td = time.time() - t
  print "Eigendecomposition of ({:d},{:d}) matrix in {:0.3f} s".format(n, n, td)

And on my machine, I get these results

.. sourcecode:: bash

  multiplied two (1000,1000) matrices in 49.8 ms
  dotted two (4000) vectors in 6.87 us
  SVD of (2000,1000) matrix in 1.192 s
  Eigendecomposition of (1500,1500) matrix in 7.805 s

If just the matrix-matrix multiply is slow, it's likely because
``core/_dotblas.so`` didn't get created, or can't find your OpenBLAS library
(see above). If the SVD and Eigendecomposition are slow, it's likely that
you have a problem with the LAPACK linking (this only happened when I tried
to use the OpenBLAS installation from ``apt-get``).


Scipy
=====

Scipy is easy to install, because it will make use of Numpy's OpenBLAS bindings.
Just run ``pip install scipy`` and you should be good to go, or install
normally from source to get the lastest development version.

.. _odsf's blog: http://osdf.github.io/blog/numpyscipy-with-openblas-for-ubuntu-1204-second-try.html
.. _Numpy install notes: http://docs.scipy.org/doc/numpy/user/install.html
