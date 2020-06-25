---
title: Generating Obstacles
slug: adding-obstacles-to-the-world
---

Just like when we created the custom button we will be making a new swift file and subclassing the _SKSpriteNode_ class
> [action]
> press _Command+N_ to create a new swift file and name it `ObstacleSpawner`
>

# Making the class
once you have your new swift file set you should have a class that looks like this

```
import SpriteKit

class ObstacleSpawner: SKSpriteNode {

}
```

## Variables we will be using.

this class will require a few variables that are similar to the ones we used for the world scroll.

### add these variables to the ObstacleSpawner class:

```
var spawnTimer: CFTimeInterval = 0
let fixedDelta: CFTimeInterval = 1.0 / 60.0 /* 60 FPS */
var timeSpace: CFTimeInterval?
let scrollSpeed = 0.3
var columnPointer = 0
var currentScheme: [[Int]] = []
```

- `spawnTimer` will be used to determine when we want to spawn the obstacles
- `fixedDelta` is a constant we will be adding to other variables and will be added 60 times per second
- `timeSpace` is a timer that allows us to know the minimum time to wait before spawning another obstacle and giving them enough space.
- `columnPointer` since we are going to be working with a 2D array this variable will be used to keep track of what column we are spawning.
- `currentScheme` is the current obstacle our generator will be spawning.

# The setup
before we actually get started with the meat and potatoes of this class, we need to add a few things to our project.

>[action]First create a new file and put this code in it.

```
struct Schemes {

    var schemes: [[[Int]]] = [
        [[0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 1, 0, 0, 0],
         [0, 0, 0, 0, 1, 1, 0, 0, 0],
         [0, 0, 0, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0]],

        [[0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0]],

        [[0, 0, 0, 1, 1, 0, 0, 0, 0],
         [0, 0, 0, 1, 1, 0, 0, 0, 0],
         [0, 0, 0, 1, 1, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0]],

        [[0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0]],

        [[0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0]],

        [[0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0]],

        [[0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 0, 0, 1, 0, 0, 0],
         [0, 0, 1, 0, 0, 1, 0, 0, 0],
         [0, 0, 1, 1, 1 ,1, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0]],

        [[0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 1, 0, 0, 0, 0],
         [0, 0, 0, 0, 1, 1, 0, 0, 0],
         [0, 0, 0, 0, 1, 1, 1, 0, 0],
         [0, 0, 0, 0, 1, 1, 1, 1, 0]],

        [[0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 1, 1, 1, 0, 0, 0],
         [0, 0, 0, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 0, 0, 0]],

        [[0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0],
         [0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]]
    ]
   static var randomElement: [[Int]]!
 // returns a random element from our schems array
    func getRandomScheme() -> [[Int]]{
        return self.schemes.randomElement()!
    }

}
```
>[info] This struct will house all of the level designs you can come up with. I made 10 simple ones to get you started. You can either design entire levels or create obstacles and use them to generate a random level like we'll be doing today. The X axis doesn't matter, but the Y axis should be the same as how ever many blocks you have in the level editor. in this case that number is 9. The designs or `Schemes` are 2D arrays where "0" represents an empty space and "1" represents a block.

_

> [action] Go back to the ObstacleSpawner file and add the scheme struct as a variable below the ones we just added.


```
let schemes: Schemes = Schemes()
```

 Next we need to make a method that will initialize the `timeSpace` variable. Since different devices have different screen sizes and the space between nodes could change. This way we will always have the correct distance between two obstacle blocks.

>[action] Right below the variables you just declared write this helper methods.

```
func initTimeSpace(){
  // check
    if timeSpace == nil {
        timeSpace = 0
        // get a child from self since we should have 9
        let child = self.children[0] as! SKSpriteNode
        // record the initial position to test speed
        let initialPos = self.children[0].position.x
        // move the child the at the same speed they will move towards player
        let move = SKAction.move(by: CGVector(dx: 100, dy: 0), duration: scrollSpeed)
        child.run(move)
        // user a timer that fires at 60 FPS to mesure the time it takes to get to a certain distance
        let _ = Timer.scheduledTimer(timeInterval: fixedDelta, target: self, selector: #selector(timerCheck(_:)), userInfo: ["child":child, "pos":initialPos], repeats: true)
        currentScheme = schemes.getRandomScheme()
    }
}

@objc func timerCheck(_ timer: Timer){
  // get the info we sent through the timer in this case a dictionary because we needed more than one data.
    let info = timer.userInfo as! [String: Any]
    // get thhe child from dict
    let child = info["child"] as! SKSpriteNode
    // get initial pos from dict
    let initialPos = info["pos"] as! CGFloat
    // check the position of child until it's more than 90% past itself
    if child.position.x <= initialPos + (child.size.width * 0.90) {
      // add time everytime the child is still not where we need it
        timeSpace! += fixedDelta
    } else {
      // we are done the child is where we need it so we stop the timer
        timer.invalidate()
        // remove all of the  animations
        child.removeAllActions()
        // reset the position
        child.position.x = initialPos

    }
}
```

