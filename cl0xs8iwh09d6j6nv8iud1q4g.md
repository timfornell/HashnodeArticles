## Jumping on the Coding Train of creative coding

## Introduction
Ever since learning how to program at the university, I have wanted to do more "visual" programming projects. This is probably the reason why I recently spent a weekend looking at a bunch of videos on the Coding Train [youtube channel](https://www.youtube.com/c/TheCodingTrain) by Daniel Shiffman. Since I am a math nerd and I enjoy experimental work I found Daniel's videos interesting. He manages to achieve some pretty impressive results with, what I would say, a small amount of code. The stuff that Daniel does is a bit more lightweight than what e.g. Sebastian Lague does ([link to his channel](https://www.youtube.com/c/SebastianLague)) so I thought that I would try and follow along with some of his challenges. It would also serve as a good excuse to get into JavaScript and creative coding.

Daniel has written a book called [**The nature of code**](https://natureofcode.com/) where he talks about the connection between math and nature and how one can use programming to visualize/simulate these relationships. The project I tried to follow along with is from the chapter about autonomous agents. More explicitly it is from chapter 6.13 and is about flocking and Daniel implements the algorithm in this [video](https://youtu.be/mhjuuHl6qHM).

If you don't want to read all the boring details you can jump immediately to the results section, where you can find a link to the actual finished project that you can interact with.

## Flocking algorithm
As I just mentioned, the algorithm itself is explained in the Daniels book The nature of code. But, to make this article understandable by itself I will include a brief explanation of the algorithm here. If you want it properly explained I advise you to either read his book or look at the YouTube video linked above. 

The algorithm is based upon work done by Craig Reynolds, who did a computer simulation of this back in 1986. It is available on this [website](https://www.red3d.com/cwr/boids/) together with some videos (that may or may not work depending on your browser). Either way, to achieve the flocking behavior Craig defined three rules that the actors should follow.

1. **Separation **(also referred to as avoidance) - Steer to avoid crowding local flockmates
2. **Alignment** - Steer towards the average heading of local flockmates
3. **Cohesion** - Steer to move toward the average position of local flockmates

![FlockingRules.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1646510328304/YoBJ43yKg.png)

The idea is that each of these criteria will exert a steering force on the actors according to *steering force = desired - velocity*. However, the way it is implemented (in my code at least) makes steering *acceleration* a more suitable name.

Looking at the images above it looks like the two forces **separation** and **cohesion** would cancel each other out. That is just an unfortunate consequence of how I chose to draw the images. Because both of the forces are needed if you want to have a flock that looks somewhat natural. For example, without **cohesion** the actors would not try to group with their neighboring actors, making it hard to form proper flocks, and without **separation** the actors will just move around without worrying about colliding with other actors. Although, it should be mentioned that I have not implemented any functionality for what happens when two actors collide with each other.

## Implementation
As I mentioned in the introduction, this is a way for me to get introduced to javascript. The reason for this is that Daniel is working with something called [p5](https://p5js.org/). I don't plan to go into detail about what p5 is since that information is available on their website. But in short, it is a JavaScript library for creative coding. 

Since this was my first ever time writing JavaScript I used Daniel's video (linked previously) as a reference. Naturally, this lead to the overall structure of the code being very similar to the code Daniel wrote. For anybody who might be interested in comparing, you can find the code here:

- [Daniels code](https://thecodingtrain.com/CodingChallenges/124-flocking-boids)
- [My code at GitHub](https://github.com/timfornell/P5-Flocking)

Either way, the project consists of two files, one called *sketch.js* and one called *boid.js*. There are other files in the repository but they do not contain any functionality I have implemented so I will not go through them here. 

### Sketch.js
My understanding of p5 and JavaScript is a bit limited but from what I understand, the sketch.js file works as the *main* file of the program. Meaning that this file is in charge of setting up your canvas and any drawing you want to do on it. In this project the sketch file contains three functions: **setup**, **draw** and **updateParameters**.

#### Setup
The **setup** function is called once upon startup and initializes the following variables:
- sliders
- sliderValues
- flock

It also initializes the canvas that is used to draw on and to display stuff to the user. The sliders variable is an object that stores, you guessed it, sliders. These sliders allow the user to tune some parameters of the algorithm. The syntax for creating an *object* is quite similar to what one might encounter when working with dictionaries in Python:

```
sliders = {
      "detDistance": createSlider(1, height, value=20, step=1),
      "maxSteeringForce": createSlider(0.05, 1, value=0.05, step=0.05),
      "maxVelocity": createSlider(0.5, 10, value=2, step=0.5),
      "collAvoidance": createSlider(0, 100, value=10, step=0.5),
      "alignment": createSlider(0, 100, value=10, step=0.5),
      "centerAdjust": createSlider(0, 100, value=10, step=0.5)
   };
```
As you can see there are six different parameters available to modify. To make it somewhat user-friendly the current values can be displayed on the canvas by using the function `createP()`. This is what is stored in the variable sliderValues, one for each parameter. 

Lastly, the variable flock, which is an array, is filled with the actors that will make up the flock:
```
for (let i = 0; i < 200; i++) {
      flock.push(new Boid(random(0, width), random(0, height), i));
}
```
In this case, there are 200 actors and they are all initialized at a random position on the canvas. As you can see, each actor is an instance of the class called *Boid* (this is the same name as Daniel used as well). I will go into more detail further on what this class contains. 

#### Draw
The **draw** function is the main loop of the program and in this project, it is fairly simple:

```
function draw() {
   background(51);

   updateParameters();

   for (let boid of flock) {
      boid.move(flock);
      boid.draw();
   }
}
``` 
It starts by drawing the background (in a grayscale), it then calls the function updateParameters (see below for an explanation of that function). After this, it iterates through every actor in the flock, updates their state, and then draws them. 

#### UpdateParameters
The **updateParameters** function is called once every iteration of the draw loop and reads the values from the sliders mentioned earlier and then displays these values. It also updates these parameters in the Boid class since they are static and consequently shared by all entities of that class. 

### Boid.js
This file contains the majority of the "functional" parts of the code. It implements the class called *Boid* and all functions that are necessary to make the individual boids abide by the three rules specified by the flocking algorithm. 

As mentioned earlier, the class contains some static parameters that can be tuned via the sliders to change the behavior of the boids. Since these parameters should be the same for all objects this is a way to prevent having to update the parameters for *all* 200 boids individually. To update these parameters, a static function is needed. This function is suitably called updateParameters and, as mentioned earlier, is called once for every iteration of the draw loop.

The constructor for the class is not that interesting so I will only mention that it initializes three vectors that are unique for each instance of the class:

1. Position
2. Velocity
3. Steering force (acceleration)

Apart from this, the class also contains several functions:

1. `draw()` - Draws the object as a circle at the given position. Also visualizes the direction of travel.
2. `move(flock)` - Adjusts the steering force according to the rules of the flocking algorithm, then updates the velocity and position.
3. `wrapPosition()` - Used to avoid boids from moving outside of the visible area, e.g. a boid moving outside the left edge will reappear at the right edge and so on.
4. `setVelocity()` - First limits the steering force to a max value, then updates the velocity with it. Then it also limits the velocity to a max value. Both the max steering force and max velocity are tunable.
6. `findNeighbours(flock)` - Loops through all actors and finds those that are within detection distance. The detection distance is a tunable parameter.
7. `adjustVelocityToNeighbours(neighbors)` - Takes the boids that are within detection distance and runs the flocking algorithm using them. 

#### AdjustVelocityToNeighbours
Since this function can be considered the "brain" of the actor/boid I thought I would explain it a bit more in detail. It loops through all the neighbors and calculates their average position and velocity. These values are used for the alignment and cohesion rules. At the same time, it also calculates the vector between the current boid and each neighbor. This is needed to abide by the separation rule (collision avoidance). This vector is then scaled by dividing it by the magnitude. This means that boids that are close, have a bigger impact on the separation steering force (or collision avoidance force). Also, since nothing is limiting two boids to have the same position, a factor of 0.01 is added when dividing to avoid division by zero.
```
   for (let i = 0; i < neighbours.length; i++) {
         let boid = neighbours[I];
         
         // Used for average heading
         averageVelocity.add(boid.velocity);
        // Used for average flock position
         averagePosition.add(boid.position);

        // Used for collision avoidance
         let diff = this.position.copy();
         diff.sub(boid.position);
         // +0.01 to avoid dividing by 0
         diff.div(diff.mag() + 0.01);
         collisionAvoidanceForce.add(diff);
      }
}
```
These values are then used to calculate the steering force. The average position and velocity of the flock are divided by the number of neighbors to make it an actual average. This is also the point where some tuning parameters come into play. Before adding each component to the final steering force they are multiplied by a factor. Meaning, by changing the values of these three parameters the behavior of the actors can be changed. 

## Results
To conclude this article (that became way longer than I expected) I want to give a few examples of the results and some stuff that could potentially be improved upon.

Github has a neat feature where you can "deploy" your code directly on Github. I have done this with this project and you can find it here: 
- [Flocking algorithm deployment](https://timfornell.github.io/P5-Flocking/)

The initial parameters provided upon loading the page can achieve flocks of boids that look and behave reasonably well. Currently, 200 boids are moving around and if one waits a minute they start to form small flocks, which eventually combine into larger flocks.

![FlockingSmallGroups.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647688097988/tX83Snb5Mr.png)
Small groups of boids form after only a few seconds.

![FlockingBigGroups.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647688097069/OzQ9Szuaq.png)
After approximately a minute the previously small groups have combined into larger groups.

By tweaking the different parameters one can achieve some rather interesting formations. In the image below I managed to get the boids to arrange in what I can only describe as a cell-like structure.

![CellStructure.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647688496694/06tXzaPII.png)

If you have some time to waste, feel free to play around with it. I found it to be a bit therapeutic to just sit and watch the little circles move around. 

## Discussion
Now, there is stuff that I intentionally did not implement when doing this project. I tell myself that I did it because this is my "first project" but in all honesty, some of the stuff I just don't know how to implement. Either way, I'm gonna mention a few things that I think would improve the behavior of the flock.

Since the canvas is limited in size, one can choose to either make the actors wrap around the edges (as I have done) or to try and avoid the edges. Both of these approaches have consequences for the behavior of the boids. 

**Edge wrapping:** The way I implemented the edge wrapping and the way the boid finds it neighbors a boid that is part of a flock that wraps around e.g. the left edge will not see the other boids that appeared at the right edge. This will affect the steering force it will calculate and it can cause the flock to either split up or to start oscillating a bit as it wraps around the edge. 

**Edge avoidance:** Another thing that would make sense instead of edge wrapping is to make the boids steer away from the wall, similar to how they steer away from their neighbors. This would remove the detection issue mentioned above.

**Collision:** It would be interesting to see if flocks can form if the boids can bump into each other. As I mentioned earlier, the boids can freely move through other boids since there is no collision implemented. 

**Field of view:** Currently, the boids can see 360 degrees around them, this doesn't make sense in real life. So to make it more "life-like" one could limit the boids field of view to e.g. 120 degrees. 

**Optimization:** I have limited the number of boids to 200 since having more than that significantly decreases the framerate. I have no experience in optimizing this type of algorithm where you have actors moving around freely on a canvas. But to make it possible to run with e.g. 1000 actors some optimization will be needed.

There are probably many more things one can improve upon, but that will have to wait for another project. 

That is gonna wrap this article up. Thank you for reading! 