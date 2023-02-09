---
title: "Tinygrad"
date: 2023-01-12T17:25:18+01:00
draft: true
---

In this post, I am going to explore the tinygrad deep learning framework with the aim of making a contribution: a bug fix, refactor or new tests. First, the high level view of the project structure and of each file.

```
.
├── accel    # high perf hardware specific code.
├── datasets
├── docs     # "extended comments" shall we say.
├── examples # tinygrad use cases.
├── extra    # no clue.
├── models   # ML model impls.
├── test
├── tinygrad # < 1000 line core
└── weights  # some model's weights.
```

The simplest place to start is by looking at `mnist_gan.py` in the examples folder.

```python
class LinearGen:
  def __init__(self):
    lv = 128
    self.l1 = Tensor.uniform(128, 256)
    self.l2 = Tensor.uniform(256, 512)
    self.l3 = Tensor.uniform(512, 1024)
    self.l4 = Tensor.uniform(1024, 784)

  def forward(self, x):
    x = x.dot(self.l1).leakyrelu(0.2)
    x = x.dot(self.l2).leakyrelu(0.2)
    x = x.dot(self.l3).leakyrelu(0.2)
    x = x.dot(self.l4).tanh()
    return x
```

A tensor is a multi-dimensional matrix, so `l1, l2, l3, l4` look like the nn layer paramaters/weights. `l1` has 128 neurons, `l2` has 256, ..., and `l4` has 1024. Below summarises the data stored by the `Tensor` class.

```python
class Tensor:
  training, no_grad = False, False
  def __init__(self, data, device=Device.DEFAULT, requires_grad=None):
    self.lazydata = # omitted code
    self.grad : Optional[Tensor] = None # tensors have gradients, buffers do not
    self.requires_grad : Optional[bool] = requires_grad
    self._ctx : Optional[Function] = None # internal variables used for autograd graph construction
```