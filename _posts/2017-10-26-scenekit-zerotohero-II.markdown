---
layout: post
title:  "SceneKit - From Zero to Hero, Part 2"
image: assets/images/scenekit-2.png
author: roro
featured: false
hidden: true
rating: 5
---

In the first part of this tutorial, we have created the base for our game and as main part a (simple) *Terrain* class to build an interesting game arena without too much of effort.
In this second part we bring now our *Player* class alive by adding a model, create the needed animations and allow the user, to control it.

## The Player game object

### Add a model
At first we need a 3D model which we use for this tutorial. Apple provides a nice starship in it's game template, which we want also use here. The easiest way to do this is:

- Get the code from part 1 of this tutorial and clone it to your disk
- Create a new and empty game project
- Be sure that you use "SceneKit" as game technology
- Drag&Drop the files *ship.scn* and *texture.png* to the *art.scnassets* folder in our project

Your project should now look something like this:

![Screenshot 1]({{"/assets/images/zerotohero-II-1.png"}})

The scene file contains the spaceship we add now to our *Player* class in *Player.swift*. Remove at first the following lines in the `init()` initializer:

{% highlight swift %}
let cubeGeometry = SCNBox(width: 0.5, height: 0.5, length: 0.5, chamferRadius: 0.0)
_playerNode = SCNNode(geometry: cubeGeometry)
_playerNode?.isHidden = true
addChildNode(_playerNode!)

let colorMaterial = SCNMaterial()
cubeGeometry.materials = [colorMaterial]
{% endhighlight %}

And add the following code:

{% highlight swift %}
let scene = SCNScene(named: "art.scnassets/ship.scn")
if (scene == nil) {
    fatalError("Scene not loaded")
}

_playerNode = scene!.rootNode.childNode(withName: "ship", recursively: true)
if (_playerNode == nil) {
    fatalError("Ship node not found")
}

_playerNode!.scale = SCNVector3(x: 0.25, y: 0.25, z: 0.25)
self.addChildNode(_playerNode!)
{% endhighlight %}

What happens here? Well in easy words, we load the scene file (a scene file can contain different objects or even an entire level) and search for a node called **ship**. Like before the cube, we assign now the ship node to our *_playerNode* variable and add it as child node to the scene. Because the spaceship is to big, relative to our terrain, we *shrink* it by set the scale to 0.25, means we make it 4 times smaller.

**Now let the code run!**

![Screenshot 2]({{"/assets/images/zerotohero-II-2.png"}})

*Try on a real device to have a good frame rate. Looks already nice, not?*


### Let it fly

But of course like this, the game would be a little bit boring ;). So we add now the possibility to control the spaceship. To keep it simple, we add two gestures. One to fly left and one to fly right. Of course in a real game you would add something more fancy like motion control (See part 5) or even a game controller. But to show the technique, swipe gestures are just fine. Add two of them in the *GameViewController* class

