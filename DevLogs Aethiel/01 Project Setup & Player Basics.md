personal notes from # Rob Ager's tutorial : 
https://www.udemy.com/course/unity-2d-game-developer-course-farming-rpg/
Art from : 
https://seliel-the-shaper.itch.io/
bought bundle 
-> https://itch.io/s/82469/-farming-goods-bundle
-> https://itch.io/s/77934/-home-style-bundle
-> https://itch.io/s/77152/-animated-monster-bundle
-> https://itch.io/s/29693/-iconic-village-bundle
-> https://itch.io/s/68533/-castle-builders-bundle
## Day 1 | 2/12 (~10h)
# Game Architecture && Project Creation
Tuto build on unity v2019.3.7f1, tried to go with the latest version and 2d URP template.
- ***Installed Packages*** :
	-> *Cinemachine* : camera game object
	-> all others comes from original packages
- ***Custom Packages comes with tuto***
	-> Caracters / tiles / buildings / crops / etc
	-> Animations
	-> audio 
	-> custom tilemap brush (CoordinateBrush.cs)
	
## Project Settings [4.5]
--- Project Settings / Edit project Settings
		--- Quality :
		1. Deleted all settings beside [Low quality]
		2. [~X]  Check [Anti Aliasing] (avoid staircaise appearance on object borders) is disabled (Anti aliasing as moved from volume to main camera>rendering)
		--- Graphics :
		3. Set [Transparancy Sort Mode] to (Custom Axis) (in 3d games, things are sorted along the Z axis which gives an appearance of depth, but 2d game dont need Z axis, everything is flat on screen. Depth in 2d is represented by sorting things along the Y axis ) and then set the [Transparancy Sort Axis]  X:0 Y:1 Z:0 to represent 2d depth. [~ X] No Transparancy Sort Mode in URP graphics settings, go to - - >> Assets(folder)/Settings/*Renderer2D* theres the [Transparancy Sort Mode] with corresponding settings, linked by *UniversalRP*
	
- ***Set VS Code as Pref Ide***
	1. --- Edit/Preference/External Sript Editor = Visual Studio Code
	
- ***Scene Settings(*persistent* scene concept)***
  a ==persistent scene will remained loaded all times==. Player location throught differents map will be loaded additivly, and unloaded when player moves out of location
 1. Create a New P. scene by file/save as/scenes/ and call it PersistentScene
 2. Delete the Sample Scene, the PersistentScene is now Loaded in our Hierarchy
 
- ***Create a Virtual Cinemachine Camera***
	1. --- GameObject>Cinemachine>VirtualCamera
	VCamera renamed PlayerFollowVirtualCamera and placed in the Hierarchy
	2. change the orthographic size from 5 to 8.4375
			![[Pasted image 20221202143949.png]]
	3. Change the Body>from [Transposer] to [Framing Transposer]. Keep the GameObject thats its followed within a frame a applied additionnal parameters.
	4. Change [X Damping] from 1 to 0.7 and [Y Damping] from 1 to 0.7 and [Z Damping] from 1 to 0.7. Allow a little bit of movement within that frame without snapping to the object that it following
	5. 5. Change the [Dead Zone Width] and [Dead Zone Height] from 0 to 0.1
	6. Change Aim> from [Composer] to Do Nothing
- ***Setup Main Camera***
	1. change the camera>environment>background from blue to black
	2. add pixel perfect component and set [Assets Pixels Per Unit] to 16 (as our assets here), and the [Reference Resolution] by the size calculated above X:480 Y:270
	3. [?]Crop Frame X and Y + Stretch Fill (on new version of pixel perfect, tried the Stretch Fill mode on Crop Frame) 
	4. [?]Then go to edit>grid  and snap settings> and set Move, X,Y,Z from 0.25 to 0.0625 (in version 2021 of unity, grid and snap settings has now moved cf picture below)
	 ![[Pasted image 20221202154934.png]]

# Player Basics
## Player GameObject Set-Up [5.1]

goals :
-> setup player object and child object hierarchy
->setup the cinemachine camera to tracks players as it moves around the screen

concept :
-> [RigidBody2D] Component allow to interact with unity physics system.
-> [Layers] In 3D Game, the Z Axis is used to determine the draw order of objects, based on a distance away from the camera. in 2D game, we have the concept of *Sorting Layers*. The order of layers is used to determine the order of which sprites will be drawn on screen
-> [Sorting Group] Component Ensure that all subobjects of a parent are sorted in single entity
->  ==**"Pivot"**== allow to center the sprite on the pivot point if the image has been designed for (middle bottom of the image, allow to sort caracter by his feet). Used with a Sorting Group component on the parent object, it allow to sort in depth correctly represented object. 


### Create the player GameObject :
1.  Add a new empty GO to Persistent Scene in Hierarchy named Player 
2. Create a new [Sorting Layer] named "Instance" : Layers>Edit Layers>Sorting Layers>+ to add a new, named it and place the order (Highest is the nearest of the camera)
4. Create child objects underneath the player GO to represent/layer different body parts (hair, arms, tronc, tools, etc) for maximum flexibility, and to be customized with others tools/objects.
	1. Create a child GO underneath the Player GO named "body"and add a [Sprite Renderer] Component to it. Set its [Sorting Layer] to "Instance", and its [Order in Layer] to 1. And Its [Sprite Sort Point] to "Pivot" 
	2. For time saving, duplique the body part (CTRL+D) and renamed it "hair", and change its [Ordering Layer] to 2, to be drawn above the body
	3. Repeat for Hat, Arms, EquippedItem, Tool, and *Tool Effect* at order in layer 7
	4. Give to each body part his appropriate Sprite Render>Sprite (asset) and a Transparent Sprite for empty atm (like hat, equiped item, tool, tool effect)
5. Add a [Rigid Body 2D] Component to the parent GO, Player. 
	set
	-> its [Angular Drag] from 0.05 to 0
	-> its [Gravity Scale] from 1 to 0
	-> its [Collision Detection] from Discrete to Continuous (makes it more responsive in terms of colisions with other GO)
	-> its [Interpolate] from None to Interpolate (increase also the accuracy of colision detection)
	
	Constraints>
	-> Enable Z [Freeze Rotation], which prevent from spinning the player around
	
6. Add a [Sorting Group] Component to the Player Object, and set his [Sorting Layer] to "Instances". 
7. set the Player [Tag] as Player, and set a color mark ('yellow' here)

### Setup the Cinemachine Vcam to Follow the Player:

1. Pass to CinemachineVirtualCamera>Follow the Player GameObject(transform, its position atm in the game)

## Player Class And Abstract Singleton Class [5.2]

goals :
-> create a singleton abstract class
->create the player class (first steps, will be filled later on)

concept :
-> [Singleton Monobehavior Abstract Class] will be used by the *Game Manager*, and the *Player Class*, it ensure that theres is only one instance of the chilld inheriting class created/running, it also make game object accessible pretty much globally by having a static instance. This abstract class create a pattern and avoid dupliquate code. (Abstract class are made to be inherited to be instanciated, they cannot be accessed on their own)
-> Game Object : Every Game Object comes with a [Transform] component, which specify the current location of the GO in the world and give access to this variable to other GO / Scripts. Default (from search bar) Components can be attached to a game object, or scripted ones, which make it kind of a Container of variables which can be accessed

### Create the Singleton Monobehavior Class :
1. Create a new SingletonMonobehavior.cs script, and set its prototype and import as :
``` c#
using UnityEngine;

public abstract class SingletonMonobehavior<T> : MonoBehaviour where T:MonoBehaviour
{

}
```
	- Here, we define the class as abstract to ensure its inheritance, and we define a Generic with <T> (can be any letter, but T is a convention) (respond for ex with the unity function getComponentByType(<type>) => retrieve corresponding component).
	- We also define <T> of type Monobehavior that ensure that any class that we pass in as generic types are of types Monobehavior.
	- Then we define a private static variable of type T named "instance", private for accessible only by our program, static for globally accessible once instanciated by his classname (instance), T for its type. Thats whats make the singleton globaly accessible.

2. Define the variables with getter :
```c
	{
		private static T instance;
		//Getter for the private variable
		public static T Instance
		{
			get
			{
				return instance;
			}
		}
	}
```
- we define a private static variable of type T named "instance", private for accessible only by our program, static for globally accessible once instanciated by his classname (instance), T for its type. Thats whats make the singleton globaly accessible.
- then a public var for getting the private one (safety)



3. Create the awake method (unity built in method whos run once at the initialisation of a GO)
``` c#
protected virtual void Awake()
{
	if (instance == null)
	{
		instance = this as T;
	}
	else
	{
		Destroy(gameObject);
	}
}
```
protected means that it can be accessed by inheriting classes and virtual means that it can be overriden by inheriting classes
- if the instance doesn't already exist, then we're going to start the instance and set it to this GO. Else if one exist, we r gonna destroy this double instance 

### Create the Player Class :
1. Create a Player.cs script and define the proto : 
``` c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class Player : SingletonMonobehavior<Player>
{
}
```
Player inherit of SingletonMonobehavior
2. Ensure only one instance of singleton exist (player)

## Player Animation Controllers [5.3]

goals :
-> Set up animations for the player

concept :
-> [Animator] Component to attach an Animation Controller to a GO
-> [Animation Controller] Interface allowing to manage transitions between states of an animation (idle => run / walk / pick, etc), handle Event at specific timeframe (for exemple foot step sound). Animator Layer allow to compartiment different animation in a parent GO in 3D games

1. Add an [Animator] Component to every body part (exept for "equippedItem", which is a static sprite for holding items) and set its appropriate Animation Controller in [Controller]
2. Fix eventual event (methods) error linked to animation (empty mvmt footstep method attached to GO to fix issue)

## Events and Event Handler [5.4]

goals :
-> Create an centralized static Event Handler Class for different uses. This class will contain static event, delegate variables that subscribers can reference anywhere in the code, to add methods to which should be run when the event is triggered. Will Also contain corresponding static functions that will be used to trigger these events by publishers.
-> Create a movement events that can be subscribed to, and a method to call that movement event.
-> used to test player animation in next chapter

concept :
-> Use of a Design Patter, named [Publisher and Subscriber] cf image below
![[Pasted image 20221202231644.png]]
the idea is that throughtout our game, well need to triggers event. For exemple when player press a key(publisher), it can be received as a movement event(green) and trigger a methods, animation, etc (subscriber).

In C#, Event Handling is managed by functionnality named delegates and event : 
![[Pasted image 20221202232210.png]]
theres also a predefined delegate type called Action in the system namespace that we will use.
A Delegate is a reference data type that holds references to methods and is specified using the delegate keyword.
The delegate signature should math the methods that will be called, for ex, the number of parameters should match

If a subscriber wants a function or method to be triggered as a result of an event, that method is added to that event's delegate variable, then every time that event is triggered, all the methods are being added to the delegate variable by possibly multiple subsribers, are run.

in c# the event keywork gives extra functionality to delegate. It enable multiple subscribers to easily and safely subscribe to events using the += notation, without overriding methods other subscribers have added to that delegate variable.

if subscribers no longer want to be notified of an event, the -= is then used to unsubscribe from that event.
Event are triggered by calling the event delegates, passing in any required parameters.

### Create Event Handler Class & a Movement Event :
1.  Create an Enums.cs which contain all the tool effect (will be used as a type)
``` c#
public enum ToolEffect
{
	none,
	watering,
}
```
an enum defines a number of specific values.
Here we define the normal state, and the watering state (effect)
2. Create and EventHandler.cs script :
``` c#
public delegate void MovementDelegate(float inputX, float inputY, boo isWalking, bool isRunning, bool isIdle, bool isCarrying,
ToolEffect toolEffect,
bool isUsingToolRight, bool isUsingToolLeft, bool isUsingToolUp, bool isUsingToolDown,
bool isLiftingToolRight, bool isLiftingToolLeft, bool isLiftingToolUp, bool isLiftingToolDown,
bool isPickingRight, bool isPickingLeft, bool isPickingUp, bool isPickingDown,
bool isSwingingToolRight, bool isSwingingToolLeft, bool isSwingingToolUp, bool isSwingingToolDown,
bool idleRight, bool idleLeft, bool idleUp, bool idleDown);

public static class EventHandler
{
}
```
-> First we define our public static class EventHandler (which is not Monobehaviour)
-> Then we create our own delegate for the movement event, (the reason why we create our own rather than using the system action delegates, is our MovementEvent has more than 16 parameters (max input sys call))
specified by the public delegate MovementDelegate. Parameters will match methods added to the variable
3. Then we're going to create the movement event : 
``` c#
public static class EventHandler
{
	// Movement Event
	public static event MovementDelegate MovementEvent;
	// Movement Event Call for Publishers
	// (function below)
}
```
-> for that we define the var [MovementEvent] of type MovementDelegate, event keyword gonna wrap our delegate type keyword to allow subscribers to subscribe or unsubscribe to that event
``` c# 
public static void CallMovementEvent(float inputX, float inputY, boo isWalking, bool isRunning, bool isIdle, bool isCarrying,
ToolEffect toolEffect,
bool isUsingToolRight, bool isUsingToolLeft, bool isUsingToolUp, bool isUsingToolDown,
bool isLiftingToolRight, bool isLiftingToolLeft, bool isLiftingToolUp, bool isLiftingToolDown,
bool isPickingRight, bool isPickingLeft, bool isPickingUp, bool isPickingDown,
bool isSwingingToolRight, bool isSwingingToolLeft, bool isSwingingToolUp, bool isSwingingToolDown,
bool idleRight, bool idleLeft, bool idleUp, bool idleDown)
{

if (MovementEvent != null)

MovementEvent(inputX, inputY, isWalking, isRunning, isIdle, isCarrying,
toolEffect,
isUsingToolRight, isUsingToolLeft, isUsingToolUp, isUsingToolDown,
isLiftingToolRight, isLiftingToolLeft, isLiftingToolUp, isLiftingToolDown,
isPickingRight, isPickingLeft, isPickingUp, isPickingDown,
isSwingingToolRight, isSwingingToolLeft, isSwingingToolUp, isSwingingToolDown,
idleRight, idleLeft, idleUp, idleDown)

}
```
-> Then we define in the same Class the method for publisher which call this particular event, same input as the delegate define above. 
First Thing is to check if theres any subscriber to this event, and only execute the event if theres some.
![[Pasted image 20221203133240.png]]
### Day2 | 3/12 (9h)
## Player Animation Test Harness [5.5]

goals :
-> Create an animation test harness which will be the publisher (trigger event on interface click), the subscriber will be our animation 

concept :
-> In unity, we can either call and set animation parameters from code using the string parameter name, or by creating a hash representing the string parameter name (hash faster until unity has to create a string object for every call)
-> Creation of a static Settings file (globally accessible whitout instanciation) which can be referenced throughout the project 

### Settings.cs And  MovementAnimationParameterControl.cs:
-> Define hash corresponding to event parameter
-> Set up the subscriber mecanic for each body part to animate on event trigger

1. Create the Settings.cs file, and define a set of integer value (whats use to set the hash value for animation parameter)
& Define a static constructor (named Settings() ) which gets run when class is created, and set the variable defined with an integer hash corresponding to the string parameter
``` c#
using UnityEngine;

public static class Settings
{
	public static int xInput;
	public static int yInput;
	public static int isWalking;
	public static int isRunning;
	public static int toolEffect;
	public static int isUsingToolRight;
	public static int isUsingToolLeft;
	public static int isUsingToolUp;
	public static int isUsingToolDown;
	public static int isLiftingToolRight;
	public static int isLiftingToolLeft;
	public static int isLiftingToolUp;
	public static int isLiftingToolDown;
	public static int isSwingingToolRight;
	public static int isSwingingToolLeft;
	public static int isSwingingToolUp;
	public static int isSwingingToolDown;
	public static int isPickingRight;
	public static int isPickingLeft;
	public static int isPickingUp;
	public static int isPickingDown;

static Settings()

{
// Player Animation Parameters
xInput = Animator.StringToHash("xInput");
yInput = Animator.StringToHash("yInput");
isWalking = Animator.StringToHash("isWalking");
isRunning = Animator.StringToHash("isRunning");
toolEffect = Animator.StringToHash("toolEffect");
isUsingToolRight = Animator.StringToHash("isUsingToolRight");
isUsingToolLeft = Animator.StringToHash("isUsingToolLeft");
isUsingToolUp = Animator.StringToHash("isUsingToolUp");
isUsingToolDown = Animator.StringToHash("isUsingToolDown");
isLiftingToolRight = Animator.StringToHash("isLiftingToolRight");
isLiftingToolLeft = Animator.StringToHash("isLiftingToolLeft");
isLiftingToolUp = Animator.StringToHash("isLiftingToolUp");
isLiftingToolDown = Animator.StringToHash("isLiftingToolDown");
isSwingingToolRight = Animator.StringToHash("isSwingingToolRight");
isSwingingToolLeft = Animator.StringToHash("isSwingingToolLeft");
isSwingingToolUp = Animator.StringToHash("isSwingingToolUp");
isSwingingToolDown = Animator.StringToHash("isSwingingToolDown");
isPickingRight = Animator.StringToHash("isPickingRight");
isPickingLeft = Animator.StringToHash("isPickingLeft");
isPickingUp = Animator.StringToHash("isPickingUp");
isPickingDown = Animator.StringToHash("isPickingDown");

// Shared Animation parameters

idleUp = Animator.StringToHash("idleUp");

idleDown = Animator.StringToHash("idleDown");

idleLeft = Animator.StringToHash("idleLeft");

idleRight = Animator.StringToHash("idleRight");

}
}
```
2. then in the MovementAnimationParameterControler.cs script (the one attached to each body part of the player), we want to make it subscribe to the movement event. For that, we first start by defining the Animator component attached to our GO bodypart, here named animator, and set its value in our Awake methods with GetComponent()
``` c#
public class MovementAnimationParameterControl : MonoBehaviour
{
	// compoment defined in an Animator type variable named animator
	private Animator animator;
	// animator is then set with proper Animator compoment at the instanciation of the GO
	private void Awake()
	{
	animator = GetComponent<Animator>();
	}
	
	private void AnimationEventPlayFootstepSound()
	{
	}

}
```
3. We'll use the OnEnable / Disable Unity Methods to subscribe / unsubscribe to our static class EventHandler > MovementEvent, and define the methods that we want to call when tiggered, here SetAnimationParameters()
``` c#
[...]Awake()
private void OnEnable()
{
	EventHandler.MovementEvent += SetAnimationParameters;
}

private void OnDisable()
{
	EventHandler.MovementEvent -= SetAnimationParameters;
}
[...]AnimationEventPlayFootstepSound()
```

4. Then we define the SetAnimationParameters(), which propagate received data from Publisher to animator variable with *corresponding hash* from Settings.cs:
``` c#
[...]OnDisable()
private void SetAnimationParameters(float xInput, float yInput, bool isWalking, bool isRunning, bool isIdle, bool isCarrying, ToolEffect toolEffect,

bool isUsingToolRight, bool isUsingToolLeft, bool isUsingToolUp, bool isUsingToolDown,

bool isLiftingToolRight, bool isLiftingToolLeft, bool isLiftingToolUp, bool isLiftingToolDown,

bool isPickingRight, bool isPickingLeft, bool isPickingUp, bool isPickingDown,

bool isSwingingToolRight, bool isSwingingToolLeft, bool isSwingingToolUp, bool isSwingingToolDown,

bool idleUp, bool idleDown, bool idleLeft, bool idleRight)

{
animator.SetFloat(Settings.xInput, xInput);
animator.SetFloat(Settings.yInput, yInput);
animator.SetBool(Settings.isWalking, isWalking);
animator.SetBool(Settings.isRunning, isRunning);

animator.SetInteger(Settings.toolEffect, (int)toolEffect);
  
if (isUsingToolRight)
animator.SetTrigger(Settings.isUsingToolRight);

if (isUsingToolLeft)
animator.SetTrigger(Settings.isUsingToolLeft);

if (isUsingToolUp)
animator.SetTrigger(Settings.isUsingToolUp);

if (isUsingToolDown)
animator.SetTrigger(Settings.isUsingToolDown);


if (isLiftingToolRight)
animator.SetTrigger(Settings.isLiftingToolRight);

if (isLiftingToolLeft)
animator.SetTrigger(Settings.isLiftingToolLeft);

if (isLiftingToolUp)
animator.SetTrigger(Settings.isLiftingToolUp);

if (isLiftingToolDown)
animator.SetTrigger(Settings.isLiftingToolDown);

  

if (isSwingingToolRight)
animator.SetTrigger(Settings.isSwingingToolRight);

if (isSwingingToolLeft)
animator.SetTrigger(Settings.isSwingingToolLeft);

if (isSwingingToolUp)
animator.SetTrigger(Settings.isSwingingToolUp);

if (isSwingingToolDown)
animator.SetTrigger(Settings.isSwingingToolDown);

  
if (isPickingRight)
animator.SetTrigger(Settings.isPickingRight);

if (isPickingLeft)
animator.SetTrigger(Settings.isPickingLeft);

if (isPickingUp)
animator.SetTrigger(Settings.isPickingUp);

if (isPickingDown)
animator.SetTrigger(Settings.isPickingDown);

  

if (idleUp)
animator.SetTrigger(Settings.idleUp);

if (idleDown)
animator.SetTrigger(Settings.idleDown);

if (idleLeft)
animator.SetTrigger(Settings.idleLeft);

if (idleRight)
animator.SetTrigger(Settings.idleRight);
}

[...]AnimationEventPlayFootstepSound()
```

### Creating the PlayerAnimationTest.cs :
Now that Events and Subscribers are sets, time to code the Test Publisher, PlayerAnimationTest.cs.

-> here we set a list corresponding to actionable parameters, which can be access on Unity UI, and then call in an Update() methods (run every frame) the EventHandler.CallMovementEvent with last value 
``` c#
using UnityEngine;
  
public class PlayerAnimationTest : MonoBehaviour
{
	public float inputX;
	public float inputY;
	public bool isWalking;
	public bool isRunning;
	public bool isIdle;
	public bool isCarrying;
	public ToolEffect toolEffect;
	public bool isUsingToolRight;
	public bool isUsingToolLeft;
	public bool isUsingToolUp;
	public bool isUsingToolDown;
	public bool isLiftingToolRight;
	public bool isLiftingToolLeft;
	public bool isLiftingToolUp;
	public bool isLiftingToolDown;
	public bool isPickingRight;
	public bool isPickingLeft;
	public bool isPickingUp;
	public bool isPickingDown;
	public bool isSwingingToolRight;
	public bool isSwingingToolLeft;
	public bool isSwingingToolUp;
	public bool isSwingingToolDown;
	public bool idleUp;
	public bool idleDown;
	public bool idleLeft;
	public bool idleRight;
	
	private void Update()
	{
		EventHandler.CallMovementEvent(inputX, inputY, isWalking, isRunning, isIdle, isCarrying, toolEffect,
		isUsingToolRight, isUsingToolLeft, isUsingToolUp, isUsingToolDown,
		isLiftingToolRight, isLiftingToolLeft, isLiftingToolUp, isLiftingToolDown,
		isPickingRight, isPickingLeft, isPickingUp, isPickingDown,
		isSwingingToolRight, isSwingingToolLeft, isSwingingToolUp, isSwingingToolDown,
		idleUp, idleDown, idleLeft, idleRight);
		}
}
```

-> now we can attach the PlayerAnimationTest.cs to the Player GO and set different value in the Inspector (game must be running) (X/Y [-1/0/1])

## Basic Player Movement [5.6]

goals :
-> capture player movement input, then move player sprite with proper animations based on the inputs

concept :
-> 

### Pimp Settings.cs & Enum.cs:
-> add some hardcoded movement speed value 
1. Settings
``` c# 
public static class Settings
{
// Player Movement
	public const float runningSpeed = 5.333f;
	public const float walkingSpeed = 2.666f;
[...]
```
2. Enum.cs / add direction state 
``` c#
public enum Direction
{
	up,
	down,
	left,
	right,
	none
}
```

### Add input and movement code to class Player.cs :
1. set same variable as movement event 
-> then define the rigid body of the player rigidBody2D
-> define a var playerDirection with the enum "Direction" created, used later for *save game functionality*, hold / save current player direction
-> cach others variables
``` c#
using UnityEngine;

public class Player : SingletonMonobehaviour<Player>
{
// Movement Parameters
	private float xInput;
	private float yInput;
	private bool isCarrying = false;
	private bool isIdle;
	private bool isLiftingToolDown;
	private bool isLiftingToolLeft;
	private bool isLiftingToolRight;
	private bool isLiftingToolUp;
	private bool isRunning;
	private bool isUsingToolDown;
	private bool isUsingToolLeft;
	private bool isUsingToolRight;
	private bool isUsingToolUp;
	private bool isSwingingToolDown;
	private bool isSwingingToolLeft;
	private bool isSwingingToolRight;
	private bool isSwingingToolUp;
	private bool isWalking;
	private bool isPickingUp;
	private bool isPickingDown;
	private bool isPickingLeft;
	private bool isPickingRight;
	
	private ToolEffect toolEffect = ToolEffect.none;

private Rigidbody2D rigidBody2D;

private Direction playerDirection;

private float movementSpeed;

private bool _playerInputIsDisabled = false;

public bool PlayerInputIsDisabled { get => _playerInputIsDisabled; set => _playerInputIsDisabled = value; }
}
```

2. set/cache values in awake (base.awake for the SingletonMonobehaviour):
``` c#
[...]Player.cs

protected override void Awake()
{
	base.Awake();
	rigidBody2D = GetComponent<Rigidbody2D>();
}
```

3. Then use the unity Update() and FixedUpdate() methods to :
- update ()
-> Capture & Cache Player Movement Input 
-> Send Cached Mvmt data to GO bodyparts subscribed to the MovementEvent to synch accordingly the animation 
- fixedUpdate()
-> Move player Rigidbody2D from Cached Mvmt Data
``` c#
[...]Player.css

private void Update()
{
#region Player Input
  
if (!PlayerInputIsDisabled)
{
// reset all animations
ResetAnimationTriggers();

// capture player mvmt input (set direction, speed, etc)
PlayerMovementInput();

// capture if player press shift for running of not (walking)
PlayerWalkInput();

// Send event to any listeners from captured/cached movements input (body part sprites animators for ex)
EventHandler.CallMovementEvent(xInput, yInput, isWalking, isRunning, isIdle, isCarrying, toolEffect,
isUsingToolRight, isUsingToolLeft, isUsingToolUp, isUsingToolDown,
isLiftingToolRight, isLiftingToolLeft, isLiftingToolUp, isLiftingToolDown,
isPickingRight, isPickingLeft, isPickingUp, isPickingDown,
isSwingingToolRight, isSwingingToolLeft, isSwingingToolUp, isSwingingToolDown,
false, false, false, false);
}
#endregion Player Input

}
  
private void FixedUpdate()
// move the player (not animation, position of player on the screen with Rigidbody2D)
// FixedUpdated for physics mvmt, run in // with update() to send animation events
{
PlayerMovement();
}
  
private void PlayerMovement()
{
Vector2 move = new Vector2(xInput * movementSpeed * Time.deltaTime, yInput * movementSpeed * Time.deltaTime);
  
rigidBody2D.MovePosition(rigidBody2D.position + move);
}
  
private void ResetAnimationTriggers()
{
isPickingRight = false;
isPickingLeft = false;
isPickingUp = false;
isPickingDown = false;
isUsingToolRight = false;
isUsingToolLeft = false;
isUsingToolUp = false;
isUsingToolDown = false;
isLiftingToolRight = false;
isLiftingToolLeft = false;
isLiftingToolUp = false;
isLiftingToolDown = false;
isSwingingToolRight = false;
isSwingingToolLeft = false;
isSwingingToolUp = false;
isSwingingToolDown = false;
toolEffect = ToolEffect.none;
}

private void PlayerMovementInput()
{
// return [-1/0/1] from player input
yInput = Input.GetAxisRaw("Vertical");
xInput = Input.GetAxisRaw("Horizontal");
  
if (yInput != 0 && xInput != 0)
{
xInput = xInput * 0.71f;
yInput = yInput * 0.71f;
}

if (xInput != 0 || yInput != 0)
{
isRunning = true;
isWalking = false;
isIdle = false;
movementSpeed = Settings.runningSpeed;
  
// Capture player direction for save game
if (xInput < 0)
{
playerDirection = Direction.left;
}

else if (xInput > 0)
{
playerDirection = Direction.right;
}

else if (yInput < 0)
{
playerDirection = Direction.down;
}

else
{
playerDirection = Direction.up;
}
}

else if (xInput == 0 && yInput == 0)
{
isRunning = false;
isWalking = false;
isIdle = true;
}
}
  
private void PlayerWalkInput()
// test is shift is push for run or walk
{
if (Input.GetKey(KeyCode.LeftShift) || Input.GetKey(KeyCode.RightShift))
{
isRunning = false;
isWalking = true;
isIdle = false;
movementSpeed = Settings.walkingSpeed;
}

else
{
isRunning = true;
isWalking = false;
isIdle = false;
movementSpeed = Settings.runningSpeed;
}
}
```

