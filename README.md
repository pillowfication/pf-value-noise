# pf-value-noise

**N-Dimensional Value Noise Generator** - A [value noise](https://en.wikipedia.org/wiki/Value_noise) generator for any number of dimensions. Similar to, but faster than [Perlin noise](https://en.wikipedia.org/wiki/Perlin_noise).

![Rainbow Value Noise](/rainbow-value-noise.png)

## Examples

```javascript
// Require the module to use it.
const ValueNoise = require('pf-value-noise')

// Create a 3D value noise generator.
const noise3D = new ValueNoise({ dimensions: 3 })

// Use it to make a 100×100×100 grid of values
const resolution = 100
let data = []
for (let x = 0; x < resolution; ++x) {
  for (let y = 0; y < resolution; ++y) {
    for (let z = 0; z < resolution; ++z) {
      data.push(noise3D.get([ x / resolution, y / resolution, z / resolution ]))
    }
  }
}

const _ = require('lodash')
data = _(data).chunk(resolution).chunk(resolution).value()
data[5][62][17]
// 0.6594545530358533
```

The following example creates the above picture.

```javascript
// Create the canvas
const { createCanvas } = require('canvas')
const [ width, height ] = [ 800, 200 ]
const canvas = createCanvas(width, height)
const ctx = canvas.getContext('2d', { alpha: false })

// Create the image data
const ValueNoise = require('pf-value-noise')
const noise3D = new ValueNoise({ dimensions: 3, seed: 'pillow' })
const resolution = 100
const imageData = ctx.createImageData(width, height)
let dataIndex = 0
for (let row = 0; row < height; ++row) {
  for (let col = 0; col < width; ++col) {
    imageData.data[dataIndex++] = noise3D.get([ row / resolution, col / resolution, 0 ]) * 256 | 0
    imageData.data[dataIndex++] = noise3D.get([ row / resolution, col / resolution, 1 ]) * 256 | 0
    imageData.data[dataIndex++] = noise3D.get([ row / resolution, col / resolution, 2 ]) * 256 | 0
    ++dataIndex
  }
}

// Export the image data
const fs = require('fs')
ctx.putImageData(imageData, 0, 0)
canvas.createPNGStream()
  .pipe(fs.createWriteStream('rainbow-value-noise.png'))
```

## API

### `ValueNoise`

*({Class})*: Represents a value noise generator.

```javascript
const ValueNoise = require('pf-value-noise')
const noiseGenerator = new ValueNoise()
```

### `ValueNoise.constructor([options])`

**Arguments**
 1. `[options]` *(Object)*: An objects of options.

|  Option         | Type     | Default  | Description                    |
|:---------------:|:--------:|:--------:|:-------------------------------|
| `seed`          | String   | `null`   | RNG's seed                     |
| `dimensions`    | Number   | `2`      | Number of dimensions           |
| `min`           | Number   | `0`      | Minimum value returned         |
| `max`           | Number   | `1`      | Maximum value returned         |
| `wavelength`    | Number   | `1`      | Size of the first octave       |
| `octaves`       | Number   | `8`      | Number of octaves to sample    |
| `octaveScale`   | Number   | `1/2`    | Scaling for successive octaves |
| `persistence`   | Number   | `1/2`    | Weight for successive octaves  |
| `interpolation` | Function | *cosine* | Interpolation function used    |

Note that even with the same seed, a different order of `<Perlin>.get()` calls can change the overall noise function since its values are generated lazily.

`wavelength` sets the size of the first octave, and each successive octave will be `octaveScale` times the previous. The octaves are centered about the origin and added together according to their weight. The first octave has a weight of `1`, and each successive octave will be `persistence` times the previous.

The octaves are sampled using the `interpolation` function with signature `function(a, b, t)` that returns a value between `a` and `b` according to the parameter `0 <= t <= 1`. The default interpolation function used is cosine interpolation.

```javascript
interpolation: function (a, b, t) {
  return (1 - Math.cos(Math.PI * t)) / 2 * (b - a) + a
}
```

After the octaves are sampled and added together, the values are adjusted to fall between `min` and `max`. Note that the value distribution is roughly Gaussian depending on the number of octaves.

### `ValueNoise.prototype.get(coordinates)`

**Arguments**
 1. `coordinates` *(Array<Number>)*: The data point to get. Its length should match `dimensions`.

**Returns**
 * *(Number)*: The value at those coordinates.

```javascript
const noise4D = new ValueNoise({ dimensions: 4 })

noise4D.get([ 1, 2, 3, 4 ])
// 0.538503118881535
```
