---
layout: post
title:  "OpenGL - Part 2: The first little game on macOS!"
image: assets/images/opengl-1.png
author: roro
featured: false
hidden: true
rating: 4
---

### Introduction
In the first part we have talked about how to create an OpenGL environment on macOS
to create games using OpenGL and C++. We have created all the boilerplate code, a simple C++
Game Engine to implement the Render Loop and an example triangle to proof it works.

**Important**: *Part 2 is using OpenGL 1.x, so uses the fixed function pipeline. I've made this to show that for the game engine itself it doesn't matter
so much. However because today's OpenGL implementation uses a programmable graphics pipeline (using shaders) i will "update" this in part 5.
(See the Android implementation in part 4 to have an idea how it looks using OpenGL ES 2.0)*

### Let's write a simple game
Of course just watch a black screen with a white triangle is not very impressive,
so to make this tutorial about OpenGL more fun, I will write in part II a little game.


## Architecture
To write platform independent code, it's very important that you have a clear architecture/structure.
Especially differentiate between platform dependent and platform independent code is needed.
Of course with the goal to write as much as possible platform independent.


### OpenGL Boilerplate code
The base OpenGL code from part I is almost the same as i use in part 2. I just added the possibility to get user input to it,
which is not really OpenGL related but in most cases platform dependent. In macOS, the keyboard events are catched by a view (or a window) and
because we have just one view (**RBNSOpenGLView**), get the keyboard events is also done there and passed over to the **Game** class.
But i also have to enhance the OpenGL part. We need functions to draw a rectangle in any color.
Also we need a way to write numbers to the screen to show the score of the game. Those are in this case also created by rectangles.
For that, i've added a new file *RBRenderHelper.cpp* which contains two public methods to exactly do that.

{% highlight cpp %}
void RBDrawRect(float x, float y, float width, float height, RBColor color);
void RBDrawNumber(float x , float y , int n, RBColor color);
{% endhighlight %}

Also the implementation is straight forward and drawing a rectangle is realised in a few lines.

{% highlight cpp %}
void RBDrawRect(float x, float y, float width, float height, RBColor color) {
    glColor4f(color.r, color.g, color.b, color.a);

    glBegin(GL_QUADS);

    glVertex2f(x, y);                     // Bottom left
    glVertex2f(x + width, y);             // Bottom right
    glVertex2f(x + width, y + height);    // Top right
    glVertex2f(x, y + height);            // Top left

    glEnd();
}
{% endhighlight %}

So basically i set the color, define 4 vertices (this time in code and not as data array) and surround it with
`glBegin()` resp. `glEnd()`. That's basically all needed to render the game. Of course it's 2D and has no textures,
but we are able to write our example game. `RBDrawNumber()` calls internally also `RBDrawRect()` to write
the different segments of a number from 0 to 9.
So at the base of the engine there is really just `RBDrawRect()`. That means also,
if we change the platform, that's all we have to change (besides the boilerplate code of course), which is great!


### Game Engine
I will enhance our "Engine" code also a little bit to make it more universal and also to get user input and fixed timing.
Besides that i introduce some helper classes for handling vertices and colors.

- **RBColor** is a simple container class for holding r(ed), b(lue), (g)reen and a(alpha) values
- **RBVector2** holds two float values to define a 2D vector (x,y). Besides that it contains some methods and
  operators to simplify the work with a vector.

Also I have enhanced the **Game** class a little bit.

1. It has become a method `OnKey()` to receive Keyboard changes and make them accessible
2. A method `AddGameObject()` to add a game object to an internal list.
   This allows the **Game** class to render and update all of them

And last but not least the `OnUpdate()` method gets a parameter *delay*, which is actually the time since the last call.
So every time related variable (like *speed*) will be multiplied by it to have a constant movement.


#### The game object

Also i introduce a **GameObject** class that holds at the moment 4 values:

- *position*: The position in the 2D space
- *size*: The size of the game object (currently rectangles)
- *speed*: The speed of the game object
- *color*: The color (for Pong we just use white)

On top of that we have three methods to mention here:

{% highlight cpp %}
virtual void Update(float delay);
virtual void Render();
{% endhighlight %}

This two supports the game loop and get called by the **Game** class. It does
that by calling **all** game objects added to the *_gameObjects* list.


{% highlight cpp %}
bool Collide(GameObject *object);
{% endhighlight %}

Also to mention is the method `OnCollide()` which is a simple test if two rectangles are intersect
each other. We will need it in the little game we create.


### The Game: Pong
To keep the game logic and graphic requirements simple, i have decided for a game that just uses rectangles.
[Pong](https://en.wikipedia.org/wiki/Pong). One of the first Arcade Games ever.
It's very simple to create just with rectangles, but makes a lot of fun.
Two players can play something similar to tennis and you will see, that a good game is more dependent on
good gameplay then on good graphics. Sometimes even better then todays high-res 3D graphics.


#### Pong class
To keep the class **Game** reusable for any kind of games, i implement the game logic in a new class
**Pong** which is derived from **Game**. We just have to overwrite the three virtual methods, to do so.
In the header file *Pong.hpp* i declare the 3 game elements we will use in the game.

{% highlight cpp %}
GameObject* _ball;
GameObject* _paddle1;
GameObject* _paddle2;
{% endhighlight %}

As you see, all of them are from class **GameObject** but we could also create a class for the ball and derive it from
**GameObject** and another one for each of the paddles. But for this simple game we will not do that.

Now to the implementation of the game.

1. CreateContent(): Here I create all the classes and position them on the screen and add to the list of game objects.
2. Update(): This method is responsible for the game logic (see below).
3. Render(): Because the objects itself gets rendered by the **Game** class, we just render here the score and middle line


#### Game Logic
The game logic is easy to explain and therefore also easy to write ;). It contains the following rules:

- The ball starts in one direction
- Whenever it touches the screen on top or at bottom it bounces in the same angle to the other direction
- When a paddle hits the ball it bounces to the other side. The angle depends if it hits the paddle in
  the middle or more outside.
- When the paddle misses the ball, the other player get one point and the ball starts again in the middle
- When a player has 10 points, he wins and the game starts again

{% include youtubeplayer.html id="ijuXTnqpZrA" %}

The source can be found at [Github](https://github.com/rogerboesch/OpenGLTutorial).

That's all! I hope you like and you see how a simple game can make a lot of fun. Especially when programmed
by yourself! But the most best is we have now the foundation to build many different games
and also the possibility to port them easily to any platform. **Both of it i will show in the next articles!**


- Part 1: [OpenGL - Is Apple killing it?!]({{ site.baseurl }}{% post_url 2018-07-25-opengl-on-macos %})
- Part 2: [OpenGL - The first little game on macOS!]({{ site.baseurl }}{% post_url 2018-07-31-opengl-game %})
- Part 3: [OpenGL - From macOS to iOS!]({{ site.baseurl }}{% post_url 2018-08-04-opengl-game-ios %})
- Part 4: [OpenGL - Porting to Android!]({{ site.baseurl }}{% post_url 2018-08-08-opengl-game-android %})
