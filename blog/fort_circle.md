# Math Behind Fortnite Shrinking Circle

I've been playing a lot of Fortnite OG, and it got me wondering how exactly the "storm" circle shrinks into the non-concentric "safe" circle. The math behind it is actually extremely simple.

<img src="https://www.echogear.com/product_images/uploaded_images/eye-of-storm-smaller-example.png" width="50%" alt="OG Fortnite Storm Example"></img>
<br>Credit: *echogear.com*

## Definition of Circles
Circles have 2 basic properties: a center point ($x_c, y_c$), and a radius $r$.

For simplicity sake, let's assume there is a predefined circle [($x_c, y_c$), $r$]. (Not proper mathematical notation, but it will simplify writing.)

The new, smaller circle is defined as such  [$(x_c', y_c')$, $r'$], where $r' < r$.<br>

## Randomly Selecting New Center Point
To ensure the smaller circle is inside the larger one, we use this constraint:

$d = \text{Distance between centers}$<br>
$d + r' \leq r$

or

$\sqrt{(x_c' - x_c)^2 + (y_c' - y_c)^2} \leq r - r'$

The new center $(x_c', y_c')$ can be randomly chosen in this region.

Now, we can generate a random radius and its center.

1. Generate a random angle $\theta$ between $0$ and $2\pi$
2. Generate a random radius d: <br>$d=\sqrt{Random(0,1)}*(r-r')$
3. Calculate the new center as
<br>$x_c'=x_c+d*cos(\theta)$
<br>$y_c'=y_c+d*sin(\theta)$

## Shrinking into the Smaller Circle
To shrink the circle, we can use a technique called linear interpolation.

For 2 points $a$ (start) and $b$ (end value), it can be shrunk using the following formula:

$i = \text{interpolated value}$<br>
$t = \text{value between [0, 1] that represents the progress of interpolation}$<br>

$i = a + t*(b-a)$

So when $t = 0$ $i = a$, when $t = 1$ $i = b$, and when $t = 0.5$ $i = $ midpoint between a and b.

Applying this to our circle definition gives us:

$x_t = x_c + t*(x_c'-x_c)$<br>
$y_t = y_c + t*(y_c'-y_c)$<br>
$r_t = r + t*(r'-r)$

Specifically for Fortnite, $t$ can be represented as $\frac{t_p}{T}$, where $t_p$ is time passed in shrinking phase, and $T$ is total length of the phase.

## Desmos Animation
A sample animation & implemented math can be found in this Desmos calculation.
<iframe src="https://www.desmos.com/calculator/ssxsw7jhbv?embed" width="500" height="500" style="border: 1px solid #ccc" frameborder=0></iframe>

This same shrinking math also works for the "new" storms where at endgame, the inner circle may be placed inside the current storm.
<iframe src="https://www.desmos.com/calculator/dvxrdyeqou?embed" width="500" height="500" style="border: 1px solid #ccc" frameborder=0></iframe>

## Conclusion
Epic Games may use a different formula than this, but it's what I found first and the animation looks very similar.