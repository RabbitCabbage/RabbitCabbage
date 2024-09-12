<script type="text/javascript" charset="utf-8" 
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML,
https://vincenttam.github.io/javascripts/MathJaxLocal.js">
</script>

# Lasso Lookup

## Brief view of lookup arguments
What is a lookup argument? Briefly speaking, it's a cryptographic proof that a value (or some values) is in a table. Formally, a (non-interactive) lookup argument is a SNARK, for the prover to claim that it knows an opening of a commitment, and that the openning set is in a public table of field elements. The table can be provided to the verifier as a commitment for verification. 
The main goal of new lookups is to decrease the prover time. Plookup bounds the prover time to the table size, and Caulk/Caulk+ can prove $m$ lookups in more reasonable $O(m^2)$, making it possible for large tables but inconvenient for mass lookups. Then Baloo achieves quasilinear prover in the field and linear in the group.

And the line of works (Caulk, Caulk+, flookup, Baloo and cq) prefer the commitment of the table and lookups to be additively homomorphic. But they require a structured reference string of the table size as pre-processing ($O(N\log N)$ group exponentiations).

## The goal of Lasso
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

> Here is formmally how the Spark-only structure (SOS) is defined. The vector $t$ of size $N$ is the tensor product of smaller vectors $t_1,t_2,\cdots,t_c$ each of size $N^{1/c}$. And the index $r$ **(where the entry is non-zero)** is also decomposed $r=(r_1,r_2,\cdots,r_c)\in \left(\mathbb F^{\log N^{1/c}}\right)^c$.

![large structured lookup table](/assets/blogs/whole_sos_lookup.png)

![decomposed small lookup tables](/assets/blogs/decomposed_tables.png)

> One possible requirement for the decomposed tensor: no collision in the decomposed smaller tables. (?)

Therefore we have $c$ re-mapping for the decomposed index $r$, so the decomposed tables have the equation:

$$\sum_{k\in\{0,1\}^{\log m}} \left(\prod_{i=1}^c \widetilde{eq}(r_i,idx_i(k))\right)\cdot g\left(\widetilde{t_1}\left({idx_1}(k)\right), \widetilde{t_2}\left({idx_2}(k)\right),\cdots,\widetilde{t_c}\left({idx_c}(k)\right)\right)=\widetilde{a}(r)$$

where $g$ is the multilinear polynomial for the tensor product of the smaller tables.
For simplicity we use the notation $E_i(k)=\widetilde{eq}(r_i,idx_i(k)),\ T_i(k)=\widetilde{t_i}\left({idx_i}(k)\right)$ in the following.

$$Mt = a \Longleftrightarrow \sum_{k\in\{0,1\}^{\log m}} \left(\prod_{i=1}^c E_i(k)\right)\cdot g(T_1(k),T_2(k),\cdots,T_c(k))=\widetilde{a}(r)$$

So this form is the final form of the goal of Lasso lookup argument.

## Steps of Lasso

Therefore, the goal of Lasso is to look up indices $M$ in table $t$ to get $a$, thus we have a polynomial IOP. The oracles for verifier are $c$ ($\log m$)-variate multilinear polynomials $E_i(k)$, which are supposed to be the multilinear extension of the index mapping of each subtables.

More detailed steps of Lasso:

1. **Commitment:** Prover sends Surge commitment to the vector, including table values, indices and some counters for memory checking.
2. **Primary Sumcheck:** The two parties run a sumcheck protocol to reduce the check 

    $$\sum_{k\in\{0,1\}^{\log m}} \left(\prod_{i=1}^c E_i(k)\right)\cdot g(T_1(k),T_2(k),\cdots,T_c(k))\stackrel{?}{=}\widetilde{a}(r)$$
    
    to equations 
    
    $$\begin{aligned}
   t(x)&\stackrel{?}{=} g(T_1(x_1),T_2(x_2),\dots,T_c(x_c))\\
    \{\text{index of $x$ in $T_i$}\}&\stackrel{?}{=} E_i(x_i)\quad \text{(TODO in step 3)}
    \end{aligned}$$
    
    where the verifier chooses random $x\in \mathbb F^{\log N^{1/c}}$ as challenges, and get $\widetilde{a}(x)$ by one evaluation to $\widetilde{a}$.
3. **Memory Checking**: Prover and verifier run Surge to prove the equation above, and use memory checking to ensure the integrity of the IOP oracle $E_i$.

Here is some intuition for the memory checking. Lasso treaks a little the original version, changing from a *global timestamp* to a *local counter for each cell*, getting rid of the need for a trusted checker.

1. Init: initialize the table into the memory, indices as addresses, table values as data, and counters set to 0. Construct an initial set $I_i$ for each subtable $t_i$.
2. Read & Write: Every $E_i:=\widetilde{eq}(r_i,idx_i(k))$ oracle needs a memory access to $r_i$ in table $t_i$, thus there is a "read" operation. And after one cell is accessed, the counter of the cell is increased by 1, which is the "write" operation. After every operation, the `(addr, data, counter)` tuple is added to a corresponding set: $R_i$ for read, $W_i$ for write 
3. Final: Get final configuration set $F_i$ Prove that $I\cap W=R\cap F$ (size of $R_i$ and $W_i$ is $m$, small).

Intuitively, the counter in tuples ensures that every memory read over the course of an algorithm’s execution returns the value last written to that location, thus the memory checking guarantines honest oracle $E_i$.

In Lasso, the prover commits to the matrix $M$ using sparse polynomial commitment Spark, and proves the equation using Surge/Spark. But any multilinear PCS can be used.