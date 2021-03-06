## Self-Normalizing Neural Networks

**Authors**: *Günter Klambauer, Thomas Unterthiner, Andreas Mayr, Sepp Hochreiter*

**Gist**: The authors introduce self-normalizing neural networks (SNNs) whose layer activations automatically converge towards zero mean and unit variance and are robust to noise and perturbations. 

**Significance**: Removes the need for the finicky [batch normalization](https://arxiv.org/abs/1502.03167) and  permits training deeper networks with a robust training scheme.

## Picture Says It All

<p align="center">
 <img src="/img/self_norm/loss.png" alt="Drawing">
</p>

## Activation Function

Uses **Scaled Exponential Linear Units** or SELUs which are defined as follows:

<p align="center">
 <img src="/img/self_norm/eq.png" alt="Drawing" width="300px">
</p>

For the case where we would like zero mean and unit variance, `alpha = 1.6732` and `scale = 1.0507`.

**Properties**

- negative and positive values to control the mean
- derivatives approaching 0 to dampen variance
- slope larger than 1 to increase variance
- continuous curve

```python
def selu(x):
	alpha = 1.6732632423543772848170429916717
	lamb = 1.0507009873554804934193349852946
	return lamb * np.where(x > 0., x, alpha * np.exp(x) - alpha)
```

## Weight Initialization

Draw the weights from a Gaussian distribution with mean 0 and variance variance = `1/N`.

In python, this is equivalent to doing the following:

```python
mu = 0 
sigma = np.sqrt(1.0 / N)
W = np.random.normal(mu, sigma, N)
```

## Alpha Dropout

New variant designed for SELU activation. Randomly sets inputs to `alpha_drop = alpha * lamb` then performs an affine transformation  with parameters a and b that preserve the self-normalizing property of the activations.

Remember to use the `lambda` and `alpha` values corresponding to zero mean and unit variance. On the other hand, the parameters of the affine transformation can be determined as follows:

```python
# dropout params
keep = 0.95
q = 1 - keep

# selu params
alpha = 1.6732632423543772848170429916717
lamb = 1.0507009873554804934193349852946
alpha_p = - alpha * lamb

# affine trans params
prod = q + np.power(alpha_p, 2)*q*(1-q)
a = np.power(prod, -0.5)
b = -a * (alpha_p*(1-q))
```

And here's a quick NN implementation:

```python
def alpha_drop(x, alpha_p=-1.758, keep=0.95):	
  # create mask
  idx = np.random.rand(*x.shape) < keep

  # apply mask
  x[~idx] = alpha_p

  # apply affine transformation (using a and b from before)
  out = x*a + b

  return out
	
# example
H = selu(np.dot(W, X) + b)
H_drop = alpha_drop(H)
```
The authors report that dropout rates `1 - q = 0.05` and `1 - q = 0.1` work well.
