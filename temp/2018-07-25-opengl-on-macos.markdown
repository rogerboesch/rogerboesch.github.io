---
layout: post
title:  "OpenGL - Part 1: Is Apple killing it?!"
image: assets/images/opengl-1.png
author: roro
featured: false
hidden: true
rating: 4
---

### Introduction
The bad news is that Apple want stop OpenGL support on their platforms completely. It's already marked as deprecated in iOS 12 and macOS Mojave. So sooner or later OpenGL will die (at least on Apple platforms). But is that an issue? We have Unity3D, Unreal etc. But let us first make a short recap, what the different levels are.

**Important**: *Part 1 is using OpenGL 1.x, so uses the fixed function pipeline. I've made this to show that for the game engine itself it doesn't matter
  so much. However because today's OpenGL implementation uses a programmable graphics pipeline (using shaders) i will "update" this in part 5.
   (See the Android implementation in part 4 to have an idea how it looks using OpenGL ES 2.0)*

#### Engine
When it comes to graphics programming, either in 3D or 2D, most of the programmers use engines like Unity 3D, Unreal or other more 2D oriented solutions. I call them Engine's because their support goes far beyond just "simple" graphic calls. They are a combination of Tools, Code and even entire Asset Stores.


#### Framework
The next lower level are frameworks like SceneKit, Ogre3D and others. So more or less "libraries" which are helpful to create graphic applications and games in a abstract way, but not more. Sometimes physics support is included, sometimes even not that.


#### API
The lowest level are then API's like OpenGL, Metal or Vulkan. So more or less direct interface's between the GPU and the Application.

This article is dedicated to the API level and more precisely on OpenGL. Of course some might ask why you should still work on this level anyways, it's so much more convenient to use an Engine like Unity 3D with all the support, scripting, helpful modules etc. to create games or applications.

While this is true, there are still many reasons to use and more important learn:

- It gives a solid understanding of how graphics programming works
- It's available on every platform you can imagine
- I has no boilerplate which is especially important when you want to integrate graphics as part of an existing app-(lication)
- It's the fasted way to create graphic and often also the only way on special hardware.

But Apple will kill OpenGL! So does it make sense to learn now OpenGL? Why not use Metal then? The replacement API for OpenGL from Apple. The most obvious reason is that it's proprietary (Apple) and makes most sense on iOS. But in todays landscape it's important to support as many platforms as possible. That's also the reason why i will use C++ (for 99%) of the code.