>[info]
> The body of this method gets ran once at the start of the game, when the timeSpace variable is nil. first we set it time space to 0 to initialize it. Then we get the first child node from from the obstacle node and record it's initial position. we  then make an SKAction that will move our node at the speed we declared at the top. We then start a time that will fire at 60 times per second, because of the fixedDelta. This timer will do a couple of things it will check the position of the node, add time to the timeSpace. invalidate the timer once it gets to where we need it and reset the node back to it's initial position. Take a look at the @objc function to see how to pass information to the timer's #Selector


Before we start on the main method of this class we need two more things
> [action]
> add this to the bottom of the file outside the class.

```
extension UIColor {
    static var random: UIColor {
        return UIColor(red: .random(in: 0...1),
                       green: .random(in: 0...1),
                       blue: .random(in: 0...1),
                       alpha: 1.0)
    }
}
```

>[info] This extension to UIColor will allow you to get random colors. We will be using this to make the obstacles random colors.

# Meat and Potatoes

Now that we have that finalized we can get started on the generation logic.
The first thing we want to do is make the function with whatever parameters we're going to be using. In this case we'll only need to pass in an SKScene.
>[action]
> Create a new function called generate and have it take in an SKScene.

your function should look something like this.

```
func generate(scene: SKScene) {

}
```

the first thing we want to do is call the initTimeSpace method


> [action]
> Make a call to the initSpaceTime
>

now your function should look like this

```
func generate(scene: SKScene) {
  // initialize
  initTimeSpace()

}
```

We now want to start adding to our spawnTimer so we can know when you spawn our obstacles


```
func generate(scene: SKScene) {
  initTimeSpace()
  // start adding to our spawn timer.
  spawnTimer += fixedDelta
}
```

Now we can check if we can spawn our obstacle using a simple if statement

```
func generate(scene: SKScene) {
  initTimeSpace()
  // start adding to our spawn timer.
  spawnTimer += fixedDelta
  // check if itis time to spawn another obstacle column
  if spawnTimer > Double(timeSpace!) {

  }
}
```

if we can spawn another obstacle we can go ahead and make a copy of our the entire stack of nodes like so.

```
func generate(scene: SKScene) {
  initTimeSpace()
  // start adding to our spawn timer.
  spawnTimer += fixedDelta
  // check if itis time to spawn another obstacle column
  if spawnTimer > Double(timeSpace!) {
    let copy = self.copy() as! SKSpriteNode
  }
}

```

Now create a variable called column that will store the obstacles we are going to display. your code should be looking like this.

```
func generate(scene: SKScene) {
  initTimeSpace()
  // start adding to our spawn timer.
  spawnTimer += fixedDelta
  // check if it is time to spawn another obstacle column
  if spawnTimer > Double(timeSpace!) {
    // make a copy of the entire column
    let copy = self.copy() as! SKSpriteNode
    // make an empty column that will have all of the modified obstacles
    var column: [SKSpriteNode] = []
  }
}

```

To start adding obstacles to our column variable we need to do a for loop through our currentScheme and check what gets added and not.

```
func generate(scene: SKScene) {
  initTimeSpace()
  spawnTimer += fixedDelta
  if spawnTimer > Double(timeSpace!) {
    let copy = self.copy() as! SKSpriteNode
    var column: [SKSpriteNode] = []
    for row in 0...currentScheme.count - 1 {
    }
  }
}

```

This is where our column pointer comes into play. since we are working on a 2D array we need two variables to track row and column. we need a simple check to know when to get another scheme and when to reset our columnPointer varible. this can be done like so.

```
func generate(scene: SKScene) {
  initTimeSpace()
  spawnTimer += fixedDelta
  if spawnTimer > Double(timeSpace!) {
    let copy = self.copy() as! SKSpriteNode
    var column: [SKSpriteNode] = []
    // for every row in the current scheme we want to check if we've reached the end of the columns
    for row in 0...currentScheme.count - 1 {
      if columnPointer > currentScheme[row].count - 1{
        // reset the columnPointer
        columnPointer = 0
        // get the nextScheme
        currentScheme = schemes.getRandomScheme()
      }

    }
  }
}

```


