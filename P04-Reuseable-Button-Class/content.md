---
title: Making a custom Button
slug: making-a-custom-button-node
---
Here we will go through how to subclass an _SKSpriteNode_ and make a button that will come in handy in this project and anyone other Spritekit projects you work on.

### you will learn the following
- Subclassing

- Assigning custom classes

- Handlers

- States


# Getting Started
> [action] press _Command+N_ to make a new File, when the promp comes up click on Swift File, Then it should ask you to name it.
>
> Name it CustomButtonNode or something along those lines.
>

 ![new File](../Tutorial-Images/firstSceneFinal.png)

>[info]
> Once you make the new Swift file you should be greeted with a blank File
> You want to import `SpriteKit` as we will need it to subclass the `SKSpriteNode`
>

# creating the class
>[action] create the custom class that subclasses an `SKSpriteNode`
>
> I _highly_ recommend not copying and pasting but instead writing out the code that we will be going through. Just a reminder that you're only doing yourself a disservice by copying and pasting.

```
import SpriteKit

class CustomButtonNode: SKSpriteNode {
  // required method to use nodes from level editor
  required init(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
  }
}

```

# Adding the states
>[action] Now you need to add the different states that a button might have. Try coming up with some states before looking at the code below, make sure to put them inside an enum outside of the class.
>

```
import SpriteKit

enum ButtonNodeState {
  case Active, Selected, Hidden
}

class CustomButtonNode: SKSpriteNode{
  // required method to use nodes from level editor
  required init(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
  }
}
```
>[info] This enum will be used to keep track of what state our button is in.
>

# Adding button Variables

> [action]
> we need a few variables for the button to work effectively
>
> try coming up with what kind of variables might be needed.

```
enum ButtonNodeState {
  case Active, Selected, Hidden
}

import SpriteKit

class CustomButtonNode: SKSpriteNode{
  // variable to control when we want the button enabled or not
  // initialized to true because we want the button be enabled by default
  var isButtonEnabled: Bool = true

  // This is what will run when we tap on the button
  // We initialize it as a simple print statement and we can later reassign it
  var selectedHandler: () -> Void = {print("No Button action is set")}
  // required method to use nodes from level editor
  required init(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
  }
}
```
For the simplicity of this tutorial we will only be using 3 variables. But wait a second There are only two shown above...
the three variables we will use are

- isButtonEnabled : This will allow us to control when we want the button to be enabled or not.
- selectedHandler : This is what our button will call when we tap on it. It's basically storing a function inside a variable that we can later call when a user taps the button.
- State : This will be our "state manager" meaning this is what will manage all of the button states and make sure the appropriate things happen at different states.

>[action] Try and implement the state variable. _Hint_: It will include a didSet and a switch
>

```
enum ButtonNodeState {
  case Active, Selected, Hidden
}

import SpriteKit

class CustomButtonNode: SKSpriteNode{
  // variable to control when we want the button enabled or not
  var isButtonEnabled: Bool = true

  // This is what will run when we tap on the button
  // We initialize it as a simple print statement and we can later reassign it
  var selectedHandler: () -> Void = {print("No Button action is set")}

  //stateManager that will manage the different states of the button
  var state: ButtonNodeState = .Active {
    didSet {
      switch state {
        case .Active:
        // when the button is active we want to enable user interaction and set the alpha to fully visible
          self.isUserInteractionEnabled =  true
          self.alpha = 1
          break
        case . Selected:
        // when the button is selected we want to make the button slightly transparent
          self.alpha = 0.7
          break
        case .Hidden:
        // when the button is hidden we want disable button interaction and hide and make the alpha fully invisible
          self.isUserInteractionEnabled =  false
          self.alpha = 0
          break
      }
    }
  }
  // required method to use nodes from level editor
  required init(coder aDecoder: NSCoder) {
    super.init(coder: aDecoder)
  }
}
```

Now that we have the three main variables set there is one more thing we need to do before we implement the main functions.

>[action] go to the required init function and add
>
```
self.isUserInteractionEnabled =  true
```
>

The required init function should now look like This

```
required init(coder aDecoder: NSCoder) {
  super.init(coder: aDecoder)
  // makes sure whenever me initialize a button it's intractable be default
  self.isUserInteractionEnabled =  true
}
```


# Main functions
The Button is nearly finished we just need to override two functions that come with the _SKSPriteNode_ class.

The functions we will override are:


```
func touchesBegan(_ touches: set<UITouch>, with event: UIEvent?){}
```
and

```
func touchesEnded(_ touches: set<UITouch>, with event: UIEvent?){}
```

>[action] Try to implement these two functions on your own before looking at the code below.
>

Once the functions have you have been overridden you should have a class that looks like this:

```
import SpriteKit

enum ButtonNodeState {
    case Active, Selected, Hidden
}

class CustomButtonNode: SKSpriteNode {
    var buttonEnabled = true
    /* Setup a dummy action closure */
    var selectedHandler: () -> Void = { print("No button action set") }

    /* Button state management */
    var state: ButtonNodeState = .Active {
        didSet {
                switch state {
                case .Active:
                    /* Enable touch */
                    self.isUserInteractionEnabled = true

                    /* Visible */
                    self.alpha = 1
                    break
                case .Selected:
                    /* Semi transparent */
                    self.alpha = 0.7
                    break
                case .Hidden:
                    /* Disable touch */
                    self.isUserInteractionEnabled = false

                    /* Hide */
                    self.alpha = 0
                    break
            }
        }
    }

    /* Support for NSKeyedArchiver (loading objects from SK Scene Editor */
    required init?(coder aDecoder: NSCoder) {

        /* Call parent initializer e.g. SKSpriteNode */
        super.init(coder: aDecoder)

        /* Enable touch on button node */
        self.isUserInteractionEnabled = true
    }

    // MARK: - Touch handling
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        if isButtonEnabled{
          // change state
            state = .Selected
        }

    }

    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        if isButtonEnabled {
          // run code assigned by other section
            selectedHandler()
            // change state back to active
            state = .Active
        }

    }

}
```

# how to implement
Now you have a new Swift file and within that swift file you have a button class. How in the world you do you implement that into the game?

>[action] head over to the GameScene.sks file and click on the playButton
> In the property inspector click on the _Custom Class Inspector_
> Where is says Custom class you want to type in the class we just made
>`CustomButtonNode`
> and you're done!

## Finish the connection

>[action] Head over to `GameScene.swift` and make a reference to the playButton in the SceneDidLoad method.

but before that change the playButton variable from ``` var playButton: SKSpriteNode! ``` to ``` var playButton: CustomButtonNode! ```

now the code to connect it should look something like this.

```
if let playButton = self.childNode(withName: "playButton") as? CustomButtonNode {
  self.playButton = playButton
} else {
  print("playButton was not initialized properly")
}
```




# Summary

Now you have a great button class that you can use through out your game

- This is a great spot to make another commit! you don't want to loose all the hard work you just did.

In the next chapter you will be making the world Scroll!
