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

Once a shape is created, you want to provide a different stride than what is implicitly derived. For example, if the shape is (64, 128), the default stride is (1, 64). Which means, 64 is along the "leading" dimension. But if I want to represent the shape as (128, 64) and have the same behavior as previously, I have to explicitly describe strides at each mode.

```cpp
auto tile_shape = cute::make_shape(128, 64);
/// Creates stride where mode-1 of tile_shape is the leading dimension and mode-0 is the strided dimension
auto tile_stride = cute::make_stride(64, 1);
```

### `make_layout`

> `make_layout` holds information about the layout of a tensor

Creating shape and stride may not be much useful when used separately. Shape doesn't provide information about how the data is laid out in memory or in some physical space or Stride doesn't provide information about the extents of the tensor.

But, when used together in `make_layout` it becomes a powerful tool to describe any tensor and how its laid out in memory or physical space

```cpp
int m = 1024, k = 512;
auto tensor_shape_a = cute::make_shape(m, k);

/// shape: (m, k), stride: (1, m). M is leading for A, meaning Column-Major Layout
auto tensor_layout_a = cute::make_layout(tensor_shape_a);

/// shape: (m, k), stride: (k, 1), K is leading for A, meaning Row-Major Layout
auto tensor_layout_a_t = cute::make_layout(tensor_shape_a, cute::make_stride(k, 1));
```

### `make_tensor`

> `make_tensor` creates a tensor from a pointer

Creates a tensor from raw pointer

```cpp
using namespace cute;
int m = 1024, k = 512;

/// Create Row-Major A matrix cute::Tensor
Tensor mAT = make_tensor(make_gmem_ptr(a_ptr), make_shape(m, k), make_stride(k, 1));
/// Create Col-Major A matrix cute::Tensor
Tensor mA = make_tensor(make_gmem_ptr(a_ptr), make_shape(m, k), make_stride(1, m));
```

### `make_coord`

> `make_coord` creates a coordinate to access

Create a coordinate

```cpp
auto cta_coord = cute::make_coord(blockIdx.x, blockIdx.y, _);
```

### `logical_divide`

> `logical_divide` divides a tensor with a tiler and the number of tiles for each mode are at the end of the divided tile

```cpp
auto tensor_shape = tensor.shape(); //< lets say (1024, 256)
auto tile_shape = tile.shape(); //< lets say (16, 32)

auto logical_divide_tile = cute::logical_divide(tensor, tile);
```

### `zipped_divide`

> `zipped_divide` divides a tensor with a tiler and the number of tiles for each mode are at the end of the divided tile packed as a single mode.

```cpp
auto tensor_shape = tensor.shape(); //< lets say (1024, 256)
auto tile_shape = tile.shape(); //< lets say (16, 32)

/// shape is ((16, 32), (64, 8));
auto zipped_divide_tile = cute::zipped_divide(tensor, tile);
```

### `tiled_divide`

> `tiled_divide` divides a tensor with a tile and the number of tiles for each mode are at the end of the divided tile and each of them will be a new mode

```cpp
auto tensor_shape = tensor.shape(); //< lets say (1024, 256)
auto tile_shape = tile.shape(); //< lets say (16, 32)

/// shape is ((16, 32), 64, 8);
auto tiled_divide_tile = cute::tiled_divide(tensor, tile);
```

### `flat_divide`

> `flat_divide` divides a tensor with a tile and the number of tiles for each mode are at the end of the divided tile and each of them will be a new mode along with all the tile modes.

```cpp
auto tensor_shape = tensor.shape(); //< lets say (1024, 256)
auto tile_shape = tile.shape(); //< lets say (16, 32)

/// shape is (16, 32, 64, 8);
auto flat_divide_tile = cute::flat_divide(tensor, tile);
```

### `local_tile`

> `local_tile` 

### `compose`


### `tile`

### `get_1d_coord`

### `get_hier_coord`

### `get_flat_coord`

### `layout(tensor)`

> Returns layout of a mode

```cpp
```

### `shape(tensor)`

> Returns shape of a mode

```cpp
```

### `stride(tensor)`

> Returns stride of a mode

```cpp
```

### `size(tensor)`

> Returns the number of elements in a mode

```cpp
```

### `rank(tensor)`

> Returns the rank of a mode

Rank is the number of dimensions of a mode

```cpp
auto s0 = 100;
auto s1 = make_shape(11, 12); //< (11, 12)
auto s2 = make_shape(13, 14, 15); //< (13, 14, 15)
auto s3 = make_shape(s1, s2); //< ((11, 12), (13, 14, 15))
auto s = make_shape(s0, s3); //< (100, ((11, 12), (13, 14, 15)))

auto rank_0 = rank(s, 0); //< rank_0 = 1, rank((100));
auto rank_1 = rank(s, 1); //< rank_1 = rank(s3) = 2, rank((11, 12), (13, 14, 15));
auto rank_2 = rank(s2); //< rank_2 = 3, rank((13, 14, 15))
```

### `depth(tensor)`

> Returns the depth of a mode

Returns the number of levels of sub-tiles/sub-shapes inside a tensor

```cpp
auto s0 = 100;
auto s1 = make_shape(11, 12);
auto s2 = make_shape(13, 14, 15);
auto s3 = make_shape(s1, s2);
auto s = make_shape(s0, s3);

auto depth_ = depth(s); //< depth_ = 3
auto depth_0 = depth(s3); //< depth_0 = 2
auto depth_1 = depth(s1); //< depth_1 = 1
```

### `flatten(tensor)`

> Returns 

### `local_tile(tensor, tiler, coord, proj)`

It returns a tile after diving the `tensor` into multiple `tiler`s at `coord`s. The `proj` is used to select different modes in the `coord` and `tiler`.

```cpp
//< ltile is (32,4):(1,64)
auto ltile = local_tile(
  tensor, //< (64,16):(1:64)
  tiler, //< (32,4)
  coord, //< = (blockIdx.x, blockIdx.y)
  proj
);
```

### `local_partition(tensor, layout, index)`

It returns a tile after dividing the `tensor` into sub-tiles of `layout` shapes and assign each of these sub-tiles to `index`

```cpp
/// lpart is (8,2):(4:128)
auto lpart = local_partition(
  tensor, //< (32,4):(1:64)
  layout, //< (4,2):(1,64)
  index); //< = threadIdx.x
```

