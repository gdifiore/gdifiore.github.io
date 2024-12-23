py# Math Behind Fortnite Shrinking Circle

I've been playing a lot of Fortnite OG, and it got me wondering how exactly the "storm" circle shrinks into the non-concentric "safe" circle. The math behind it is actually extremely simple.

<img src="https://www.echogear.com/product_images/uploaded_images/eye-of-storm-smaller-example.png" width="50%" alt="OG Fortnite Storm Example">
<br>Credit: *echogear.com*

## Definition of Circles
Circles have 2 basic properties: a center point ![x_c, y_c](images/equation.png), and a radius ![r](images/equation-1.png).

For simplicity sake, let's assume there is a predefined circle ![alt text](images/equation-2.png) (Not proper mathematical notation, but it will simplify writing.)

The new, smaller circle is defined as such  ![alt text](images/equation-3.png), where ![alt text](images/equation-4.png).<br>

## Randomly Selecting New Center Point
To ensure the smaller circle is inside the larger one, we use this constraint:

![alt text](images/equation-5.png)

or

![alt text](images/equation-6.png)

The new center ![alt text](images/equation-7.png) can be randomly chosen in this region.

Now, we can generate a random radius and its center.

1. Generate a random angle ![alt text](images/equation-9.png) such that ![alt text](images/equation-8.png)
2. Generate a random radius d: <br>![alt text](images/equation-10.png)
3. Calculate the new center as<br>
![alt text](images/equation-11.png)

## Shrinking into the Smaller Circle
To shrink the circle, we can use a technique called linear interpolation.

For 2 points ![alt text](images/equation-12.png) (start) and ![alt text](images/equation-13.png) (end value), it can be shrunk using the following formula:

![alt text](images/equation-14.png)

So,<br>
![alt text](images/equation-16.png)

Applying this to our circle definition gives us:

![alt text](images/equation-15.png)

Specifically for Fortnite, ![alt text](images/equation-17.png) can be represented as ![alt text](images/equation-18.png) where ![alt text](images/equation-19.png) is time passed in shrinking phase, and $T$ is total length of the phase.

## Desmos Animation
A sample animation & implemented math can be found in this Desmos calculation.
<iframe src="https://www.desmos.com/calculator/ssxsw7jhbv?embed" width="500" height="500" style="border: 1px solid #ccc" frameborder=0></iframe>

This same shrinking math also works for the "new" storms where at endgame, the inner circle may be placed inside the current storm.
<iframe src="https://www.desmos.com/calculator/dvxrdyeqou?embed" width="500" height="500" style="border: 1px solid #ccc" frameborder=0></iframe>

## Conclusion
Epic Games may use a different formula than this, but it's what I found first and the animation looks very similar.

[Desmos Calculator](https://www.desmos.com/calculator/uzgt6jsagu)