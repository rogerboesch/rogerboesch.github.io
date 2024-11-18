---
layout: post
title:  "SceneKit - From Zero to Hero, Part 4"
image: assets/images/scenekit-4.png
author: roro
featured: false
hidden: true
rating: 5
---

### Introduction
In the first three parts of this tutorial, we've used a straight forward method to let the game run by just using *SCNAction*.
This changes now. In professional game development there is such a thing called **game loop**. While SceneKit itself has a slightly different
approach we can also implement a game loop. Therefore we change in this chapter of the tutorial the architecture and design of the game
slightly and come closer to what is needed for a *real game*.


## Game Loop

What is a game loop? The definition is simple:

> A game loop runs continuously during gameplay. Each turn of the loop, it processes user input without blocking, updates the game state, and renders the game. It tracks the passage of time to control the rate of gameplay.

Modern graphic UI applications are surprisingly similar to this. Your application just waits there doing nothing, until you press a key or click something:

{% highlight swift %}
while (true) {
  Event* event = waitForEvent()
  dispatchEvent(event)
}
{% endhighlight %}

A game loop looks in the simplest possible form like this:

{% highlight swift %}
while (true) {
  processInput()
  update()
  render()
}
{% endhighlight %}

SceneKit does that already for us and allows at several time during a frame to hook in by set a delegate and implement
the relevant methods.

