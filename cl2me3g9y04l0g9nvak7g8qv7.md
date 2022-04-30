## Marching squares

# Introduction
I recently stumbled upon a video on Youtube about an algorithm called *Marching Cubes* (I think it was [this video by Sebastian Lague](https://www.youtube.com/watch?v=M3iI2l0ltbE)). I found it interesting and thought that it would be fun to understand how the algorithm works. Now, I have never worked with graphics in 3D and neither have I worked in Unity so I was initially a bit skeptical of the idea. However, I found out that there is a simplified version of this algorithm for 2D that is called *Marching Squares*. Since I recently started programming in P5 I thought that this would make for a neat little project. A project that potentially could be built upon in the future. 

# How do the squares march?
Contrary to what the name of the algorithm might suggest, the squares aren't *actually * marching, they are, in fact, quite stationary. Either way, let's put the confusing name aside and focus on the algorithm instead. The 2D version of the algorithm has a similar approach to what is done in 3D. There are very good explanations of both algorithms on Wikipedia:

- [Marching Cubes](https://en.wikipedia.org/wiki/Marching_cubes)
- [Marching Squares](https://en.wikipedia.org/wiki/Marching_squares)

So, the algorithm itself consists of three steps but to apply these steps, one needs to have a 2D field of binary values that are evenly separated in both directions (e.g. in the x and y-axis). One way to achieve this field is to do the following:
1. Define a canvas of size \\(m x n\\) 
2. Divide it into squares of size \\(s_x * s_y\\). Preferably, \\(m\\) and \\(n\\) should be evenly divisible by these values as it makes it look a bit better. 
3. Assign a value to every corner in every square. If the canvas size is evenly divisible by the square size there will be \\(n_x = m / s_x + 1\\) and \\(n_y = n / s_y + 1\\) number of points in the x and y-direction, respectively. The \\(+1\\) is there to make sure that the edges of the canvas are included.

Once the three steps above have been conducted one should have \\(n_x * n_y\\) points with a designated value. These values can be chosen in many different ways, one easy way is to assign random values in some range, e.g. values in the range \\([0,1]\\). The layout should end up looking something like this: 

![Canvas.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650382950146/oKSMS1LLi.png align="center")

Where each gray point \\(p_i\\) has an assigned value, as described above. 

The next step is to convert this 2D field of decimal values to a 2D field of binary values. This can be done simply by iterating over all the points and converting them to binary values using a threshold \\(\alpha\\) according to:

$$
p(x,y) = 0 \; \text{if} \; p(x,y) < \alpha \; \text{else} \; 1, \; \forall x \in [0, n_x] \; \text{and } \forall y \in [0, n_y]
$$

Once the 2D field of binary values is available it is just a matter of iterating through each square and, using a lookup table, assigning them a shape. As illustrated on the Wikipedia page, there are 16 different shapes that can be constructed (see the illustration below). The image shows all different shapes that can be assigned to one square, depending on what binary values are assigned to its corners. In this image, the corners with a gray circle have a binary value of 1, while the white circles have a binary value of 0.

![Shapes.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651346801805/jmOkOeXZP.png align="center")

The lines inside of each square are referred to as "contour lines"  and are selected so that they align with any neighboring squares. Once this has been done for all squares on the canvas the contour lines can be drawn on the canvas and a shape will be visible. However, implementing it like this means that the contour lines will *always* have their end points in the middle of the squares. Depending on what the goal is, one might want to make the end points a bit more flexible. 

This is briefly mentioned on the Wikipedia page where they say to do linear interpolation to achieve smoother contour lines. However, it doesn't explain *how* one would go about doing this. 

## Linear Interpolation
There are many ways to do this and the way I chose to do it is probably not an ideal way. But, it gave me satisfactory results so I thought that I would try to explain it here.

Since my implementation assigns decimal values in the range \\([0,1]\\) to the corner points, my idea was to try and move the end points of the contour lines to the corner with the largest value. For example, if one end point is on the left edge of a square, the top left corner has a value of 0.8 and the bottom left corner has a value of 0.1, the end point would be moved towards the top left corner. 

To give a more "mathematical" description of the "interpolation" an image is needed. Below you can see an image of a square where corner \\(p_4\\) has the binary value 0 and the corners \\(p_1, p_2, p_3\\) have the value 1. Looking at the image above, this would correspond to case 13. This means that a contour line should be drawn between the bottom and right edge of the square, as illustrated by the "Original line".

![Interpolation.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651352947422/domGIL6dO.png align="left")

As mentioned earlier, a corner will get the binary value 0 if it has a decimal value *lower* than the threshold \\(\alpha\\). This means that the bottom right corner (\\(p_4\\)) has a smaller value than the lower left (\\(p_3\\)) and the top right (\\(p_2\\)) corners. This would mean that the end point should be moved towards the corners \\(p_2\\) and \\(p_3\\).

To achieve this, my idea was to map the difference between decimal values for the corner pairs \\((p_2, p_4)\\) and \\((p_3, p_4)\\) to the range \\([0,1]\\). Where a value close to 0 would mean that the end point should be moved closer to either the left point or the top point and a value close to 1 would move it towards the bottom or to the right. The equation to achieve this looks like this:
\\[
c = \frac{((p_h - p_l) + r_d)}{r_v / r_d}
\\]
Where \\(p_h\\) is the corner with the highest x or y coordinate, \\(p_l\\) is the corner with the smallest x or y coordinate, \\(r_d\\) is the desired range (\\([0,1]\\)) and \\(r_v\\) is the value range of \\(p_h-p_l\\). The value range of \\((p_h - p_l)\\) is \\([-1,1]\\), since the decimal values are limited to \\([0,1]\\). The output value (\\(c\\)) is the scaling factor by multiplying this value with either \\(s_x\\) or \\(s_y\\) (depending on what edge is being looked at) one can simply move the end point by this value. Hopefully, that made some sense.

# The squares are marching
As mentioned in the beginning, I implemented this in P5. The code is available here:

- [Marching squares GitHub link](https://github.com/timfornell/P5-MarchinSquares)

I won't go through the code in detail, but the marching squares algorithm is implemented in the file called *MarchingSquares.js*. To run the program one can run the *sketch.js* file like any other P5 project. Upon doing so, you are greeted by the output of the algorithm. I have included some examples of the output for some different sizes of squares down below. They are not super interesting to look at since I haven't bothered to draw anything other than the contour lines. But nonetheless, it seems to work and more importantly, the effects of the interpolation are clearly visible as there are many lines that have had their end points moved.

![Example_300_300.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651354735764/P4SbqJw--.png align="center")

![Example_100_100.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651354738643/XKBkcvAm8.png align="center")

![Example_10_10.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1651354740637/jKo-qKmgH.png align="center")