{% highlight swift %}
let swipeLeftGesture = UISwipeGestureRecognizer(target: self, action: #selector(handleSwipe))
swipeLeftGesture.direction = .left
_sceneView!.addGestureRecognizer(swipeLeftGesture)

let swipeRightGesture = UISwipeGestureRecognizer(target: self, action: #selector(handleSwipe))
swipeRightGesture.direction = .right
_sceneView!.addGestureRecognizer(swipeRightGesture)
{% endhighlight %}

And also add a new function called *handleSwipe()*. Like this:

{% highlight swift %}
func handleSwipe(_ gestureRecognize: UISwipeGestureRecognizer) {
    if (gestureRecognize.direction == .left) {
    }
    else if (gestureRecognize.direction == .right) {
    }
}
{% endhighlight %}

Now we let the *Player* class know that we want fly to the left and right. There are many ways to do that, but to make a good separation of code, i prefer to have the user input handling in the *GameViewController* class and the logic of what we do with it in the *GameLevel* class. Like that the code is more universal and easier to change or extend.
That means, we add the following statements to the *handleSwipe()* function:


{% highlight swift %}
func handleSwipe(_ gestureRecognize: UISwipeGestureRecognizer) {
    if (gestureRecognize.direction == .left) {
        _level.swipeLeft()
    }
    else if (gestureRecognize.direction == .right) {
        _level.swipeRight()
    }
}
{% endhighlight %}

And of course add the functions *swipeLeft()* and *swipeRight()* to the *GameLevel* class and call there the new  functions `moveLeft()` and `moveRight()` in the *Player* class:


{% highlight swift %}
func swipeLeft() {
    _player!.moveLeft()
}
func swipeRight() {
    _player!.moveRight()
}
{% endhighlight %}

Like this, we can easily change it later, when we want to use the swipe gesture for something else or when we want use a different *Player* class.

But now to the real work. We let the starship fly left and right. With the help of animations, this is also easily done.


{% highlight swift %}
func moveLeft() {
    let moveAction = SCNAction.moveBy(x: 2.0, y: 0.0, z: 0, duration: 0.5)
    self.runAction(moveAction, forKey: "moveLeftRight")
}

func moveRight() {
    let moveAction = SCNAction.moveBy(x: -2.0, y: 0.0, z: 0, duration: 0.5)
    self.runAction(moveAction, forKey: "moveLeftRight")
}
{% endhighlight %}

Try it out! We can now move the starship to the right or to the left. Ok, not yet that fancy but we work on it. To make it more realistic, we let the starship also rotate a little bit...

{% highlight swift %}
func moveLeft() {
    let moveAction = SCNAction.moveBy(x: 2.0, y: 0.0, z: 0, duration: 0.5)
    self.runAction(moveAction, forKey: "moveLeftRight")

    let rotateAction1 = SCNAction.rotateBy(x: 0, y: 0, z: -degreesToRadians(value: 15.0), duration: 0.25)
    let rotateAction2 = SCNAction.rotateBy(x: 0, y: 0, z: degreesToRadians(value: 15.0), duration: 0.25)

    _playerNode!.runAction(SCNAction.sequence([rotateAction1, rotateAction2]))
}

func moveRight() {
    let moveAction = SCNAction.moveBy(x: -2.0, y: 0.0, z: 0, duration: 0.5)
    self.runAction(moveAction, forKey: "moveLeftRight")

    let rotateAction1 = SCNAction.rotateBy(x: 0, y: 0, z: degreesToRadians(value: 15.0), duration: 0.25)
    let rotateAction2 = SCNAction.rotateBy(x: 0, y: 0, z: -degreesToRadians(value: 15.0), duration: 0.25)

    _playerNode!.runAction(SCNAction.sequence([rotateAction1, rotateAction2]))
}
{% endhighlight %}

Like this, it already looks a little bit more better?! Now it becomes also visible why we have in the *Player* class not just one node, but many. As you can see, the movement of the starship is done on the *Player* class itself, while the side rotation is just applied on the child node *_playerNode*. Why this? Because otherwise we also would rotate the camera, when we rotate the starship. This can be wanted, but in our case not.

The next thing we wanna do is improving the camera, so that it looks more cool, when we move the starship.

### Cool camera handling

With the camera node you can do some really nice effects, esp. when used together with animations. That's the second thing in this tutorial i want show you. For that we change at first our camera position a little bit:

{% highlight swift %}
private let lookAtForwardPosition = SCNVector3Make(0.0, -1.0, 6.0)
private let cameraFowardPosition = SCNVector3(x: 5, y: 1.0, z: -5)
{% endhighlight %}

The result is that we are more behind the starship and also more left to it. Like this, the effect we now add, looks even more better. We will move the camera with a delay, depending on which direction we move. This is done the same way like on any other node. We just change the camera node's position. But to make this smoothly. We do this inside of a **SCNTransaction**, to animate this change:

{% highlight swift %}
private func toggleCamera() {
    var position = _cameraNode!.position

    if position.x < 0 {
        position.x = 5.0
    }
    else {
        position.x = -5.0
    }

    SCNTransaction.begin()
    SCNTransaction.animationDuration = 1.0

    _cameraNode?.position = position

    SCNTransaction.commit()
}
{% endhighlight %}

And we call this function inside of the moveLeft() and moveRight() function. Easy, not? Of course this is a simple implementation of all, but the mechanism remains the same. As an exercise, try to implement a 180 degree turn. The code for this i will show in the next tutorial, when we talk about (good) gameplay...

![Screenshot 3]({{"/assets/images/zerotohero-II-3.png"}})


Tutorial Chapters:
- **[Part 1 - Building a Terrain]({{ site.baseurl }}{% post_url 2017-07-15-scenekit-zerotohero-I %})**
- **[Part 2 - Create a real player game object]({{ site.baseurl }}{% post_url 2017-10-26-scenekit-zerotohero-II %})**
- **[Part 3 - Add life to your terrain]({{ site.baseurl }}{% post_url 2018-12-23-scenekit-zerotohero-III %})**
- **[Part 4 - Implement a Game Loop]({{ site.baseurl }}{% post_url 2018-12-26-scenekit-zerotohero-IV %})**
- **[Part 5 - Fly smoothly with CoreMotion]({{ site.baseurl }}{% post_url 2018-12-27-scenekit-zerotohero-V %})**
- **[Part 6 - Finish the game (The missing parts)]({{ site.baseurl }}{% post_url 2018-12-29-scenekit-zerotohero-VI %})**
- **Part 7 - ARKit: Play in your environment (Coming in June 2020)**


And not to forget...
[The link to the source code again](https://github.com/rogerboesch/SceneKitTutorial)
