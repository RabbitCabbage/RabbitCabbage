<script type="text/javascript" charset="utf-8" 
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML,
https://vincenttam.github.io/javascripts/MathJaxLocal.js">
</script>


Take away:
Saber can be viewed as an aggressive optimization of Kyber, changing all random errors to deterministic rounding error. 

Kyber only skips one random error using LWR assumption (argued in their "Security of the real scheme"), in other rounding (compression) to decrease bandwidth. Saber goes further by changing all random errors to deterministic rounding error.