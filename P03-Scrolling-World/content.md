---
title: Scrolling the world
slug: start-world-scroll
---

Time for you to bring this world to life, you will be creating a conveyor belt
system to scroll the world objects toward the player, giving illusion of the
player moving.

![Conveyor Belt](https://media.giphy.com/media/WFkbyRl2Ke1oY/giphy.gif)

# Ready to roll

To get started there are a couple of variables we are going to need

> [action]
> Add the following varaibles to `GameScene.swift`.

```
let fixedDelta: CFTimeInterval = 1.0 / 60.0 /* 60 FPS */
let scrollSpeed: CGFloat = 200
var scrollNode: SKNode!
var player: SKSpriteNode!
var obstacleSpawner: SKNode!
var playButton: SKSpriteNode!
var frontBarrier: SKSpriteNode!
```
we will be using these variables through out the game.

## Virtual scroll layer

You will need to use the scroll speed and the fixedDelta properties to manipulate the scroll speed of the
conveyor belt.

> [action]
> In `GameScene.swift`, add the following code to connect the scrollNode
> to the code. add the code in the `SceneDidLoad` function.
>
```
/* Set reference to scroll Node */
if let scrollNode = self.childNode(withName: "scrollNode") {
  self.scrollNode = scrollNode
} else {
  print("ScrollNode could not be connected properly")
}
```

# Working with physics Categories


We need to define the physics bodies that our game will have. This will make it easier later in the code to tell that nodes what to collide with
> [action]
> Add this Struct the top of the file outside of the class.

```
struct PhysicsCategory {
    static let None: UInt32 = 0
    static let Player: UInt32 = 0b1
    static let Obstacle: UInt32 = 0b10
    static let PlayerBody: UInt32 = 0b100
    static let Barrier: UInt32 = 0b1000
}
```


>[info] These are just 32 bit integers that Apples physics engine uses, because you can only have so many different entities on the screen at once and UInt32 has significantly less numbers than your regular Int which defaults to a 64 bit Integer. So apple uses the 32 bit Integer to limit the amount of unique physics bodies you can have.

Now connect the frontBarrier like so

```
// referecing the barrier node from the scene
if let frontBarrier = self.childNode(withName: "frontBarrier") as? SKSpriteNode {
  self.frontBarrier = frontBarrier
} else {
  print("frontBarrier could not be connected properly")
}

// setting the barrier physics body preferences
frontBarrier.physicsBody?.categoryBitMask = PhysicsCategory.Barrier
frontBarrier.physicsBody?.collisionBitMask = PhysicsCategory.None
frontBarrier.physicsBody?.contactTestBitMask = PhysicsCategory.Obstacle
```

>[info] Here we're just giving it info
>
- categoryBitMask: Telling the node what it is.
>
- collisionBitMask: Telling the node what it'll collide with.
>
- contactTestBitMask: Telling the node what it'll actually acknowledge that it hit.

# Scroll World

To help organize your code, let's create a new method called **scrollWorld** and
call this in the `update(...)` method.

> [action]
> In `GameScene.swift`, add the following method at the end of the
> _GameScene_ class (but before the last closing bracket):
>
```
func scrollWorld() {
  /* Scroll World */
  scrollNode.position.x -= scrollSpeed * CGFloat(fixedDelta)
}
```
>
> Then add the following to the bottom of your `update(...)` method:
>
```
/* Process world scrolling */
scrollWorld()
```

<!-- -->

> [info]
> Defining a member variable for the scroll speed rather than simply
> defining a players position to be increased by `100` \* _delta_ every time is
> an important programming practice. Variable names offer us clarity - if
> someone else looks at your code, or even if you revisit it next week, it may
> not be clear what `100` affects. Explicitly using the variable `scrollSpeed`
> alleviates this problem. They also offer us flexibility. Imagine we were
> writing a larger program which used `scrollSpeed` in several places and
> instead of using a variable, we used `100` every time. What happens if we
> decide our scroll speed is a little slow? We will need to visit every place we
> wrote `100` and change it. It's not hard to understand how this could quickly
> get messy and inefficient.

Run the game!

# Adding objects to scroll

Run the game. The ground should be scrolling, keep watching...


## Loop the ground

Eventually you will run out of ground...

You can make the ground loop by implementing
an endless scrolling technique using both ground sprites we added at the start. When a ground sprite
leaves the left edge you'll move it back to the right edge of the screen to make
the ground seem endlessly repeating.

In the update method, you will perform a check against every ground object in
the _scrollLayer_ to see if it has moved outside of the left edge of the screen,
if so you will then relocate it to back to the right edge.

> [action] In `GameScene.swift`, add the following code to the end of the
> `scrollWorld()` method:
>
```
/* Loop through scroll layer nodes */
for ground in scrollNode.children as! [SKSpriteNode] {
>
  /* Get ground node position, convert node position to scene space */
  let groundPosition = scrollNode.convert(ground.position, to: self)
>
  /* Check if ground sprite has left the scene */
  if groundPosition.x <= -ground.size.width / 2 {
>
      /* Reposition ground sprite to the second starting position */
      let newPosition = CGPoint(x: (self.size.width / 2) + ground.size.width, y: groundPosition.y)
>
      /* Convert new node position back to scroll layer space */
      ground.position = self.convert(newPosition, to: scrollNode)
  }
}
```

## Relative node position

This code retrieves the current screen position for each ground sprite. Since
the ground sprites aren't children of the _GameScene_, you need to convert their
relative position inside the _scrollLayer_ to _GameScene_ co-ordinate space
using the `convertpoint(...)` method.

Once you have this world space position, you check if the ground sprite is
outside of the the screen. If so then move it to the right edge of the screen.
You calculate the new position for the node in _GameScene_ (world space) and
then convert this back to get the relative position in _scrollLayer_ space.

This creates the ground's endless repeating effect.

Run the game. The ground should now scroll for eternity. you can test this by waiting for eternity.

# Summary

The game now has a sense of movement. You've learned:

- A helper function that takes parameters.
- Move objects using their position.x property.
- Create an endless scrolling mechanic.

In the next chapter it's time to add the challenge of obstacles.