Now that we can keep track of the column and row we need to start making copies of individual nodes. This is where the naming comes into play. we will make a copy of a specific node depending on what row we are checking and we will check it against the current scheme to see if it is added to the scene or not.

```
func generate(scene: SKScene) {
  initTimeSpace()
  spawnTimer += fixedDelta
  if spawnTimer > Double(timeSpace-!) {
    let copy = self.copy() as! SKSpriteNode
    var column: [SKSpriteNode] = []

    for row in 0...currentScheme.count - 1 {
      if columnPointer > currentScheme[row].count - 1{
        columnPointer = 0
        currentScheme = schemes.getRandomScheme()
      }
      // copy the node that represents the row we are currently working on
      let child = copy.childNode(withName: "spawnBlock-\(row)") as! SKSpriteNode
      // if spot in the scheme is == 1 then we modify it else we remove it.
      if currentScheme[row][pointer] == 1 {
        // make it  random color to add variety
        child.color = UIColor.random
        child.colorBlendFactor = 0.7
        // convert the position from it's parent space to the scene space.
        child.position = scene.convert(child.position, to: scene)
        let scenePos = scene.convertPoint(fromView: copy.position)
        // move the x to the location of its parent
        child.position.x = scenePos.x + child.size.width
        // edit the physicsBody properties
        child.physicsBody?.categoryBitMask = PhysicsCategory.Obstacle
        child.physicsBody?.collisionBitMask = PhysicsCategory.Player | PhysicsCategory.Obstacle
        // add a copy to the column array
        column.append(child.copy() as! SKSpriteNode)
      } else {
        child.removeFromParent()
      }
    }
    // once we are done remove all the children that are left over if any and remove the copy itself
    copy.removeAllChildren()
    copy.removeFromParent()

    // make animations to move the child forever
    let move = SKAction.moveBy(x: -100, y: 0, duration: scrollSpeed)
    let repeater = SKAction.repeatForever(move)
    // for all the children in the column we add it to the scene and run the animation
    for copiedChild in column {
      scene.addChild(copiedChild)
      copiedChild.run(repeater)
      }
      // we then reset the column back to an emty array to get it ready for the next column
    column = []
    //reset the spawnTimer and columnPointer
    spawnTimer = 0
    columnPointer += 1
  }
}

```
>[info] That seems like a lot at once so lets walk through it.
> first we make a copy of the child node named `spawnblock-\(row)` row meaning what row we are currently processing. after that we set the the physicsBody's categoryBitMask as Invisible because we want it to default to an invisible block  unless told otherwise, we also make the collisionBitMask as None so it doesn't interact with anything. We then proceed to check if that block is equal to a `1` in the 2D matrix and if it does we edit the physics body and other properties accordingly. we then copy the child over to an array that has all of the nodes we have edited, No longer needing the copy of the stack we remove all it's children and the copy itself. we make an infinite moving action and assign it to all the nodes that don't have the invisible PhysicsCategory. we then reset the column to get it ready for the next column in the matrix and reset the spawn timer and the columnPointer.



# Implementation
Now that we have the main function of the spawner complete there are a few more things we need to do to get it working.

> [action]
> go to `GameScene.sks`, once there click on the ObstacleSpawner and head over to the custom class inspector. Where is it says custom class type in `ObstacleSpawner`
>

This connects the class you just made to the stack of nodes in the scene. Neat!

>[action]
> Head over to `GameScene.swift` Once you're there change ```  var obstacleSpawner: SKnode !``` to ``` var obstacleSpawner: ObstacleSpawner! ```
>
> connect the node Like so in the `SceneDidLoad`


```
if let spawner = self.childNode(withName: "obstacleSpawner") as? ObstacleSpawner {
  self.obstacleSpawner = spawner
} else {
  print("spawner could not be connected properly")
}
```


>[info] to test call ```obstacleSpawner.generate(scene: self.scene!)``` inside the update method and run the game!
> I would not leave it in there as the update method gets called once every frame and frames are not constant thus being a bad practice
> we will be implementing a way to generate the obstacles in a different way in the last part of the tutorial.
>

-

>[info] The blocks might look a little off that will be fixed in the last part.

# Summary

we now have the object spawner working! This is the good but we can't control the player yet :(

- This is a great place to make a commit so you don't loose any of your progress.
- In the next chapter it's time to finally add the player controls and make the game challenging!
