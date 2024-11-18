---
layout: post
title:  "OpenGL - Part 4: Porting to Android!"
image: assets/images/opengl-1.png
author: roro
featured: false
hidden: true
rating: 4.5
---

### Introduction
The first three parts of the tutorial was related to Apple platforms (macOS and iOS).
Now in this part we go really multi-platform and create an Android version!

**Important**: *This part is already using OpenGL 2.x, so uses a programmable graphics pipeline (using shaders).*

#### Android NDK
"Normal" Android apps are written in Java and are based on the Java SDK. For this tutorial we want of course use
C++ and therefore we need the Android NDK (Native development kit). For more infos on how to to install and use
see also the links section below.


### Boilerplate code for Android
On macOS, resp. iOS we have created the boilerplate code in it's native language Objective-C. (Swift would also be possible)
Now on Android, this part is written in Java. It's also just a minimal code that creates an Activity and the OpenGL context.
Also it's responsible to setup JNI which allows to call after C/C++ code from Java. But step by step.


#### GLActivity.java
The activity class is where all starts. It mainly creates three views and layout them on the screen:

{% highlight java %}
GL2JNIView mView;
GLTouchView mLeftView;
GLTouchView mRightView;
{% endhighlight %}

The first is the OpenGL view and the two other views are a the touch based interface similar to the iOS
implementation. (*See GLTouchView.java*)

{% highlight java %}
@Override protected void onCreate(Bundle icicle) {
     super.onCreate(icicle);

     setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE);

     DisplayMetrics dm = new DisplayMetrics();
     this.getWindow().getWindowManager().getDefaultDisplay().getMetrics(dm);
     int width = dm.widthPixels;
     int height = dm.heightPixels;

     RelativeLayout layout = new RelativeLayout(this);
     layout.setLayoutParams(new RelativeLayout.LayoutParams(width, height));
     setContentView(layout);

     mView = new GL2JNIView(getApplication());
     mView.setLayoutParams(new RelativeLayout.LayoutParams(width, height));
     layout.addView(mView);

     mLeftView = new GLTouchView(this);
     mLeftView.tag = 1;
     mLeftView.setLayoutParams(new RelativeLayout.LayoutParams(width/2, height));
     mLeftView.setBackgroundColor(GLActivity.TRANSPARENT);
     layout.addView(mLeftView);

     RelativeLayout.LayoutParams rightLayoutParams = new RelativeLayout.LayoutParams(width/2, height);
     rightLayoutParams.leftMargin = width/2;
     mRightView = new GLTouchView(this);
     mRightView.tag = 2;
     mRightView.setLayoutParams(rightLayoutParams);
     mRightView.setBackgroundColor(GLActivity.TRANSPARENT);
     layout.addView(mRightView);
}
{% endhighlight %}

There is not much magic, i just use a relative layout and place first the OpenGL view in it. Then
(with transparent background color) the two touch views. One on the left side and one on the right side.


#### GLTouchView.java
This is the class responsible for the user input and quite easy to understand:

{% highlight java %}
@Override
 public boolean onTouchEvent(MotionEvent event) {
     mX = event.getX();
     mY = event.getY();

     switch (event.getAction()){
         case MotionEvent.ACTION_DOWN:
             GL2JNILib.touch(tag, 1, (int)mX, (int)mY);
             break;
         case MotionEvent.ACTION_MOVE:
             break;
         case MotionEvent.ACTION_UP:
             GL2JNILib.touch(tag, 0, (int)mX, (int)mY);
             break;
         default:
             return false;
     }

     return true;
 }
{% endhighlight %}

In this custom view (derived from **android.view.View**) we just override `onTouchEvent()` and send the event information
straight away (via JNI, see *GL2JNILib.java*) to the native part in *RBRender.cpp*.
The **tag** property is to differentiate between the *left* and *right* touch view (as used in this example) and the second parameter
of `GL2JNILib.touch(tag, 1, (int)mX, (int)mY);` is either **1** (Touch down) or **0** (Touch up). Easy approach, right?


#### GL2JNIView.java
This class is 1:1 from the NDK example *hello-gl2* and the view responsible to set up a valid OpenGL ES 2.0 context by
derive a view class from **GLSurfaceView**.
I use that class as is!


#### GL2JNILib.java
**GL2JNILib** is the bridge class between the Java part and the C++ code.

{% highlight java %}
public class GL2JNILib {
     static {
         System.loadLibrary("gl2jni");
     }

