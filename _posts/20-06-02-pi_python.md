---
title: "pi.py"
image: "assets/images/posts/pi_20-06_monte-carlo.png"
latex: "yesPlz"
---

My concern with attempting to keep something resembling a blog is that it will inevitably die due to neglect. The obvious solution is to have something simple that obliges me to make some small regular contribution, no less often than once a month. I'm going with programatically calculating <script markdown="0" type="math/tex">\pi</script>. Once a month, always in a different way, always with a different language. This will start fairly trivially, though I envision it getting fun.

I'll begin with Python, and one of the most intuitive methods of calculating <script markdown="0" type="math/tex">\pi</script>: a basic monte-carlo estimate. The intuition is illustrated with the graphic above, draw a quarter circle of radius 1 inside a square of side length 1. Then start dropping points at random inside the square. The probability of a point falling inside the quarter circle is <script markdown="0" type="math/tex">\pi/4</script>. Therefore if we drop a large number of points <script markdown="0" type="math/tex">\pi</script>, we would expect <script markdown="0" type="math/tex">n\pi/4</script> points to fall within the circle, and <script markdown="0" type="math/tex">n(1-\pi/4)</script> to fall beyond it.

Given a point <script markdown="0" type="math/tex">p</script> with coordinates <script markdown="0" type="math/tex">(x,y)</script> on the unit square:
<div markdown="0" style="text-align:center">{%raw%}$$
\left\lfloor x^2+y^2 \right\rfloor = \begin{cases}
    0\textrm{ if $p$ is inside the circle} \\
    1\textrm{ if $p$ is beyond the circle}
\end{cases}
$${%endraw%}</div>
Therefore if we choose a large enough <script markdown="0" type="math/tex">n</script>:
<div markdown="0" style="text-align:center">{%raw%}$$
\sum_{i=1}^n \left\lfloor x_i^2+y_i^2 \right\rfloor \approx  n\left(1-\frac{\pi}{4}\right)
$${%endraw%}</div>
giving us an expression for {%raw%}$$ \pi $${%endraw%} of:
<div markdown="0" style="text-align:center">{%raw%}$$
\pi \approx  4-4\frac{\sum_{i=1}^n \left\lfloor x_i^2+y_i^2 \right\rfloor}{n}
$${%endraw%}</div>
Thanks to some nice vectorisation from <code>numpy</code> this can be written very succinctly:
<pre><code>from numpy import floor as f, random as r
n = 2**27
print(4-4*sum(f(r.random(n)**2+r.random(n)**2))/n)
</code></pre>
This little block runs in 10 seconds or so on my laptop and consistently gives <script markdown="0" type="math/tex">\pi</script> accurate to three decimal places. Not too shabby for the most simplistic method available.