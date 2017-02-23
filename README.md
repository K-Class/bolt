
<!-- ![Bolt](/assets/bolt.jpg?raw=true =489x177) -->

<p align="center">
  <img src="https://github.com/dblalock/bolt/blob/master/assets/bolt.jpg?raw=true" alt="Bolt" style="width=489px;height:177px;"/>
</p>

Bolt is an algorithm for compressing vectors of real-valued data and running mathematical operations directly on the compressed representations.

If you have a large collection of vectors and can tolerate lossy compression, Bolt can probably save you 10-200x space. And if what you want to do with these vectors is compute dot products and/or Euclidean distances to other vectors, Bolt can probably save you 10-200x compute time.

NOTE: this project page is currenlty under construction but will soon include all the code, documentation, and results to reproduce Bolt's KDD paper.


## Example: Matrix-Matrix and Matrix-Vector multiplies

```python
import bolt
import numpy as np

X = np.random.randn(1e6, 256)  # dataset
Q = np.random.randn(1e3, 256)  # queries; need dot products with rows in X

# train a Bolt encoder on this distribution
encoder = bolt.Encoder().fit(X)

# obvious way to get dot products
out = np.dot(X, Q.T)

# obvious way to get dot products if we (hypothetically) got queries one at a time
for i, q in enumerate(Q):
    out[i] = np.dot(X, q)

# faster way: use Bolt approximate dot products
out2 = encoder.dot(X, Q.T)

# even faster way: give Bolt an X that's already compressed
X_compressed = encoder.transform(X)
out3 = encoder.dot(X_compressed, Q.T)

# massive space savings
print(X.nbytes) # 1e6 * 256 * 8B = 2.048 GB
print(X_compressed.nbytes)  # 1e6 * 32B = 32 MB

# nearly identical dot products (modulo offset/scaling)
from scipy.stats.stats import pearsonr
print("correlation between true and Bolt dot products: " {}".format(
    pearsonr(out, out3))  # >.9, despite random data being worst case

# recover offset/scaling if needed
out3 = encoder.rescale_output(out3)
```

## How does it work?

Bolt is based on [vector quantization](https://en.wikipedia.org/wiki/Vector_quantization). For details, see the Bolt paper (to be uploaded soon...).

## Benchmarks

Bolt includes a thorough set of performance benchmarks.