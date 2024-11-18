---
layout: post
title:  "SceneKit - From Zero to Hero, Part 5"
image: assets/images/scenekit-5.png
author: roro
featured: false
hidden: true
rating: 5
---

### Introduction
In the previous parts we have laid out a solid base to build a game using SceneKit and SpriteKit.
Sure, it's still rough and also the graphics could be improved, but that's secondary.
The most important is to become something playable. Make improvements is the easier part.


## Motion control
One improvement we will make now, to make the game more fun and playable.
It's something we have left out so far: **User input**.

The possibility to fly the plane more realistic. Sure we can steer it left or right with a swipe, but that doesn't feel
realistic for a plane. This is where **CoreMotion** comes into the game:

>Core Motion reports motion- and environment-related data from the iPhone. It includes data from the accelerometer, gyroscope, pedometer and magnetometer
and you can use this framework to access hardware-generated data to control a game.


### Add CoreMotion to the game
The steps to include CoreMotion in our game are not very complicated and explained below line by line.
The first change must be made in *GameViewController.swift*. Maybe you ask yourself why I don't do that directly
in the *GameLevel* class. Technically we could do this, but i suggest always to put device related things
in the *GameViewController* to keep the game itself free from any platform depending code.
This makes it much more easier when we port the game later to macOS or even tvOS.
But let's start now:

{% highlight swift %}
import CoreMotion
{% endhighlight %}

At first we import CoreMotion, so it's available in our project.
Then I've added three member variables. One for **CMMotionManager** itself and the other
two to store the start data (more on this later) and the current data of the accelerometer.

{% highlight swift %}
private var _motionManager = CMMotionManager()
private var _startAttitude: CMAttitude?
private var _currentAttitude: CMAttitude?    
{% endhighlight %}

Then we can start CMMotionManager with the following lines of code:

{% highlight swift %}
private func setupMotionHandler() {
  if (_motionManager.isAccelerometerAvailable) {
    _motionManager.accelerometerUpdateInterval = 1/60.0

    _motionManager.startDeviceMotionUpdates(to: OperationQueue.main, withHandler: {(data, error) in
        self.motionDidChange(data: data!)
    })
  }
}
{% endhighlight %}

This code should be mostly self explanatory, but in easy words: We check if an accelerometer device is available
and if so, we call *startDeviceMotionUpdates()* and assign a handler (Swift block). This block get's now called at a frequency
of 1/60.0 second and gives us the accurate data. Because we are not interested on the absolute values, but on the difference
to a given start value (How much the user roll or pitch the iPhone), we save both values. The *_startAttitude* is therefore
a reference value which we save shortly before the player starts to get movements he made after related to this.
All this is done in *motionDidChange()*:

{% highlight swift %}
private func motionDidChange(data: CMDeviceMotion) {
  _currentAttitude = data.attitude

  guard _level != nil, _level?.state == .play else { return }

  // Up/Down
  let diff1 = _startAttitude!.roll - _currentAttitude!.roll

  if (diff1 >= Game.Motion.threshold) {
    _level!.motionMoveUp()
  }
  else if (diff1 <= -Game.Motion.threshold) {
    _level!.motionMoveDown()
  }
  else {
    _level!.motionStopMovingUpDown()
  }

  let diff2 = _startAttitude!.pitch - _currentAttitude!.pitch

  if (diff2 >= Game.Motion.threshold) {
    _level!.motionMoveLeft()
  }
  else if (diff2 <= -Game.Motion.threshold) {
    _level!.motionMoveRight()
  }
  else {
    _level!.motionStopMovingLeftRight()
  }
}
{% endhighlight %}

With the previous explanation it's now easy to read this. We are interested on the *roll* parameter to
fly up and down (YES! This will now also be possible) and on the *pitch* parameter to fly left or right.
So we have now the data we need and just call some new methods in *GameLevel*, do tell
the plane in which direction we want to fly.
Here you see now also the abstraction we do. *GameLevel* itself will never know from where the data is coming.
So like this we can use whatever we like to steer the plane without ever change the game.


