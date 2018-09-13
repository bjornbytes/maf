maf
===

A small 3D math library for Lua.

**Features**

- Single file
- Uses optimized LuaJIT structs if available
- Supports vectors and quaternions
- Provides an ugly API that is garbage-free as well as a pretty API that generates garbage.

Check out [cpml](https://github.com/excessive/cpml) if you're looking for something with more features.

Example
---

```lua
local maf = require 'maf'

local a = maf.vector(1, 2, 3)
local b = maf.vector(4, 5, 6)

local c = a + b
local d = maf.rotation():between(a, b)

print(c:unpack())
print(d:getAngleAxis())
```

Documentation
---

In general, any function that returns a vector or a rotation will write its result into the last
parameter, `out`.  The function also returns `out` for convenience, to allow for method chaining.
If `out` is unspecified, then the result will be written into the first parameter instead.

### Vectors

##### `maf.vector(x, y, z)` or `maf.vec3(x, y, z)`

Create a new vector.  Omitted elements will be set to zero.

The `x`, `y`, and `z` keys contain the individual components of the vector.

The following metamethods are defined for vectors:

- `v + v` - Add two vectors.
- `v - v` - Subtract two vectors.
- `v * v` - Multiply two vectors.
- `v * n` - Scale a vector by a number.
- `v / v` - Divide two vectors.
- `-v` - Negate a vector.
- `#v` - Get the length of a vector.

##### `vector:unpack()`

Get the individual components of the vector.

##### `vector:set(x, y, z)`

Set the individual components of a vector.  `x` can also be a vector.

##### `vector:add(v2, [out])`

Add two vectors.

##### `vector:sub(v2, [out])`

Subtract two vectors.

##### `vector:mul(v2, [out])`

Multiply two vectors.

##### `vector:div(v2, [out])`

Divide two vectors.

##### `vector:scale(s, [out])`

Multiply each component of the vector by a number `s`.

##### `vector:length()`

Return the length of the vector.

##### `vector:normalize([out])`

Set the length of a vector to 1.

##### `vector:distance(v2)`

Return the distance between two vectors.

##### `vector:angle(v2)`

Return the angle between two vectors, in radians.

##### `vector:dot(v2)`

Return the dot product of two vectors, defined as `v1.x * v2.x + v1.y * v2.y + v1.z * v2.z`.

##### `vector:cross(v2, [out])`

Return the cross product of two vectors.  The cross product is a vector that is perpendicular to the
two input vectors.

##### `vector:lerp(v2, t, [out])`

Mixes two vectors together using linear interpolation.  The parameter `t` controls how they are
mixed (0 will return `vector`, 1 will return `v2`, .5 will return a 50/50 blend of the two vectors,
etc.).

##### `vector:project(u, [out])`

Projects `vector` onto `u`, storing the result in `out`.

##### `vector:rotate(r, [out])`

Applies a rotation `r` to the vector.

### Rotations

##### `maf.rotation(x, y, z, w)` or `maf.quat(x, y, z, w)`

Creates a new rotation (quaternion) from x, y, z, w components.  Unless you are some sort of wizard
that understands quaternions, it's probably easier to call `:angleAxis` or `:between` on the
rotation to initialize it.

The following metamethods are defined for rotations:

- `q + q` - Add two rotations.
- `q - q` - Subtract two rotations.
- `q * q` - Multiply two rotations, resulting in a rotation that is equivalent to applying the first
  rotation followed by the second.
- `q * v` - Multiply a rotation by a vector, returning the rotated vector.
- `-q` - Negate a rotation.
- `#q` - Get the length of a rotation.

##### `rotation:unpack()`

Get the individual components of the rotation.

##### `rotation:set(x, y, z, w)`

Set the individual components of a rotation.  `x` can also be a rotation.

##### `angle, x, y, z = rotation:getAngleAxis()`

Get the angle/axis representation of the rotation.

##### `rotation:angleAxis(angle, vector)` or `rotation:angleAxis(angle, x, y, z)`

Set the rotation using angle/axis representation.

##### `rotation:between(v1, v2)`

Set the rotation defined by rotating from `v1` to `v2`.

##### `rotation:add(r2, [out])`

Add two rotations together.  To combine two rotations together, multiply them instead.

##### `rotation:sub(r2, [out])`

Subtract two rotations.

##### `rotation:mul(r2, [out])`

Multiply two rotations together, returning a rotation that combines the two.

##### `rotation:scale(s, [out])`

Multiply each component of the rotation by a number `s`.

##### `rotation:length()`

Return the length of the rotation.

##### `rotation:normalize([out])`

Normalize a rotation, setting its length to 1.

##### `rotation:lerp(r2, t, [out])`

Mix two rotations together using linear interpolation.  This is faster than `:slerp` but less
accurate.

##### `rotation:slerp(r2, t, [out])`

Mix two rotations together using spherical linear interpolation.

Garbage Collection Is Scary
---

Keep in mind that each time a new vector or rotation is created, a small memory allocation is made.
Normally you don't need to worry about this, since Lua has a garbage collector that will clean up
after you.  However, allocating a large number of objects can put stress on the garbage collector,
which can lead to performance problems.  This can happen more often that you'd think if you're, for
example, creating new vectors in the update loop of a game.

It gets worse.  Although it's convenient to be able to write `a + b` to add vectors together, a new
vector must be allocated to store the result.  This makes it really easy to allocate a bunch of
vectors without even knowing it!  To avoid this sneaky performance trap, you can add two vectors
using `a:add(b)`, which will reuse `a` to store the result.  Another strategy is to allocate a set
of reusable temporary vectors and use `a:add(b, tmp)`.

License
---

MIT, see [`LICENSE`](LICENSE) for details.