    /**
     * @param width the current view width
     * @param height the current view height
     */
     public static native void init(int width, int height);
     public static native void step();

    /**
     * @param tag ID of touch view (useful when have multiple)
     * @param down 1 if pressed, 0 if released
     * @param x x position of touch
     * @param y y position of touch
     */
    public static native void touch(int tag, int down, int x, int y);
}
{% endhighlight %}

As you see it's just a class with static methods which describe the native methods and make
them accessible to Java. The first method `init()` will be called after initialised the OpenGL context
and the `step()` method once per frame. *Similar to renderLoop() in the iOS Version*.
While both of this calls are already part of the NDK sample, i've added a touch method, which delivers
the touch information to the native functions implemented in *RBRender.cpp*


#### RBRender.cpp
So far so good and not so much different then on iOS side, except the fact that it's implemented in Java,
which we leave now... The entry point -or- middle layer between the Java boilerplate code and our game engine
is in *RBRender.cpp*. In fact it's not C++ but C code. Calling directly C++ classes would need another wrapping,
which I avoided here at this point. So what's done in *RBRender.cpp*? Mainly three things:

1. JNI declaration
2. Create the Shader
3. Call the methods in *Game.cpp* (Our engine)


**JNI declaration**

{% highlight c %}
extern "C" {
    JNIEXPORT void JNICALL Java_com_android_gl2jni_GL2JNILib_init(JNIEnv * env, jobject obj, jint width, jint height);
    JNIEXPORT void JNICALL Java_com_android_gl2jni_GL2JNILib_step(JNIEnv * env, jobject obj);
    JNIEXPORT void JNICALL Java_com_android_gl2jni_GL2JNILib_touch(JNIEnv * env, jobject obj, jint tag, jint down, jint x, jint y);
};

JNIEXPORT void JNICALL Java_com_android_gl2jni_GL2JNILib_init(JNIEnv * env, jobject obj,  jint width, jint height) {
    setupGL(width, height);
}

JNIEXPORT void JNICALL Java_com_android_gl2jni_GL2JNILib_step(JNIEnv * env, jobject obj) {
    renderFrame();
}

JNIEXPORT void JNICALL Java_com_android_gl2jni_GL2JNILib_touch(JNIEnv * env, jobject obj, jint tag, jint down, jint x, jint y) {
    userInput(tag, down, x, y);
}
{% endhighlight %}

Besides the somewhat curious declaration, this JNI definition is needed to make the calls from Java possible.
If you want know more about the specific syntax see the JNI related link below.
Important to understand here is that we are now in C code and able to implement all the rest in C/C++.


**Create the shader**
As you see, we need here slightly more to come down where we call `RBDrawRect()`.
This is because of two reasons: We call from Java and more important: **We use OpenGL ES 2.0**.
This has one important difference: We do not longer use CPU related calls to define the projection matrix,
color etc. but use a **shader program** (Code that runs on the GPU) for this.
Shader programming language has a C like syntax and consists of a:

- **Vertex shader** (*handles the processing of individual vertices*) and a
- **Fragment shader** (*process a fragment generated by the rasterization into a set of colors*).

(*More on shader programming see links section below*)

But let's look at the code (we create in code and pass later to the GPU):

{% highlight c %}
attribute vec4 vPosition;
uniform float fWidth;
uniform float fHeight;

void main() {
  mat4 projectionMatrix = mat4(2.0/fWidth, 0.0, 0.0, -1.0,
                              0.0, 2.0/fHeight, 0.0, -1.0,
                              0.0, 0.0, -1.0, 0.0,
                              0.0, 0.0, 0.0, 1.0);

  gl_Position = vPosition;
  gl_Position *= projectionMatrix;
};
{% endhighlight %}

The first three lines declare the parameters we pass in later from our code.
Then in the main function i create the *projectionMatrix*. This replace's mainly the code
`glMatrixMode(GL_PROJECTION)` which we used in OpenGL ES 1.0.
At next we assign the vertex we passed in and multiply it with the projection matrix.
This might look complicated at first, but gives a lot of flexibility and also a performance boost.

But that's not yet all. We also need a fragment shader which defines the appearance of a vertex.
In our case simple: Just a black color. This will also change later and more important get's more dynamic.
But for now that's just fine.

{% highlight c %}
void main() {
   gl_FragColor = vec4(0.0, 0.0, 0.0, 1.0);
};
{% endhighlight %}

The next two functions are helper methods only to create the shader and load the program in the GPU.

