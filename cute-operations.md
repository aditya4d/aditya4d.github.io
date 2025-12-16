---
layout: default
---

# Operations in CuTe

This post has explanation of different operations in CuTe.

### `make_shape`

> `make_shape` creates a shape that describes the size of a N-dimensional tensor.

Takes N number of arguments to create a N-dimensional shape.
Each dimension is called a **mode**. 
The mode-0 or the left-most dimension is the leading dimension and progressively strided over subsequent dimensions

```cpp
auto tile_shape = cute::make_shape(64, 128); //< create a 128x64 tile
auto tensor_shape = cute::make_shape(1024, 1024); //< creates shape for a tensor
auto image_shape = cute::make_shape(c, w, h, n); //< creates shape for n images
```

### `make_stride`

> `make_stride` describes how striding is done for a given shape

