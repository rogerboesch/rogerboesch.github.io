---
layout: post
title:  "SceneKit - From Zero to Hero, Part 1"
image: assets/images/scenekit-1.png
author: roro
featured: false
hidden: true
rating: 5
---

This tutorial is a companion work I do parallel on the development of my next upcoming 3D Game which uses just SceneKit and it's written entirely with Swift 3.


### For who is this Tutorial?
It's not for the absolute beginner or novice, because I assume things like Swift notation, trigonometric know-how etc.
On the other hand you don't have to learn all at the begin. Like the terrain class **RBTerrain**.
You can use it just like it is and read the code later again when you have more know-how.
The most important is that you understand the principles.


### Introduction
It's important for me to start directly with a good and solid base which can be used for real Games later. So please don't create a project using the Xcode Project wizard but start directly with the Source on GitHub. Also I don't want to build a step by step tutorial which shows each line, but always cover a chapter of the Game and introduce a new component and show how and why :)

Many have asked me to show more background informations about the SceneKit based Game I currently work on. I remember how hard it was at the begin for myself to find enough informations on it. Then on using OpenGl on iOS. But even now with SceneKit.  There are so many tutorials always show the same physic affected boxes jumping around on the screen. That's not what I want to do. I have decided to make parallel to my real Game project a subset Game tutorial which shows how to create a Game ready for the AppStore with Swift and SceneKit. Same quality and concepts as the real Game, just simpler. I will not only cover SceneKit related stuff but also how to implement GameKit and other game related frameworks. Besides best practices, design patterns and architecture, related to Game programming. I will not cover the basics about 3D programming and vector geometry, there are plenty of good sources for it. But I will show how to use it in SceneKit with Swift 3. So be aware to have Xcode 8 Beta 2 (at this time) to use it. I've decided to use already Swift 3 because in the net are so many resources even focused on Swift 1 and many have told me have problems to adapt it to newer Swift versions. So this tutorial is up to date :)


**But now let's start now!**


## Tutorial 1 - Building a Terrain
Like in real world the base of a Game is the ground, the floor, or in a our case a terrain. We will build during this tutorial a game similar to [Blue Max](https://www.youtube.com/watch?v=wsu6pmztpgY), so a classical shooter Game from the 80's, but in 3D. *(Note: My real game is not like that but use some similar concepts, you will see later)*


### The Game Skeleton
A iOS Game like every iOS App in Swift has two major components responsible for make the app working and visible:


#### 1. AppDelegate
The AppDelegate is the class instantiated and called at first when the App is  loaded. In Objective-C there is *main()* function which instantiates that class, in Swift the same is done by just add *@UIApplicationMain* in front of the class, so you will not find any *main.m* like in Objective-C.

{% highlight swift %}

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
  private var _window: UIWindow?
  private var _gameViewController: GameViewController?

  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool  {
    _gameViewController = GameViewController()
    _window = UIWindow(frame: UIScreen.main().bounds)
    _window?.rootViewController = _gameViewController
    _window?.makeKeyAndVisible()    

    return true
  }
}

{% endhighlight %}