![SceneKit Game Loop](https://docs-assets.developer.apple.com/published/beef028d16/1b2e406d-c49e-4d11-9888-850579f5f8ab.png)

In our game we use

{% highlight swift %}
func renderer(_ renderer: SCNSceneRenderer, didSimulatePhysicsAtTime time: TimeInterval)
{% endhighlight %}

As soon as you use physics
in your game, it's important to **not** hook in before in the game loop (called render loop in SceneKit). Otherwise you will result in strange
behaviour. Also important to know is:

>SceneKit just call all this methods once per frame if there is something to do.

That means, if all actions are done and nothing changed between 2 frames, SceneKit will stop calling it.
Our game loop (defined in *GameViewController.swift*) looks therefore like this:

{% highlight swift %}
func renderer(_ renderer: SCNSceneRenderer, didSimulatePhysicsAtTime time: TimeInterval) {
    if _level != nil {
      _level.update(atTime: time)
    }

    renderer.loops = true
}
{% endhighlight %}

It delegates the work to *GameLevel* where we do all the important things.
Let's examine together the steps we do there.

{% highlight swift %}
func update(atTime time: TimeInterval) {
  if self.state != .play {
    return
  }
{% endhighlight %}

That's the first important thing introduced in this chapter: **Game states**.
A game is working mostly like a state machine while the different game statements
are defined in  an enum as follows:

- *initialized*: The game (level class) is just created but not yet started
- *play*: The game is currently played
- *win*: The game has ended by win the game
- *loose*: The game has ended by loose the game
- *stopped*: The game has stopped (all actions terminated)

Why we do that? Well, obviously we have to do different things in our game loop
during the different states of the game. The simplest way to implement this is by introducing
a state variable. Of course state machines can be much more complex, but for our
tutorial, this is good enough.

{% highlight swift %}
var missedRings = 0

for object in _gameObjects {
  object.update(atTime: time, level: self)

  // Check which rings are behind the player but still 'alive'
  if let ring = object as? Ring {
      if (ring.presentation.position.z + 5.0) < _player!.presentation.position.z {
          if ring.state == .alive {
              missedRings += 1
          }
      }
  }
}
{% endhighlight %}

This is an interesting part in our game loop and new in this chapter. We don't just count the
rings we fly trough, but also the one's we missed. This is a little bit more complex then
counting the rings we fly trough, because SceneKit doesn't help us on this.

At first you see an array of game objects which is new.

{% highlight swift %}
private var _gameObjects = Array<GameObject>()
{% endhighlight %}

In this we hold **all** the the game objects (like rings, the player and the new handicaps),
so we can access them easily.
To see if we passed a ring without fly trough, we just compare the z-coordinate of the object
with the z-coordinate of the player. If it's smaller then we have already passed it.
In addition to that we also check the state (see later) to see if the ring is still *alive*
(mean's not already fly trough). This we do in every frame and if the number of missed rings
is different then in the previous frame, we update the HUD.

{% highlight swift %}
if missedRings > _missedRings {
  _missedRings = missedRings
  _hud?.missedRings = _missedRings
}
{% endhighlight %}

Also we test here if the game is finished. Before, we just ended the game, when the
*SCNAction* was ended for the player object. Now we use a specific criteria:
Our game ends, when we passed all rings (either by fly trough or missed them).
If that happens, we change the game state and show a message to the user.

{% highlight swift %}
// Test for end of game
if _missedRings + _touchedRings == _numberOfRings {
  if _missedRings < 3 {
      _hud?.message("YOU WIN", information: "- Touch to restart - ")
  }
  else {
      _hud?.message("TRY TO IMPROVE", information: "- Touch to restart - ")
  }

  self.state = .win
}
{% endhighlight %}

That's the entire secret behind a game loop and implementing the gameplay.
Of course we can extend this by many more criteria's and states but the logic remains the same.

One topic we have not touched so far in the gameplay. We now have also a criteria
that changes the game state, but happens outside of the game loop.

{% highlight swift %}
func touchedHandicap(_ handicap: Handicap) {
    _hud?.message("GAME OVER", information: "- Touch to restart - ")

    self.state = .loose
 }
 {% endhighlight %}

Whenever we touch (*crash into*) a handicap object, the game ends immediately.
Immediately? Not really... As you see we just show a message to the user and change the
game's state to **loose**. That means, in the **next** frame we no longer do the
work described before, because `self.state != .play`.
This concept is also important to understand.

> Never ever do something directly in an action or as a result outside of the game loop

Just change the state of the game and do it as part of the game loop later.
Otherwise it results in graphics, physics and/or multi-threading issues.


## The Game Object

In the previous topic we covered already the most important things happen in
this part of the tutorial. But there is one more thing: The **GameObject**
This new class is not *spectacular* by itself, but it's an important concept in game
development. With this we have a common base class for **all** kind of game objects
and therefore we can store them all in one list and iterate through them in every frame.
Also we have now a state not just for the game itself, but also for every game object.

- initialized: The game object was just created
- alive: The game object is alive
- died: The game object is dead, killed, touched (Whatever you want call it)
- stopped: And last but not least stopped (All actions and resource's free'd up)

Of course there can be many more states. For example an *attack* state (When the player is in fight mode)
or *idle* (When an object just waits). But i hope you get the point.
Each game object can act now independent of another by having it's own state and
therefore behaviour.


### Some more details

The **game loop** and the (base class) *GameObject* are the most important things
in this chapter. But like always, there are some small things i have not covered so far


#### The Handicap class

While we have already talked about this class in the game loop topic, i have not described
how it works. For a good reason: It's also a derived class from *GameObject* and it behaves like the rings.
With one important difference: The game end's whenever the player touch it.
But you should now be able to understand the concept and also add your own game objects to the game.


#### HUD extensions

The HUD is essentially the same like in the previous chapter, but we show now more
informations in it. Also we animate those informations whenever they change.


#### Skybox

As you can see there is another code line in *GameLevel.swift* like this:

{% highlight swift %}
self.background.contents = UIImage(named: "art.scnassets/skybox")
{% endhighlight %}

SceneKit allows us to to set an image which is used as a background. But a background
that behaves like a real sky. That means, looks different depending in which direction we look.


#### Shared classes

To simplify the code in the tutorial, i have also added some extensions and classes
to the project. I use them by myself in different games to simplify things when programming.

- *RB+CGPoint.swift*: Some additional mathematics when working with points
- *RB+SKAction.swift*: Some nice actions used in HUD (will be extended later)
- *RBLog.swift*: See below


#### Logging

One simple class i added also is the logging class **RBLog**. It's more or less a wrapper
around the *print* function in Swift. It's main purpose is to show some informations in the console.
That's helpful when test the gameplay by showing the games entire behaviour in a sequence order.
Something which can be hard (or impossible) to see when debug.
I use something similar in my projects, but with more functionality, which i will add later also
to this tutorial.


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