### OpenGL on macOS
Because many programmers work on macOS (as i do), i've decided to show how it works on Apple platforms. Also there is a lack of good information's on using OpenGL on macOS and iOS. So another good reason to change that now ;) Apple want stop OpenGL support on their platforms completely and force to use Metal (their own Graphics API). But even with that in mind, it makes sense to understand how OpenGL (or it's successor Vulkan) works. Because when understand Graphic API's in general, it's relatively easy to learn another one. Even Metal!


## The Basics
Unlike frameworks and engines, there is some basic/setup work to do before you can start writing a real (game) code for OpenGL. In macOS, this is also the part which must be written in Objective-C or Swift, but that's just a very very small part in a full game. All the rest can be written in C++ (which is still the best multiplatform language). I don't talk too much about that part here, but i have created a basic template which contains all this, so you directly start working with OpenGL in C++. Simply spoken there are three files which are needed to create a OpenGL based graphics context (A kind of canvas you can draw in).


### main.m
The main file contains the entry point of the application `main()`. The main purpose of this is to initialise `NSApplication`, which is the base of every macOS Cocoa Application and assign the AppDelegate to it.


### AppDelegate.mm
AppDelegate is also a relatively small class and just creates the window and a small menu (to quit the application later). So it implements the application's delegate as the name says.


### RBNSOpenGLView.mm

This is the most important part of the "boilerplate code". It's a derived class from `NSOpenGLView` and
creates the OpenGL context with the correct pixel format. Also it's responsible to call `OnRender()` method
with 60 frames per second. That's all we do in Objective-C. The real code is now all in C++, so there's not much
much platform dependant code.


## All together
Like written before, the main topic is not about the boilerplate code needed to have a working OpenGL environment.
It's about using OpenGL. And that is what we will do now.


### The Game Skeleton
So far all was platform dependant by using Objective-C and Cocoa. But that changes now. 99% of all the rest
will be C++ and OpenGL, so code that can be used on any platform.
The advantage of using Objective-C on macOS is the very easy way to call C++ Code. So we will create a global **Game** class (*Game.cpp* and *Game.hpp*) which we call from *RBNSOpenGLView.mm*. We implement three methods:

- OnInit() - Used to initialise the game
- OnUpdate() - Called every frame and allows to update the game (like moving objects etc.)
- OnRender() - Called every frame and allows to render the game to OpenGL

That's it. That's all the magic of using C++ in your game on macOS, resp. iOS.
The source can be found at [Github](https://github.com/rogerboesch/OpenGLTutorial).
The target called **partI** (this companion code) can also be used as a base template for your own games.


### The first rendered Triangle
While the goal was to write the boilerplate code and have a working OpenGL context, we will not stop at this point.
We will render something simple, just to show that OpenGL works :)
OpenGL makes it easy to write complicated stuff, but at the expense that drawing a simple triangle is actually quite difficult.


#### The VAO
We don't go into details now, but at first we need to create a Vertex Array Object and set it as the current one:

{% highlight cpp %}
glGenVertexArrays(1, &vertexArrayID);
glBindVertexArray(vertexArrayID);
{% endhighlight %}

This code we write in `OnInit()`, so it's just called once at the begin. If you  want to know more about VAOs,
there are a few other tutorials out there, but this is not very important.


#### Screen Coordinates
A triangle is defined by three points. When talking about “points” in 3D graphics, we usually use the word “vertex” (vertices on the plural).
A vertex has 3 coordinates: X, Y and Z. You can think about these three coordinates in the following way :

- X is on your right
- Y is up
- Z is towards your back

So we need three 3D points in order to make a triangle:

{% highlight cpp %}
static const GLfloat gVertexBufferData[] = {
   -1.0f, -1.0f, 0.0f,
    1.0f, -1.0f, 0.0f,
    0.0f,  1.0f, 0.0f,
};
{% endhighlight %}

The first vertex is (-1, -1, 0). This means that unless we transform it in some way, it will be displayed at (-1, -1) on the screen.
What does this mean?
The screen origin is in the middle, X is on the right, as usual, and Y is up. This is something you can’t change, it’s built in your graphics card.

- (-1, -1) is the bottom left corner of your screen
- (1, -1) is the bottom right
- (0, 1) is the middle top

We define this array of vertices in `Game.cpp`.


#### Drawing the triangle
The next step is to give this triangle to OpenGL. We do this by creating a buffer:

{% highlight cpp %}
glGenBuffers(1, &vertexBuffer);
glBindBuffer(GL_ARRAY_BUFFER, vertexBuffer);
glBufferData(GL_ARRAY_BUFFER, sizeof(gVertexBufferData), gVertexBufferData, GL_STATIC_DRAW);
{% endhighlight %}

This code we also implement in `OnInit()` so it's just called once at the begin.
Now, in our main loop `OnRender()`, we can draw our awesome triangle :

{% highlight cpp %}
// 1st attribute buffer : vertices
glEnableVertexAttribArray(0);
glBindBuffer(GL_ARRAY_BUFFER, vertexBuffer);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, (void*)0);

// Draw the triangle !
glDrawArrays(GL_TRIANGLES, 0, 3);
glDisableVertexAttribArray(0);
{% endhighlight %}

And of course don't forget to declare the two variables in *Game.hpp*.

{% highlight cpp %}
GLuint vertexArrayID;
GLuint vertexBuffer;
{% endhighlight %}

If yo u’re done all right, you can see the result in white.

![Screenshot 1]({{"/assets/images/opengl-on-macos-I-1.png"}})

The source can be found at [Github](https://github.com/rogerboesch/OpenGLTutorial).

### Let's write a simple game
Of course just watch a black screen with a white triangle is not very impressive,
so to make this tutorial about OpenGL more fun, we will write in part II a little game.


- Part 1: [OpenGL - Is Apple killing it?!]({{ site.baseurl }}{% post_url 2018-07-25-opengl-on-macos %})
- Part 2: [OpenGL - The first little game on macOS!]({{ site.baseurl }}{% post_url 2018-07-31-opengl-game %})
- Part 3: [OpenGL - From macOS to iOS!]({{ site.baseurl }}{% post_url 2018-08-04-opengl-game-ios %})
- Part 4: [OpenGL - Porting to Android!]({{ site.baseurl }}{% post_url 2018-08-08-opengl-game-android %})
