---
layout: default
---

# How to create proper pytorch tensors

Lets name each dimension as `M, N, K`. The shape of A should always be `(M, K)` and B should always be `(K, N)`

```python
import torch

# the tensor A shape is (M, K) and stride is (K, 1)
# this makes A, row-major
A_row = torch.ones((M, K))

# the tensor B shape is (K, N) and stride is (N, 1)
# this makes B, column-major
B_row = torch.ones((K, N))

# the tensor A shape is (K, M) and stride is (M, 1)
A_col = torch.ones((K, M))
# As A should be (M, K), we do the transpose
# After transpose, A_col.t(), shape is (M, K) and stride is (1, M)
A_col = A_col.t()

# the tensor B shape is (N, K) and stride is (K, 1)
B_col = torch.ones((N, K))
# As B should be (K, N), we do the transpose
# After transpose, B_col.t(), shape is (K, N) and stride is (1, K)
B_col = B_col.t()
```
