.. figure:: _static/logo/keops_logo.png
   :width: 100% 
   :alt: Keops logo

Kernel Operations on the GPU, with autodiff, without memory overflows
---------------------------------------------------------------------

The KeOps library lets you compute generic reductions of **very large arrays** 
whose entries are given by a mathematical formula. 
It combines a **tiled reduction scheme** with an **automatic differentiation** 
engine, and can be used through **Matlab**, **NumPy** or **PyTorch** backends.
It is perfectly suited to the computation of **Kernel dot products**
and the associated gradients,
even when the full kernel matrix does *not* fit into the GPU memory.

Using the **PyTorch backend**, a typical sample of code looks like:

.. code-block:: python

    # Create two arrays with 3 columns and a (huge) number of lines, on the GPU
    import torch
    x = torch.randn(1000000, 3, requires_grad=True).cuda()
    y = torch.randn(2000000, 3).cuda()

    # Turn our Tensors into KeOps symbolic variables:
    from pykeops.torch import LazyTensor
    x_i = LazyTensor( x[:,None,:] )  # x_i.shape = (1e6, 1, 3)
    y_j = LazyTensor( y[None,:,:] )  # y_j.shape = ( 1, 2e6,3)

    # We can now perform large-scale computations, without memory overflows:
    D_ij = ((x_i - y_j)**2).sum(dim=2)  # Symbolic (1e6,2e6,1) matrix of squared distances
    K_ij = (- D_ij).exp()               # Symbolic (1e6,2e6,1) Gaussian kernel matrix

    # And come back to vanilla PyTorch Tensors or NumPy arrays using
    # reduction operations such as .sum(), .logsumexp() or .argmin().
    # Here, the kernel density estimation   a_i = sum_j exp(-|x_i-y_j|^2)
    # is computed using a CUDA online map-reduce routine that has a linear
    # memory footprint and outperforms standard PyTorch implementations
    # by two orders of magnitude.
    a_i = K_ij.sum(dim=1)  # Genuine torch.cuda.FloatTensor, a_i.shape = (1e6, 1), 
    g_x = torch.autograd.grad((a_i ** 2).sum(), [x])  # KeOps supports autograd!

KeOps allows you to leverage your GPU without compromising on usability.
It provides:

* **Linear** (instead of quadratic) **memory footprint** for Kernel operations.
* Support for a wide range of mathematical **formulas**.
* Seamless computation of **derivatives**, up to arbitrary orders.
* Sum, LogSumExp, Min, Max but also ArgMin, ArgMax or K-min **reductions**.
* A **conjugate gradient solver** for e.g. large-scale spline interpolation or kriging, Gaussian process regression.
* An interface for **block-sparse** and coarse-to-fine strategies.
* Support for **multi GPU** configurations.

KeOps can thus be used in a wide variety of settings, 
from shape analysis (LDDMM, optimal transport...)
to machine learning (kernel methods, k-means...)
or kriging (aka. Gaussian process regression).
More details are provided below:

* :doc:`Documentation <introduction/why_using_keops>`
* `Source code <https://github.com/getkeops/keops>`_
* :doc:`Learning KeOps with tutorials <_auto_tutorials/index>`
* :doc:`Gallery of examples <_auto_examples/index>`
* :doc:`Benchmarks <_auto_benchmarks/index>`

**KeOps is licensed** under the `MIT license <https://github.com/getkeops/keops/blob/master/licence.txt>`_.

Projects using KeOps
--------------------

As of today, KeOps provides core routines for:

* `Deformetrica <http://www.deformetrica.org>`_, a shape analysis software
  developed by the `Aramis <https://www.inria.fr/en/teams/aramis>`_ Inria team.
* `GeomLoss <http://www.kernel-operations.io/geomloss>`_, a multiscale
  implementation of Kernel and **Wasserstein distances** that scales up to
  **millions of samples** on modern hardware.
* `FshapesTk <https://plmlab.math.cnrs.fr/benjamin.charlier/fshapesTk>`_ and the
  `Shapes toolbox <https://plmlab.math.cnrs.fr/jeanfeydy/shapes_toolbox>`_,
  two research-oriented `LDDMM <https://en.wikipedia.org/wiki/Large_deformation_diffeomorphic_metric_mapping>`_ toolkits.

Authors
-------

Feel free to contact us for any bug report or feature request:

- `Benjamin Charlier <http://imag.umontpellier.fr/~charlier/>`_
- `Jean Feydy <http://www.math.ens.fr/~feydy/>`_
- `Joan Alexis Glaunès <http://www.mi.parisdescartes.fr/~glaunes/>`_


Table of content
----------------

.. toctree::
   :maxdepth: 2

   introduction/why_using_keops
   introduction/installation

.. toctree::
   :maxdepth: 2
   :caption: KeOps

   api/math-operations
   api/autodiff
   api/road-map

.. toctree::
   :maxdepth: 2
   :caption: PyKeops

   python/index
   _auto_tutorials/index
   _auto_examples/index
   _auto_benchmarks/index
   python/api/index

.. toctree::
   :maxdepth: 2
   :caption: KeopsLab

   matlab/index

.. toctree::
   :maxdepth: 2
   :caption: Keops++

   cpp/index

