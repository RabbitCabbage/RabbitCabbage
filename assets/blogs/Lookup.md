<script type="text/javascript" charset="utf-8" 
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML,
https://vincenttam.github.io/javascripts/MathJaxLocal.js">
</script>

# Lasso Lookup
What is a lookup argument? Briefly speaking, it's a cryptographic proof that a value (or some values) is in a table. Formally, a (non-interactive) lookup argument is a SNARK, for the prover to claim that it knows an opening of a commitment, and that the openning set is in a public table of field elements. The table can be provided to the verifier as a commitment for verification. 
The line of works (Caulk, Caulk+, flookup, Baloo and cq) prefer the commitment of the table and lookups to be additively homomorphic. 

They require a structured reference string of the table size as pre-processing ($O(N\log N)$ group exponentiations).

## Technical Details
Verifier has commitment to table $t\in \mathbb{F}^n$ and commitment to vector $a\in \mathbb{F}^m$. Prover want to prove that $a$ is in the table $t$. So the prover proves knowledge of a sparse matrix $M\in \mathbb{F}^{m\times n}$ such that 

$$Mt=a$$

Then we change it to multilinear extension polynomials:

$$Mt=a\Longleftrightarrow \sum_{y\in \{0,1\}^{\log n}} \widetilde{M}(r,y)\cdot \widetilde{t}(y)=\widetilde{a}(r)$$

for a random $r\in \mathbb{F}^{\log n}$ chosen by the verifier. Intuitively, if $r$ is chosen binary, then it refers to specific row of the matrix $M$, thus looking up a particular element in table $t$.

![from matrix-vector to multilinear extension](/assets/blogs/mt_a_extension.png)

**Plain sumcheck**: sum up all hypercube $\{0,1\}^{\log n}$, but it's too expensive. 
**Improvement intuition**: We only sum up over lookups, rather than all the table entries (the whoe hypercube).
**Technique**: represent the sparse polynomial with dense polynomials.

> For example we have a $m$-sparse vector $D\in \mathbb F^N$ (thus a sparse polynomial), and we want to evaluate it at a random point $r\in \mathbb \{0,1\}^{\log N}$ (i.e. access the $r$-th element of $D$).
>
> $$D(r)=\sum_{j\in\{0,1\}^{\log N}:D_j\neq 0} D(j)\cdot \widetilde{eq}(r,j)$$
>
>and there exists a $\log m$-variate multilinear polynomials $idx$ and $val$ which remaps the indices for non-zero entries:
>
> $$D(r)=\sum_{k\in\{0,1\}^{\log m}} val(k)\cdot \widetilde{eq}(r,idx(k))$$
>
> also this can be extended to a matrix or tensor, as it does in Lasso's decomposed tables

Thus we get a dense version of the above equation:

$$\sum_{y\in \{0,1\}^{\log n}} \widetilde{M}(r,y)\cdot \widetilde{t}(y)=\widetilde{a}(r)\Longleftrightarrow \sum_{k\in\{0,1\}^{\log m}} \widetilde{eq}(r,{idx}(k))\cdot \widetilde{t}({idx}(k))=\widetilde{a}(r)$$

Then for large tables, Lasso puts forward a decomposible structure which can be writen as a tensor product of smaller tables. Then Surge/Spark can be used for this structured large tables/smaller tables.

> Here is formmally how the Spark-only structure (SOS) is defined. The vector $t$ of size $N$ is the tensor product of smaller vectors $t_1,t_2,\cdots,t_c$ each of size $N^{1/c}$. And the index $r$ (or evaluation point) is also decomposed $r=(r_1,r_2,\cdots,r_c)\in \left(\mathbb F^{\log N^{1/c}}\right)^c$.

![large structured lookup table](/assets/blogs/whole_sos_lookup.png)

![decomposed small lookup tables](/assets/blogs/decomposed_tables.png)

> One possible requirement for the decomposed tensor: no collision in the decomposed smaller tables. (?)

Therefore we have $c$ re-mapping for the decomposed index $r$, so the decomposed tables have the equation:

$$\sum_{k\in\{0,1\}^{\log m}} \left(\prod_{i=1}^c \widetilde{eq}(r_i,idx_i(k))\right)\cdot g\left(\widetilde{t_1}\left({idx_1}(k)\right), \widetilde{t_2}\left({idx_2}(k)\right),\cdots,\widetilde{t_c}\left({idx_c}(k)\right)\right)=\widetilde{a}(r)$$

where $g$ is the multilinear polynomial for the tensor product of the smaller tables.
For simplicity we use the notation $E(r,k)=\widetilde{eq}(r,idx(k)),\ T_i(k)=\widetilde{t_i}\left({idx_i}(k)\right)$ in the following.

$$Mt = a \Longleftrightarrow \sum_{k\in\{0,1\}^{\log m}} \left(\prod_{i=1}^c E(r_i,k)\right)\cdot g(T_1(k),T_2(k),\cdots,T_c(k))=\widetilde{a}(r)$$

In Lasso, the prover commits to the matrix $M$ **using sparse polynomial commitment Spark**, and proves the equation using **Surge/Spark**. But any multilinear PCS can be used.