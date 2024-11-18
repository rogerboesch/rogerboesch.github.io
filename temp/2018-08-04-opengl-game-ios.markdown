---
layout: post
title:  "OpenGL - Part 3: From macOS to iOS!"
image: assets/images/opengl-1.png
author: roro
featured: false
hidden: true
rating: 5
---

### Introduction
In the first two parts of this tutorial we set the base for creating **multiplatform** game's.
So far we just on macOS, but that changes now. This part is about porting the game to **iOS**.
At next we do it then for **Android** before we create more complex games.
Like that we can after concentrate on the game itself in *C++*. The platform dependant part remains
the same.

**Important**: *Part 3 is using OpenGL 1.x, so uses the fixed function pipeline. I've made this to show that for the game engine itself it doesn't matter
so much. However because today's OpenGL implementation uses a programmable graphics pipeline (using shaders) i will "update" this in part 5. (See the Android implementation in part 4 to have an idea already how it looks using OpenGL ES 2.0)*

### Port to iOS
The way how we port our "engine" to another platform remains mostly the same.
Like already this time for iOS:

1. Create the boilerplate code for iOS
2. Implement the low level calls to OpenGL (mainly `RBDrawRect()`)
3. Implement user input (In this case *touch based* input)


#### Boilerplate code
The boilerplate part (#1) for iOS consists of three classes:

**AppDelegate**: Create similar to the macOS version the delegate class that opens in case of iOS a window
and assigns a **View Controller*

{% highlight cpp %}
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    self.viewController = [[ViewController alloc] initWithNibName:nil bundle:nil];

    self.window.rootViewController = self.viewController;
    [self.window makeKeyAndVisible];

    return YES;
}
{% endhighlight %}

Not to much to explain... we just create the window and set a view controller as the root for it.

**ViewController**: The view controller (as the name says) is just responsible to add the views and
set the right size, according to the size of the display. All the "important" things happen in **EAGLView**.


**EAGLView**:
On iOS the easiest way to use OpenGL is to create a custom view (Derived class from `UIView`). Then create
a `CAEAGLLayer`. A Core Animation layer which is capable to display OpenGL. Also create a `EAGLContext` class
which provides a OpenGL Rendering context. That's all the magic.

{% highlight cpp %}
- (id)initWithFrame:(CGRect)frame {
    if ((self = [super initWithFrame:frame])) {
        CAEAGLLayer *eaglLayer = (CAEAGLLayer *)self.layer;

        eaglLayer.opaque = YES;
        eaglLayer.drawableProperties = [NSDictionary dictionaryWithObjectsAndKeys:
                                        [NSNumber numberWithBool:NO], kEAGLDrawablePropertyRetainedBacking, kEAGLColorFormatRGBA8, kEAGLDrawablePropertyColorFormat, nil];

        context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES1];

        if (!context || ![EAGLContext setCurrentContext:context]) {
            return nil;
        }

        renderInterval = 1.0 / 60.0;
    }

    return self;
}
{% endhighlight %}

After that you just have to create a frame- and render buffer which i don't explain in detail here.
The last what we have to now is to add a timer and call it 60 per seconds (lazy approach).
In this we can then call `OnUpdate()` and `OnRender()` from our C++ engine code. So that was the boilerplate part :)


#### OpenGL calls
Because we have so far a game that just uses rectangles to draw the content, the OpenGL calls are very easy to port.
In fact it's just one method `RBDrawRect()` in file *RBRenderHelper*.

{% highlight cpp %}
void RBDrawRect(float x, float y, float width, float height, RBColor color) {
    GLfloat vertices[] = {
        x,       y+height, 0.0f, // Upper left
        x+width, y+height, 0.0f, // Upper right
        x,       y,        0.0f, // Lower left
        x+width, y,        0.0f, // Lower right
    };

    glEnableClientState(GL_VERTEX_ARRAY);
    glColor4f(color.r, color.g, color.b, 1.0);
    glVertexPointer(3, GL_FLOAT, 0, vertices);
    glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
}s
{% endhighlight %}

The main difference here is that we can not use `GL_QUADS` on iOS and use therefore `GL_TRIANGLE_STRIP` to build a rectangle.
Also we can't use immediate mode API on iOS. So what’s the solution? OpenGL Core Profile! This removes all of the old fixed pipeline calls
and replaces them with buffer objects which you only update when you need to. It is a lot more complicated, but you can do way more cool stuff.
Also it's much more performant.


#### User input
At that point we can already show the game on iOS and it works correctly. It has just one "pain-point". We have no user input...
Of course on iOS we have no keyboard and therefore we use the touch screen to control our paddle's.
For this tutorial i implemented a simple approach:

1. Create a custom view class **TouchView**
2. In the **ViewController** class i instantiate two such views; one on each side.
3. Whenever the user touches the upper part  `gGame->OnKey(KeyUp, true)` is called. When the user lift his finger `gGame->OnKey(KeyUp, false)`
   is called.

So in easy words, i let the game engine think, the user has pressed the *Up/Down* keys, (resp. *W/S* for the other player) so all the platform dependent code can remain the same.
An approach that's often used. So the engine has no idea what kind of user input device is available; it get's always the same commands.

{% include youtubeplayer.html id="b7bBmABdUE8" %}

## Platform independent: The Advantage
Maybe you already realise the beauty of multiplatform programming using OpenGL and C++.
We just have to concentrate on a very small code part to make it run on a different platform and all the game logic remains the
same and can used just used *as is* on every platform.
So far we just been on Apple platforms, but the next part will show:

> How to port it to Android!


And as always, the source can be found at [Github](https://github.com/rogerboesch/OpenGLTutorial).


- Part 1: [OpenGL - Is Apple killing it?!]({{ site.baseurl }}{% post_url 2018-07-25-opengl-on-macos %})
- Part 2: [OpenGL - The first little game on macOS!]({{ site.baseurl }}{% post_url 2018-07-31-opengl-game %})
- Part 3: [OpenGL - From macOS to iOS!]({{ site.baseurl }}{% post_url 2018-08-04-opengl-game-ios %})
- Part 4: [OpenGL - Porting to Android!]({{ site.baseurl }}{% post_url 2018-08-08-opengl-game-android %})
