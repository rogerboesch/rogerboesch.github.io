---
layout: post
title:  "SceneKit - From Zero to Hero, Part 6"
image: assets/images/scenekit-6.png
author: roro
featured: false
hidden: true
rating: 5
---

### Introduction
We have now already a working game and it's just missing some parts we have not covered so far.
So the goal of this part of the tutorial is to add them now and polish it also a little bit.


## The missing parts

### Sound

One of the most important and maybe also most underrated parts of a game is the sound. You will see and feel
the difference of part 5 to part 6 just by the fact that we will now have sound effects.
SceneKit makes it very easy to integrate them and supports also spatial sound (3D sound).
So we just add a sound effect to a given node and SceneKit makes all the handling by itself.
Like this it's easy to add sound and i do that also in a simple way in my games by create a **GameSound**
class where i implement for every sound i use in the game it's own function. With this little abstraction
i have a single point of responsibility, it's therefore easy to change and also important, the compiler is
able to check if a sound effect is there or not. But see by yourself:

{% highlight swift %}
import AudioToolbox.AudioServices

class GameSound {
private static let explosion = SCNAudioSource(fileNamed: "sounds/explosion.wav")
private static let fire = SCNAudioSource(fileNamed: "sounds/fire.wav")
private static let bonus = SCNAudioSource(fileNamed: "sounds/bonus.wav")

static func vibrate() {
  AudioServicesPlaySystemSound(kSystemSoundID_Vibrate)
}

private static func play(_ name: String, source: SCNAudioSource, node: SCNNode) {
  let _ = GameAudioPlayer(name: name, source: source, node: node)
}

static func explosion(_ node: SCNNode) {
  guard explosion != nil else { return }

  GameSound.play("explosion", source: GameSound.explosion!, node: node)
  GameSound.vibrate()
}

static func fire(_ node: SCNNode) {
  guard fire != nil else { return }

  GameSound.play("fire", source: GameSound.fire!, node: node)
  GameSound.vibrate()
}

static func bonus(_ node: SCNNode) {
  guard bonus != nil else { return }

  GameSound.play("bonus", source: GameSound.bonus!, node: node)
}
{% endhighlight %}

That's all needed to implement 3D sound effects to a SceneKit based game. Cool, isn't it?
Maybe you have noticed that i use a class called *GameAudioPlayer* instead of the *SCNAudioPlayer*.
*GameAudioPlayer* is a subclass of *SCNAudioPlayer* and implements one important thing:
Remove the sound automatically when finished playing. Besides that, all the rest is done by SceneKit itself.

{% highlight swift %}
class GameAudioPlayer : SCNAudioPlayer {
  private var _node: SCNNode!

  init(name: String, source: SCNAudioSource, node: SCNNode) {
    super.init(source: source)

    node.addAudioPlayer(self)
    _node = node

    rbDebug("GameAudioPlayer: play \(name)")

    self.didFinishPlayback = {
      rbDebug("GameAudioPlayer: stopped \(name)")

      self._node.removeAudioPlayer(self)
    }
  }
}
{% endhighlight %}

Try the game now at first and see how different it feels when there is also sound.
To add a new effect, simply add a new function and an audio file and you can use it straight away.

{% include youtubeplayer.html id="dNJqYitqGAY" %}


### Advanced HUD techniques

The second thing i want mention is a good point system and advanced animations to use in the HUD.
Whenever we reach something in our game (like fly trough a ring) or shot a plane, we get points for it.
Maybe you have seen in other games already that those points are counter up in a animation. That i have implemented
in the HUD (see *HUD.swift*) as well and as you can see it looks much more better then just show the points.
Those things (besides good gameplay of course) make the difference between a game and a **good game** at the end
and we see more of those *tricks* in part 10 of the tutorial.

{% highlight swift %}
private func changePoints(_ pointsToAdd: Int, total: Int, count: Int) {
  _totalPoints += pointsToAdd
  points = _totalPoints

  let scaling: CGFloat = 1.5

  let action = SKAction.zoomWithNode(_points, amount: CGPoint.make(scaling, scaling), oscillations: 1, duration: 0.01)
  _points.run(action)

  if count < total {
    Run.after(0.01, {
        self.changePoints(1, total: total, count: count+1)
    })
  }
}
{% endhighlight %}


### Bullets & Enemy planes

But the biggest improvement in this part is maybe the fact that we have now enemy planes and that we and also them can fire bullets!
This additional feature has needed some refactoring (something is done very often) of the current code. I've added a new class **Plane**
and and let the *Player* class be a subclass of it. Same happens with the new **Enemy** class.
Then i just moved the code which is the same for both (*Player* and *Enemy* plane) to the plane class and just let the things unique
for *Player* plane's in it.
In the *GameLevel* class (*addEnemies*) the planes are created and fly against us.

But we don't just want them fly, we also want that we are able to fire bullets and that the enemy planes can do the same against us.


#### Bullet class

You should now know all the techniques need to implement such. Yes, it will also be subclass of *GameObject* and also uses collision detection
to check if we hit an object or a bullet hit's us; the *Player* object.
Also we could add all created classes to the *_gameObjects* list. But I will use the bullets to show another aspect of game development technique:
** Re-Use**. Because we create a lot of bullets it makes sense to use this SceneKit nodes again when there state is no longer *.alive*.
For this we create an array with all bullets we fire in *GameLevel*:

{% highlight swift %}
func fireBullet(enemy: Bool, position: SCNVector3, sideDistance: CGFloat = 0, fallDistance: CGFloat = 0) -> Bullet {
  var direction : PlaneDirection = enemy ? .down : .up

  if enemy {
    direction = .down
  }

  // Search in list if we have an unused bullet
  for bullet in _bullets {
    if (bullet.state == .died && bullet.enemy == enemy) {
      // Ok, use this
      bullet.isHidden = false
      bullet.state = .initialized

      bullet.position = position
      bullet.fire(direction: direction, sideDistance: sideDistance, fallDistance: fallDistance, speed: 3.0)

      return bullet
    }
  }

  // No bullets free, so create one
  let bullet = Bullet(enemy: enemy)
  _bullets.append(bullet)
  self.rootNode.addChildNode(bullet)

  bullet.position = position
  bullet.fire(direction: direction, sideDistance: sideDistance, fallDistance: fallDistance, speed: 3.0)

  return bullet
}
{% endhighlight %}

Like this we *save* a lot of bullets and we could improve that even more, by also remove them from memory after a given time.
Exercise: Try to implement exactly that!


## Where we go from here?
A game is never really finished, you can always improve something or add a new feature. But keep in mind:

>The most important in game development (and software development in general) is to *release*.

So concentrate at first on the main (key) gameplay-feature of your game. Finish that and make it fun.
Then iterate through all the features you have in mind (or even better) your users want to see!

There is still a lot you can learn about game development in general, but with all
the content from part 1 to 6, you have a good base to start your own games.

The next and final chapter is about Augmented reality (with ARKit).

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
