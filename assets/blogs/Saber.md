<script type="text/javascript" charset="utf-8" 
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML,
https://vincenttam.github.io/javascripts/MathJaxLocal.js">
</script>


## Takeaway of comparison
Saber can be viewed as an aggressive optimization of Kyber, changing all random errors to deterministic rounding error. 

Kyber only skips one random error using LWR assumption (argued in their "Security of the real scheme" in section 3). And they use rounding (compression) just to decrease bandwidth. Saber goes further by changing all random errors to deterministic rounding error, killing both noise and compression with one stone.

Therefore, Kyber bases their security mainly on Mod-LWE, they can even maintain a reasonable security assume LWR assumption is false. Saber bases their security **solely** on LWR assumption. Therefore, NIST prefers Kyber which adds a "redundant" random noises, with only a little sacrifice of performance.

[Is Kyber redundant in its design compared to Saber?](https://crypto.stackexchange.com/q/112841/113874)

## SABER Key Exchange
**Notations**: $R_q:=\mathbb Z_q[X]/(X^n+1)$, with $n=256$ or a fixed power of 2. $\beta_\mu$ is a centered binomial distribution with parameter $\mu$ and corresponding standard deviation $\sigma=\sqrt{\frac \mu 2}$.

And define $\text{Bits}(x,i,j)$ as the function $(x\gg (i-j+1))\&(2^j -1)$.

![Bits function, get lower $j$ bits from the $i$-th position.](/assets/blogs/bits_saber.png)

Then the Saber key exchange is defined as follows:
![Saber key exchange](/assets/blogs/ke_saber.png)

## Security of KE
Here is the standard form of LWR, generating the noise deterministically by scaling and rounding coefficients modulo $q$ to modulo $p$ (with $p < q$):
$$\left( \mathbf a, b=\lfloor \dfrac p q(\mathbb a^\top \mathbf s)\rceil \right)\in \mathbb Z_q^{\ell\times 1}\times \mathbb Z_p$$

where $\mathbf s\leftarrow \beta_\mu(\mathbb Z_q^\ell)$, $\mathbb a\leftarrow \mathcal U(\mathbb Z_q^{\ell\times 1})$. Decisional version says it is hard to distinguish a LWR instance from a uniform one.

Mod-LWR:
$$\left( \mathbf a, b=\lfloor \dfrac p q(\mathbb a^\top \mathbf s)\rceil \right)\in R_q^{\ell\times 1}\times R_p$$

## Kyber IND-CPA Encryption

## Security of Kyber

## Fujisaki-Okamoto Transform