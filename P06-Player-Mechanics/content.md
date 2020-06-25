---
title: Adding Player Controls
slug: player-movement-class
---

Just like the obstacleSpawner class make a new file and name it `Player.swift`. This file will contain all of the mechanics unique to our main player.

# Creating the class
now that you have that a new file you want to add a class named Player to it. your file should look like this

```
import SpriteKit

class Player: SKSpriteNode {

}
```

Now that that's set up we need a few variables that our player will need.

```

import SpriteKit

class Player: SKSpriteNode {
  var canStack: Bool = true
  let maxStack: Int = 8
  var bodyNodes: [SKSpriteNode] = []
  var initialPos: CGPoint!
}
```

These variables is what the player will use to give the game a specific "Feeling"

- `canStack`: This is to make sure the player meets certain conditions
- `maxStack`: Allows us to limit how many times a player can stack
- `bodyNodes`: Allows us to keep try of how many nodes are stacked
- `initialPos`: will be used to check if a node has been "hit"

Now that we have our varibles set we can add the setup method to the player class like so.
>[action]
> Add the setup method under the variables we just made.


```
func setup(){
  // set the texture to the head from our assets file
    self.texture = SKTexture(imageNamed: "head")
    // give the player a physicsBody
    self.physicsBody = SKPhysicsBody(rectangleOf: CGSize(width: 100, height: 100))
    self.physicsBody?.affectedByGravity = true
    self.physicsBody?.isDynamic = true
    self.physicsBody?.allowsRotation = false
    self.physicsBody?.categoryBitMask = PhysicsCategory.Player
    self.physicsBody?.collisionBitMask = PhysicsCategory.Obstacle | PhysicsCategory.PlayerBody
    self.physicsBody?.contactTestBitMask = PhysicsCategory.Obstacle
    self.initialPos = self.position
}

```
>[info] All this method does is assign a textture and a physics body to our player. As well as assign various physics body properties and the bitMasks

The only other method our player will have is the stack class.

>[action]
>Add the following method under the setup method.

```

func stack(scene: SKScene){
    // check if we are able to stack again
    if canStack && bodyNodes.count <= maxStack{
      // while this is happening we cant stack again
        canStack = false
        // make a new spriteNode with the egg texture
        let body = SKSpriteNode(color: .cyan, size: CGSize(width: 100, height: 100))
        body.texture = SKTexture(imageNamed: "egg")
        // give it a physicsBody
        body.physicsBody = SKPhysicsBody(rectangleOf: CGSize(width: 100, height: 100))
        body.physicsBody?.allowsRotation = false
        body.physicsBody?.isDynamic = true
        body.physicsBody?.affectedByGravity = true
        body.physicsBody?.categoryBitMask = PhysicsCategory.PlayerBody
        body.physicsBody?.contactTestBitMask = PhysicsCategory.Obstacle
        body.physicsBody?.collisionBitMask =  PhysicsCategory.Obstacle | PhysicsCategory.PlayerBody
        // make animation for player
        let jump = SKAction.moveTo(y: self.position.y + self.size.height + 10, duration: 0.1)

        let addChild = SKAction.run {
          // conver the space from the player space to the scene space
            body.position = scene.convert(CGPoint(x: self.position.x , y: self.position.y - (self.size.height + 5)), to: scene)
            // add the body to our bodyNodes array
            self.bodyNodes.append(body)
            // add the body to the scene
            scene.addChild(body)
        }
        // a small wait to give a unique feel
        let smallWait = SKAction.wait(forDuration: 0.03)
        // enableStack
        let enableStack = SKAction.run {
            self.canStack = true
        }
        // animation sequence
        let sequence = SKAction.sequence([jump, addChild, smallWait, enableStack])
        self.run(sequence)
    }

}
```

>[info] This method is what will allow us to stack `eggs` or nodes under our player.
> take some time and read through the comments to see if you can understand what is happening.

# Connecting the Class
To connect the class we are going to do something very similar to what we did with the obstacleSpawner class.

>[action] Go to GameScene.swift and change ``` var player: SKSpriteNode! ``` to ``` var player: Player! ```
>

-

>[info] if you haven't done so make sure that each node that needs a customClass has it in the level editor `GameScene.sks`

- `obstacleSpawner`: custom Class = `obstacleSpawner`
- `player`: custom Class = `Player`
- `playButton`: custom class = `CustomButtonNode`

> [action] Make a reference to the player in SceneDidLoad. The code should look like this.

```
if let player = self.childNode(withName: "player") as? Player {
  self.player = player
} else {
  print("player was not initialized properly")
}
player.setup()
```

Now that we have a reference to our player how can we make our player stack?

>[action]
>Look through `GameScene.swift` and figure out where you could call the stack method.

Look for

```
override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {

}
```

add the following code to it.

```
// ray that detects other nodes above the player
let nodeCheck = physicsWorld.body(alongRayStart: player.position, end: CGPoint(x: player.position.x, y: player.position.y + 100))
if nodeCheck?.node == nil {
    player.stack(scene: scene!)
}
```

>[info] Since we have obstacles that may be above the player, we want to use
>
```
physicsWorld.body(alongRayStart: CGPoint, end: CGPoint)
```
> By using this we essentially send out a `ray` from the players center and goes 50 points above the player to check if we find a node. If we don't then we get nil in return and we can go ahead and stack and egg under the player.
>

# Summary
Our player can now stack and the obstacleSpawner spawns obstacle. Great!

- This is a great spot to make a git push make sure to save all that progress!

In the final section we will be making the final adjustments to make the game feel like a game.