### Adapt in Player class

Of course this was just the half of the story. The plane must now also be able to react on this commands.
Btw. *GameLevel* itself just passes the command over to the *Player* class, so we focus on this.
Because all four movements work the same way i explain it once based on *moveUp*.
As you have seen in *motionDidChange()* we call two methods:

1. *motionMoveUp()* or *motionMoveDown()* when the difference is >= 0.2 (threshold value)
2. *motionStopMovingUpDown()* when the difference is < 0.2

This is important to understand the concept of moving. With the first step we actually start internally an *SCNAction()*
which makes this movement and with the second step (When the user rolls the Phone back to the start position), we delete this action.
That's the entire secret and in code it looks like this:

{% highlight swift %}
func moveUp() {
  let oldDirection = _upDownDirection

  if _upDownDirection == .none {
    let moveAction = SCNAction.moveBy(x: 0, y: Game.Player.upDownMoveDistance, z: 0, duration: 0.5)
    self.runAction(SCNAction.repeatForever(moveAction), forKey: "upDownDirection")

    _upDownDirection = .up
  }
  else if (_upDownDirection == .down) {
    self.removeAction(forKey: "upDownDirection")

    _upDownDirection = .none
  }

  if oldDirection != _upDownDirection {
    adjustCamera()
  }
}
{% endhighlight %}

and

{% highlight swift %}
func stopMovingUpDown() {
  let oldDirection = _upDownDirection

  self.removeAction(forKey: "upDownDirection")
  _upDownDirection = .none

  if oldDirection != _upDownDirection {
    adjustCamera()
  }
}
{% endhighlight %}

Easy, isn't it? This we do now for all four directions and we have an almost perfect flight behaviour of the plane.
In fact, this would be enough to play the game, but as always, we will go the extra mile to make it look even more better.
One way to make it look more professional is using smooth camera movements. This is a technique using heavily in Hollywood's
blockbusters and so we also do it... Whenever we change the direction we move the camera position with a delay behind the player.
For this we use *SCNTransaction.begin()* and *SCNTransaction.end()*, set an animation time with *SCNTransaction.animationDuration = 1.0*
and every change we do in the middle will now be animated. Looks cool, right?

The second thing we do is rotating the plane slightly when move in any direction, so it looks more naturally. This happens in the
*update()* method of the *Player* class and looks almost the same as for the camera:

{% highlight swift %}
SCNTransaction.begin()
SCNTransaction.animationDuration = 1.0

_playerNode?.eulerAngles = SCNVector3(eulerX, 0, eulerZ)

SCNTransaction.commit()
{% endhighlight %}

That's all. We have now a perfect plane and the only thing I have done on top of this is to control (also in the *update()* method) the
maximum the plane is allowed to fly up, down, left or right.


{% include youtubeplayer.html id="3x3TSwcYmNE" %}


## Some smaller things

I hope you enjoyed this chapter as well. It was the last (conceptual) step which was really needed to make a real game out
of our first *Hello World* tutorial which we have had at the begin. But don't worry, there are another 5 chapters coming,
with even more interesting aspects of SceneKit. Also we will see how we can use it on the other (Apple) platforms.

Besides this main change of using CoreMotion to control the plane, i have also added some smaller improvements at various places in the project.


#### Ring positions

We can now also fly up and down, so a logical step was to position the rings on different heights.
Like this, the game becomes much more difficult and fun to play. Also you will notice that the rings have now numbers.
Just a visual effect, but nice to see and you could spend some time to make look this even more polished.


#### Terrain
I have changed also a bit the look of the terrain by make it more wavy. Then added some additional lights and changed the current
and added Fog to the scene. With just this three parameters the game looks far more better then before and you should play around
with this effects to see what's possible!


#### Game settings
Like said in an earlier chapter, i prefer to have all game parameters in one settings class (*see GameSettings.swift*).
It's easier to make changes to the gameplay and it even makes it simple after to save and load them from a file to
to change this parameters without compile the game again.


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