*(Note: I personally never use storyboards or xib files, especially not in Games. It has good reasons, but i don't want start a discussion here about that, ;) )*


#### 2. Window
Every App has it's window and most of them even just 1, which uses the entire screen. For a game, the **AppDelegate** create the window in full size.


#### 3. (Game-)ViewController and View
The ViewController is responsible for load and show a view. In our case a **SCNView**, which is a SceneKit view responsible for presenting 3D content. Also the ViewController is responsible for the rotate behaviour and status bar appearance (see code in GitHub).

{% highlight swift %}
class GameViewController: UIViewController {
  var _sceneView: SCNView?
  var _level: TutorialLevel1?`

  override func viewDidLoad() {
    super.viewDidLoad()

   _level = TutorialLevel1()
   _level!.create()
   _sceneView = SCNView()
   _sceneView!.scene = _level
   _sceneView!.allowsCameraControl = false
   _sceneView!.showsStatistics = true
   _sceneView!.backgroundColor = UIColor.black()
   _sceneView!.debugOptions = .showWireframe
   self.view = _sceneView
}
{% endhighlight %}

So that's mainly the entire Game skeleton, means the base for every App and Game. Use this as a base for every Game and forget again for now, because it's always the same ;) From here we start with the game itself...

I never put in Game logic in the ViewController but just create a **SCNScene** class and attach it to the *SCNView*. Separation of code is always a good approach, not only in Game programming.


### The Game

The Game itself it's implemented in a child class of *SCNScene* (like written before). Because we create later different levels of the Game, I name this new class *TutorialLevel1*. The beauty of Swift and SceneKit is the shortness of code you need to realise a Game. But let us take a look now, what we do in that **GameLevel** class.
Mainly it's the place to create all the Game objects and control them. So implement the gameplay and logic of our 3D shooter Game.

At first we have just two Game objects: The terrain and a player class, while the focus of tutorial 1 is the terrain class. But we implement a simple player class already to "ride" over the terrain and extend it later. But first to the terrain class.


#### Terrain class - RBTerrain.swift
I have created a basic class called *RBTerrain* which creates a 3D landscape in any size and style. So you can use it in your Game or customise it the away you want. The usage it's simple:

{% highlight swift %}
_terrain = RBTerrain(width: 320, length: 320, scale: 128)
{% endhighlight %}

This creates the terrain class and defines it's size and scale. The scale is used to define how high the hills of the landscape will gone. Play a little bit around with this parameter to see the effect. The next lines are more interesting:

{% highlight swift %}
let generator = RBPerlinNoiseGenerator(seed: nil)
_terrain?.formula = {(x: Int32, y: Int32) in
  return generator.valueFor(x: x, y: y)
}
{% endhighlight %}

So let's take a look at them. The main essence is to specify a formula which generates the "hills" of the terrain, because the terrain class itself just creates a grid of meshes, which can have any form. This is done by use **closures**. If you are not familiar with it, read on this first, because it's in all Swift related programming important. If you have used Objective-C before, they are called *blocks*. The approach of the terrain closure it's simple. The terrain class calls this function and expect for every *x,y* coordinate of the grid (mesh) a height coordinate. So you can give any value here. It's also a good place to play at first a little bit with the terrain class. But to build a real terrain with hills we use a method called **Perlin noise**. It's an algorithm which comes not from me of course and it's described in [Wikipedia](https://en.wikipedia.org/wiki/Perlin_noise) in detail. I have just made a Swift version out of existing C code. But you can also *forget* that class for a moment and just use it or write your own formula. An interesting approach for example could be to build the terrain based on a image you have... In any way, the terrain class can always keep untouched, just create another formula and assign it. Before we go deeper in the terrain class, i want talk about the other Game component, the *Player* class.


#### The Player class
The player class looks at first a little bit complicated. But at the end it's easy. It's (like the terrain class) derived from SCNNode, but contains then some child nodes that supports camera, light and the player node itself. I talk soon about camera and lights but for now keep in mind:
>Always use child nodes in a *SCNNode* for a game object

Why? You will see later, but in easy words, to manipulate them individual from each other. For example when you rotate your player, you don't want necessary rotate also your camera. With this approach you can separate all actions.


#### The Camera
The camera is an important object in 3D programming. Its essential a virtual object which describes what is currently visible from your 3D world. So it's like in real world. Depending on where and how you position your camera, you see individual parts of the 3D world. But why I added the camera to the player? In most of the other examples in the net, you see that the camera is attached to the scene. But in the Game we made, I want the camera is always follow the player object and that's an easy way to do so. So when we move the player, the camera follows. We see then in next tutorial, how to manipulate the camera more, but for now we have what we want.


#### The Lights
To understand lights in detail needs some more deeper know-how, but for us, it's like in real world. We add some light bulbs to our scene, so we can simulate sun, or add spot lights to show more detail and make shadows. Also the light I have attached to the player because I want that the light is always where the player is. We will talk more on lights after in the next chapters of this series.


#### Details of the Terrain class
As promised before, I will go now in more detail in the terrain class. *SceneKit* provides a lot of primitives like boxes, cylinders etc. but in real 3D games, we must mostly create the objects by ourself or with a 3D program like *Blender*, which I use for the Game objects. Also a terrain can be created with programs like blender, but we do that programmatically to be flexible in size and appearance by using the *Perlin Noise* formula.
When you look in the *RBTerrain* class there is one function which do all the work:

{% highlight swift %}
private func createGeometry() ->SCNGeometry
{% endhighlight %}

SceneKit gives us the possibility to create to create 3D objects by vectors and assign material to it later *(Like in OpenGL before)*. Thats exactly what we do here. We create a grid in the size given. A list of vertices defines that grid:

{% highlight swift %}
for y in 0...Int(h-2) {
  for x in 0...Int(w-1) {
  vertices[vertexCount] = bottomLeft
  vertices[vertexCount+1] = topLeft
  vertices[vertexCount+2] = topRight
  vertices[vertexCount+3] = bottomRight
  vertexCount += 4
}
{% endhighlight %}

This is then used to create a SceneKit geometry source:

{% highlight swift %}
let source = SCNGeometrySource(vertices: vertices, count: vertexCount)
{% endhighlight %}

Unfortunately this is not enough to create a custom 3D objects. Essentially it needs three data lists more:

**The geometry list:** Essentially a list of indices described how the vertices must be used

{% highlight swift %}
let geometryData = NSMutableData()
var geometry: CInt = 0
while (geometry < CInt(vertexCount)) {
  let bytes: [CInt] = [geometry, geometry+2, geometry+3, geometry, geometry+1, geometry+2]
  geometryData.append(bytes, length: sizeof(CInt.self)*6)
  geometry += 4
}

let element = SCNGeometryElement(data: geometryData as Data, primitiveType: .triangles, primitiveCount: vertexCount/2, bytesPerIndex: sizeof(CInt.self))
{% endhighlight %}

**The texture list:** Defines how textures (images) get applied to the geometry later

{% highlight swift %}
let uvData = NSData(bytes: uvList, length: uvList.count *  sizeof(vector_float2.self))
    let uvSource = SCNGeometrySource(data: uvData as Data,
            semantic: SCNGeometrySourceSemanticTexcoord,
            vectorCount: uvList.count,
            floatComponents: true,
            componentsPerVector: 2,
            bytesPerComponent: sizeof(Float.self),
            dataOffset: 0,
            dataStride: sizeof(vector_float2.self))
{% endhighlight %}


**The list of normals:** Defines how light is applied to the object

{% highlight swift %}
for normalIndex in 0...vertexCount-1 {
  normals[normalIndex] = SCNVector3Make(0, 0, -1)
}
sources.append(SCNGeometrySource(normals: normals, count: vertexCount))
{% endhighlight %}

Uff! That's all then and the creation of the terrain, it's now just one statement:

{% highlight swift %}
_terrainGeometry = SCNGeometry(sources: sources, elements: elements)
let terrainNode = SCNNode(geometry: createGeometry())
{% endhighlight %}

Now we have a **SCNGeometry** and create a *SCNNode* out of it, that we can use like the standard primitives in SceneKit.

**And now have fun!**

Here is the most important:
[The link to the tutorial source at GitHub](https://github.com/rogerboesch/SceneKitTutorial)

Tutorial Chapters:
- **[Part 1 - Building a Terrain]({{ site.baseurl }}{% post_url 2017-07-15-scenekit-zerotohero-I %})**
- **[Part 2 - Create a real player game object]({{ site.baseurl }}{% post_url 2017-10-26-scenekit-zerotohero-II %})**
- **[Part 3 - Add life to your terrain]({{ site.baseurl }}{% post_url 2018-12-23-scenekit-zerotohero-III %})**
- **[Part 4 - Implement a Game Loop]({{ site.baseurl }}{% post_url 2018-12-26-scenekit-zerotohero-IV %})**
- **[Part 5 - Fly smoothly with CoreMotion]({{ site.baseurl }}{% post_url 2018-12-27-scenekit-zerotohero-V %})**
- **[Part 6 - Finish the game (The missing parts)]({{ site.baseurl }}{% post_url 2018-12-29-scenekit-zerotohero-VI %})**
- **Part 7 - ARKit: Play in your environment (Coming in June 2020)**


Other good resources with some basic infos are:

SceneKit:
- [https://developer.apple.com/videos/play/wwdc2014/610/](https://developer.apple.com/videos/play/wwdc2014/610/)
- [https://developer.apple.com/videos/play/wwdc2016/609/](https://developer.apple.com/videos/play/wwdc2016/609/)
- [https://itunes.apple.com/us/book/3d-graphics-with-scene-kit/id936235049?ls=1&mt=11](https://itunes.apple.com/us/book/3d-graphics-with-scene-kit/id936235049?ls=1&mt=11)

Vector geometry:
- [http://physics.about.com/od/mathematics/a/VectorMath.htm](http://physics.about.com/od/mathematics/a/VectorMath.htm)
