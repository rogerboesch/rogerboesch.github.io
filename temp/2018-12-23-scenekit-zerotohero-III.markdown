---
layout: post
title:  "SceneKit - From Zero to Hero, Part 3"
image: assets/images/scenekit-3.png
author: roro
featured: false
hidden: true
rating: 5
---

In the first two parts of this tutorial, I have created the base for our game and as main part a (simple) *Terrain* class to build an interesting game arena without too much of effort. Also I have created a *Player* class, the needed animations and the swipe gestures which allow the user, to control the plane.

## Add life to the game

### Gameplay - The essence of a game
Well, so far we don't have had a real game, but just a simple base to show the fundamental stuff of SceneKit. The main thing we missed so far is what
we call **gameplay**. This mens a goal or playing-mechanics, which give the game it's purpose and let it become a game.
In part 3 of the tutorial, this happens now :)
To make it more interesting, i've added rings to the scene, where we can fly trough and which gives us a point every time we do so.
Also we have not had any possibility to show some informations to the user; in our case the number of rings we fly trough.

## Game objects

### Rings

The **Ring** class is also a *SCNNode* based class. This time we don't use a model, but one of the primitives available in SceneKit:
A *SCNSphere*. To make it more attractive we apply a texture on it and let it rotate.
But let us look at the code:

{% highlight swift %}
let ringMaterial = SCNMaterial()
ringMaterial.diffuse.contents = UIImage(named: "art.scnassets/ringTexture")
ringMaterial.diffuse.wrapS = .repeat
ringMaterial.diffuse.wrapT = .repeat
ringMaterial.diffuse.contentsTransform = SCNMatrix4MakeScale(3, 1, 1)
{% endhighlight %}

At first I create a material for the sphere. Materials are the way to give nodes a look and feel.
We can either apply colors or in this case an image, called textures. Textures are a common way to make
objects look more detailed. SceneKit makes it easy to use them as you see. The last 3 statements are needed to make
it better looking. We don't just use the texture once but let it repeat, so we get a more cubic pattern.
Play a little bit around with the parameters in `SCNMatrix4MakeScale(3, 1, 1)` to understand the effect.

{% highlight swift %}
let ring = SCNTorus(ringRadius: 5.0, pipeRadius: 0.5);
ring.materials = [ringMaterial]
let ringNode = SCNNode(geometry: ring)
self.addChildNode(ringNode)
{% endhighlight %}

After we have created the ring, the material is applied to the node which is also added to the scene.
That's all. With this we have a ring we can now place anywhere in our game scene.

{% highlight swift %}
ringNode.eulerAngles = SCNVector3(degreesToRadians(value: 90), 0, 0);
{% endhighlight %}

By default a sphere is placed vertically in the scene. To make it possible to fly trough, we apply a rotation by 90 degrees around the
x-coordinate of the ring. *Note: Like most of the 3D API's (and so also SceneKit) radians are used instead of degrees. So whenever we use it,
we must convert it from degrees to radians.*

{% highlight swift %}
let action = SCNAction.rotateBy(x: 0, y: 0, z: degreesToRadians(value: 360), duration: 3.0)
ringNode.runAction(SCNAction.repeatForever(action))
{% endhighlight %}

To make it even more nicer I let the ring rotate. The rest of the code below creates another node.
We want to check if the plane flies through the ring. So we place an invisible node in the middle and
attach a physics body to it. More about the physics engine in SceneKit and how it's exactly used read below.

{% highlight swift %}
// Contact box
let boxMaterial = SCNMaterial()
boxMaterial.diffuse.contents = UIColor(red: 1.0, green: 0.0, blue: 0.0, alpha: 0.0)

let box = SCNBox(width: 10.0, height: 10.0, length: 1.0, chamferRadius: 0.0)
box.materials = [boxMaterial]
let contactBox = SCNNode(geometry: box)
contactBox.name = "ring"
contactBox.physicsBody = SCNPhysicsBody(type: .kinematic, shape: nil)
contactBox.physicsBody?.categoryBitMask = Game.Physics.Categories.ring
self.addChildNode(contactBox)
{% endhighlight %}


### HUD

The second new class in part 3 is the HUD (which stands for *Heads Up Display*). The HUD itself is not make with SceneKit.
It's based on SpriteKit, the 2D pendant of SceneKit. At the moment we just display a text to show the number of
rings we have passed, but we will extend that later.

{% highlight swift %}
class HUD {
    private var _scene: SKScene!
    private let _points = SKLabelNode(text: "0 RINGS")

    var scene: SKScene {
      get {
        return _scene
      }
    }

    var points: Int {
        get {
            return 0
        }
        set(value) {
            _points.text = String(format: "%d RINGS", value)
        }
    }

    init(size: CGSize) {
        _scene = SKScene(size: size)

        _points.position = CGPoint(x: size.width/2, y: size.height-50)
        _points.horizontalAlignmentMode = .center
        _points.fontName = "MarkerFelt-Wide"
        _points.fontSize = 30
        _points.fontColor = UIColor.white
        _scene.addChild(_points)
    }
}
{% endhighlight %}

That's all which is needed for this. We just must attach it to the sceneView

{% highlight swift %}
_sceneView.overlaySKScene = hud.scene
{% endhighlight %}

and also connect it to the GameLevel, so we can use it as part of the game logic.

## Physics

The other thing i have introduced in this part of the tutorial is the physics engine.
In this game we just need it for collision detection, which can be difficult in 3D games.
So it's a big advantage to have it. Later we will also use it for real physics behaviour.
SceneKit makes that really simple. In fact we just have to do the following steps.

1. Set the contact delegate to our scene *self.physicsWorld.contactDelegate = self*
2. Implement *func physicsWorld(_ world: SCNPhysicsWorld, didBegin contact: SCNPhysicsContact)*
3. Add a physics body to the nodes
4. Set the *categoryBitMask* (See *GameSettings.swift*)
5. Set the *contactTestBitMask* to get a contact message with the nodes we want

The delegate message is straight forward then

{% highlight swift %}
func physicsWorld(_ world: SCNPhysicsWorld, didBegin contact: SCNPhysicsContact) {
  if let ring = contact.nodeB.parent as? Ring {
    collision(withRing: ring)
  }
}
{% endhighlight %}

Of course it will get later more complicated, but so far we just have one player and some rings.
Like that we don't have to check more. We know because of that, that nodeA will always be the player
and nodeB is a ring. The collision handling itself is then handled in the extra method

{% highlight swift %}
func collision(withRing ring: Ring) {
  if ring.isHidden {
    return
  }

  ring.isHidden = true
  _player!.roll()

  touchedRings += 1
  _hud?.points = touchedRings
}
{% endhighlight %}

![Screenshot 1]({{"/images/zerotohero-III-1.png"}})

This screenshot shows the collision boxes of the player and the ring. Useful for debug purposes.

This is more or less self explanatory. We check if we have touched the ring before, by test the *isHidden()* flag.
if not, we hide the ring, let the plane roll as a nice effect and increase the number of touched rings.
Also we update the HUD.

That's it for now! I hope you liked this tutorial part.
Next time we improve the gameplay even more and make a **real game** out of it.


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
