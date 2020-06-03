---
title: "pi.py"
image: "assets/images/posts/pi_20-06_monte-carlo.png"
---

My concern with attempting to keep something resembling a blog is that it will inevitably die due to neglect. The obvious solution is to have something simple that obliges me to make some small regular contribution, no less often than once a month. I'm going with programatically calculating {%raw%}$$ \pi $${%endraw%}. Once a month, always in a different way, always with a different language. This will start fairly trivially, though I envision it getting fun.

I'll begin with Python, and one of the most intuitive methods of calculating {%raw%}$$ \pi $${%endraw%}: a basic monte-carlo estimate. The intuition is illustrated with the graphic above, draw a quarter circle of radius 1 inside a square of side length 1. Then start dropping points at random inside the square. The probability of a point falling inside the quarter circle is {%raw%}$$\pi/4$${%endraw%}. Therefore if we drop a large number of points {%raw%}$$ n $${%endraw%}, we would expect {%raw%}$$ n\pi/4 $${%endraw%} points to fall within the circle, and {%raw%}$$ n(1-\pi/4) $${%endraw%} to fall beyond it.

Given a point {%raw%}$$ p $${%endraw%} with coordinates {%raw%}$$ (x,y) $${%endraw%} on the unit square:
<div text-align: center>
{%raw%}$$
\left\lfloor x^2+y^2 \right\rfloor = \begin{cases}
    0\textrm{ if $p$ is inside the circle} \\
    1\textrm{ if $p$ is beyond the circle}
\end{cases}
$${%endraw%}
</div>
Therefore if we choose a large enough {%raw%}$$ n $${%endraw%}:
<div text-align: center>
{%raw%}$$
\sum_{i=1}^n \left\lfloor x_i^2+y_i^2 \right\rfloor \approx  n\left(1-\frac{\pi}{4}\right)
$${%endraw%}
</div>
giving us an expression for {%raw%}$$ \pi $${%endraw%} of:
<div text-align: center>
{%raw%}$$
\pi \approx  4-4\frac{\sum_{i=1}^n \left\lfloor x_i^2+y_i^2 \right\rfloor}{n}
$${%endraw%}
</div>
Thanks to some nice vectorisation from <code>numpy</code> this can be written very succinctly:
<pre><code>from numpy import floor as f, random as r
n = 2**27
print(4-4*sum(f(r.random(n)**2+r.random(n)**2))/n)
</code></pre>
This little block runs in 10 seconds or so on my laptop and consistently gives {%raw%}$$ \pi $${%endraw%} accurate to three decimal places. Not too shabby for the most simplistic method available.