{% highlight c %}
GLuint loadShader(GLenum shaderType, const char* pSource);
GLuint createProgram(const char* pVertexSource, const char* pFragmentSource);
{% endhighlight %}

Let me focus here on the next functions, which are directly called based on the JNI exports we made earlier.
This is the last step in our boilerplate code. After we are in our platform independent game engine.

{% highlight c %}
bool setupGL(int w, int h) {
    gProgram = createProgram(gVertexShader, gFragmentShader);

    if (!gProgram) {
        return false;
    }

    gPosition = glGetAttribLocation(gProgram, "vPosition");
    gWidth = glGetUniformLocation(gProgram, "fWidth");
    gHeight = glGetUniformLocation(gProgram, "fHeight");

    glViewport(0, 0, w, h);

    gGame->OnInit((float)w, (float)h);

    return true;
}
{% endhighlight %}

This is the setup code that get's **one time** called after the OpenGL context is created.
At first we create the shader program (happens just once). *After that our code
is compiled and already loaded in the GPU.* The next lines get accessors to the parameters
in the shader code, so we can use them later. An at the end we call the `OnInit()` method of
the **Game** class (Like we do on iOS and macOS)


**Render the frame**
As on the other platforms before, we have a method that get's called once per frame.
Because we have loaded the shader before we now just activate it by call `glUseProgram(gProgram)`.
Then we just pass the width and height of the game screen to the shader, so that the shader
can calculate and use the actual projection matrix.
Last but not least we call the game engine. That's all :)
*(Note: As you see here the time is not really calculated but pass in as 1/60; we fix that later)*

{% highlight c %}
void renderFrame() {
    glUseProgram(gProgram);

    // Set width and height
    RBVector2 size = gGame->GetGamesSize();
    glUniform1f(gWidth, size.width);
    glUniform1f(gHeight, size.height);

    gGame->OnUpdate(1.0/60.0);
    gGame->OnRender();
}
{% endhighlight %}


**User input**
Last but not least, we just miss now the user input.
This is finally handled in `userInput()`. This method just converts the arguments passed from *GLTouchView.java* to the
keyboard based events we use inside of the game engine an doesn't need further explanation.


#### OpenGL calls
Because we have done many things before by setup and use a shader. This part get's now much more simpler.
To draw a rectangle we just have to implement the following code:

{% highlight c %}
void RBDrawRect(float x, float y, float width, float height, RBColor color) {
    GLfloat vertices[] = {
        x,       y+height, 0.0f, // Upper left
        x+width, y+height, 0.0f, // Upper right
        x,       y,        0.0f, // Lower left
        x+width, y,        0.0f, // Lower right
    };

    glVertexAttribPointer(gPosition, 3, GL_FLOAT, GL_FALSE, 0, vertices);
    glEnableVertexAttribArray(gPosition);
    glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
}
{% endhighlight %}

Quite similar to what we have done on iOS side :)

{% include youtubeplayer.html id="0Mj-r7JjpeE" %}

### Summary
At that point we have now a little "game engine" that is really platform independent and already runs on
**macOS**, **iOS** and **Android**. It has needed maybe more boilerplate code on every platform then you have expected,
but remember one thing: This must be done **only once** and now we can create a **variety of (2D) games** that will run on all there
platforms with almost **no change**.

> Cool, isn't it? And what's next?

**We will do exactly that: Creating some more cool games. First in 2D, later also in 3D :)**


#### Useful links
- [NDK: Installation and usage](https://developer.android.com/ndk/guides/)
- [NDK: OpenGL ES 2.0 example *hello-gl2*](https://github.com/googlesamples/android-ndk/tree/master/hello-gl2)
- [JNI, Good introduction and tutorial](https://code.tutsplus.com/tutorials/how-to-get-started-with-androids-native-development-kit--cms-27605)
- [Shaders, Getting started](https://learnopengl.com/Getting-started/Shaders)


And as always, the source can be found at [Github](https://github.com/rogerboesch/OpenGLTutorial).


- Part 1: [OpenGL - Is Apple killing it?!]({{ site.baseurl }}{% post_url 2018-07-25-opengl-on-macos %})
- Part 2: [OpenGL - The first little game on macOS!]({{ site.baseurl }}{% post_url 2018-07-31-opengl-game %})
- Part 3: [OpenGL - From macOS to iOS!]({{ site.baseurl }}{% post_url 2018-08-04-opengl-game-ios %})
- Part 4: [OpenGL - Porting to Android!]({{ site.baseurl }}{% post_url 2018-08-08-opengl-game-android %})
