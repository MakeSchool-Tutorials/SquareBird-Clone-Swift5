---
title: Final Touches
slug: the-final-steps
---
Now that we have the basic aspects of the game built. It is time to optimize and make it feel like a game with the ability to lose and make the play button work

# Adding Function to the play button
We have a button but It doesn't do any thing yet. lets change that.
since we already have a reference to our button let go ahead and make it do something.
But before that let's add a way to tell when we're playing vs when we're in the menu.

>[action] At the top of the `GameScene.swift` add this enum

```
enum GameState: Equatable {
    case Active
    case Menu
}
```

>[action]
> Go ahead and make two new variable called `obstacleTimer` and `gameState` and make it of type `Timer`
it should look like so.

```
var obstacleTimer: Timer!
var gameState: GameState = .Menu
```

Lets look at `sceneDidLoad` and add some code to the bottom

```
playButton.selectedHandler = {
  self.gameState = .Active
  }
```

# Generating Obstacles
>[action] now lets add `startGenerator` method to implemented simply add the code below right below `sceneDidLoad`
>

```
@objc func startGenerator(){
    self.obstacleSpawner.generate(scene: self.scene!)
}
```

>[info]
> Since the Timer class requires an `@objc` function we'll simply rap our generate inside an `@objc` method.


Cool so now we have a method that will generate our obstacle and when we click the play button it changes the gameState to active. Nothing will happen if you test the game now so lets change that.

>[action] add a didSet to the gameState variable and add a switch statement for gameState. It should look something like this.

```
var gameState: GameState = .Menu {
    didSet{
        switch gameState {
        case .Active:
          break
        case .Menu:
          break
        }
    }
}
```
>[info] adding a didSet to any variable makes it so that you cna run code whenever the variable changes.
>

Now that we have that set up Theres still nothing happening! so how do we change that?

>[action] Add this code to the `Active` case.

```
// for the nodes in bodyNodes we well remove all of them
for node in player.bodyNodes {
  node.removeFromParent()
}
// remove all animations from the player
player.removeAllActions()
//reset the bodyNodes array
self.player.bodyNodes = []
// reset the rotation to make sure its facing the correct way.
self.player.zRotation = 0
// hide the play button
self.playButton.isHidden = true
reset the player location
self.player.position.x = self.player.initialPos.x
// start a timer by assigning a new one.
self.obstacleTimer = Timer.scheduledTimer(timeInterval: self.fixedDelta, target: self, selector: #selector(self.startGenerator), userInfo: nil, repeats: true)  
```

>[info]
>
- 1: remove all nodes that where added to the scene by the player
>
- 2: remove all animations from the player
>
- 3: set the bodyNodes array to an empty array
>
- 4: set the zRotation to 0 to make sure the player is facing the correct way
>
- 5: hide the play button
>
- 6: create a timer that will fire 60 time a second and will call the generate method.

Now when the play button is press all that will happen.

But what about when the game ends?

>[action] Add this code to the `Menu` case


```
// show the play button
self.playButton.isHidden = false
// stop the timer
self.obstacleTimer.invalidate()

// for all nodes that are an obstacle remove them from the scene
for node in self.children {
  if node.physicsBody?.categoryBitMask == PhysicsCategory.Obstacle {
    node.removeFromParent()
}
```

>[info]
>
- 1: show the play button
>
- 2: stop the timer so no more obstacle are generated
>
- 3: remove all the current obstacles

# Game loop
Now when the games ends it'll basically reset everything.
But wait.. How does the game end? Lets work on that


>[action] add this method to `GameScene.swift`

```
func checkPlayer() {
  // if the player gets bumped back past the error margin of 10 we play an animation and change the game state
    if player.position.x < player.initialPos.x - 10{
      // animations
        let rotate = SKAction.rotate(byAngle: 15, duration: 2.5)
        let pushBack = SKAction.moveTo(x: player.position.x - 400, duration: 2)
        let seq = SKAction.group([pushBack, rotate])
        player.run(seq)
        gameState = .Menu
    } else {
      // else if the player gets bumped an insignificant amount we just reset the play back to the original spot.
        self.player.position.x = self.player.initialPos.x
    }
}
```

>[info] This method essentially checks the position of the player and compares it to the initial position. we add 10 to give it a little room error and if does move with in that error margin we set it back to the initial position. We then define some animations and play them if the player have moved too much, as well as set the gameState back to the menu.


This is fine but you will notice that the eggs don't really do anything when they get hit. Let's change that.

>[action] add this method under `checkPlayer`

```
func checkBody(){
  // if we actually have nodes to check if they have been moved back more then the error margin of 5 we play an animation and remove them from the scene
  if player.bodyNodes.count >= 1 {
    for sprite in player.bodyNodes {
      if sprite.position.x < player.position.x - 5 {
        // filter out the current node that is past the margin of error
        player.bodyNodes = player.bodyNodes.filter {return $0 != sprite }
        // animations
        let rotate = SKAction.rotate(byAngle: 20, duration: 2)
        let pushBack = SKAction.moveTo(x: sprite.position.x - 400, duration: 2)
        // remove the node after the animation is done
        let remove = SKAction.run {
          sprite.removeFromParent()
        }
        let group = SKAction.group([pushBack, rotate])
        let seq = SKAction.sequence([group, remove])
        sprite.run(seq)
            }
        }
    }
}
```

>[info] This method checks the body nodes that the player stacks. if the x position is less than the players. it gets filtered out of the bodyNodes array and then plays animation and gets removed from the scene.


To implement simply call the methods in the update method like so.


```
if gameState == .Active {
    checkBody()
    checkPlayer()
}
```

now test your game!


# Optimization

Notice that the node counts keeps going up? That's a big no no. It'll eventually become too large and crash your game. Let's fix that.


>[action] change the class so that is also subclasses `SKPhysicsContactDelegate`

the top of your class should now look like this.

```
class GameScene: SKScene, SKPhysicsContactDelegate {
```

>[action] at the bottom of `sceneDidload` add this
```
physicsWorld.contactDelegate = self
```

This is letting the scene know it'll be handling contact between nodes.


>[action] Add this method to the class

```
func didBegin(_ contact: SKPhysicsContact) {
    let bodyA = contact.bodyA
    let bodyB = contact.bodyB
// check the categoryBitMasks to make sure we are removing the correct node.
    if bodyA.categoryBitMask == PhysicsCategory.Barrier {
        if bodyB.categoryBitMask == PhysicsCategory.Obstacle {
            bodyB.node?.removeFromParent()
        }

    }
if bodyB.categoryBitMask == PhysicsCategory.Barrier {
    if bodyA.categoryBitMask == PhysicsCategory.Obstacle {
            bodyA.node?.removeFromParent()
        }

    }

}
```

>[info] This method gets called whenever two nodes touch. It's part of the SKSPhysics engine so it does all of that for you. The only thing we will be doing is checking that we remove the correct node when it touches the barrier. In this case we will remove the obstacle nodes.

You are done!

Test your game and make sure everything is working fine.


#Summary
