
## Game Location Scene [6.1]

goals :
-> Create the farm scene for the game
-> Create a hierarchy of tile map to enable the visual details to be created with a sens of depth 

concept :
-> ![[Pasted image 20221203185422.png]]

### Create New Scene 
-> New Farm Additive Scene.
1. Create a new File>New Scene and save it to Scenes Folder "Scene1_Farm". 
2. Right clic and "Open Scene Additive". This scene we'll be loaded additivly (in addition of the persistent scene already loaded at the beginning)
-> Now right clic on it in the hierarchy, et "set active scene"
1. 

### Create Tile Map
-> 
1. right click on "Scene1_Farm" in the hierarchy, GO>2D>tilemap>rectangular
-> Create 2 object :
- Grid object
- Tilemap, always set within a Grid
2. Rename Grid > Tile Map Grid, and first tile map > Ground 1
3. set tilemap hierarchy (layer of drawing) : 
- Ground1
- Ground2
- Ground3
- GroundDecoration1
- GroundDecoration2
- Instances
- Front1
- Front2
- Collisions
4. Now the invisible one (utilitary, like diggable place), create new empty GO under TileMap Grid, named GridProperties, et create child tilemap :
- BoolCanPlaceFurniture
- CanDropItem
- BoolDiggable
- BoolPath
- BoolNPCObstacle
-> Set their [Sorting Layer] to "Collisions" (invisible)

5. create news sorting layers (same as tilemaps) and associate to their corresponding tilemap
Layer 1-> Ground1
Layer 6-> Instances 
Layer 9 -> Collisions
6. Set the TileMap Grid from hierarchy as a Prefabs, but unpack to avoid slowing down process an keep clean for next usage (other maps)

## Create Farmyard Scene [6.2]

goals :
-> Create Main FarmYard Scene
-> Draw with painting tools on tile maps previously created

concept :
-> To Prepare 2D assets 
- [Sprite Mode] -> Multiple for multiple sprites imported
- [Pixels Per Units] 16 if 16x16, 32 for ...
- [Filter Mode] set to point/no filter (?)
-> then in sprit editor, slice sprites by grid by cells size 

### Create the ground tile set 
-> 
1. Create new palette (Tile Palette Window)named GroundTileSet
2. Drag and drop the GroundTile Sprite Sheat to the palette editor (save to tile Tilemap>Tiles>GroundTileSet, loads of images)
3. check on new tile created (select all images) the collider type (in inspector), and set it from Sprite to None
4. at the bottom of the Tile Palette, change the brush from Default Brush to Coordinate BRush (custom script which give the coordinate below the selected tile)
5. Define Ground1 border (-40,30) -> (40, -30) {i to select and copy a pattern drawn}
6. and lets creativity play ! 
"[]" to rotate brush / shift + "[]"
-> Ground 1
![[Pasted image 20221203201639.png]]
-> Ground 2
-> Ground 3
![[Pasted image 20221203205842.png]]
-> GroundDecoration1
-> Ground Decoration2

## Add Farmhouse And Collision Tiles [6.3]

goals :
-> Add the Farmhouse to the previous scene
-> add colision tiles

### Draw the Farm House 
-> Cut the bottom of the house in ground2 and the top in front1  
![[Pasted image 20221203213959.png]]

### Draw Colisions
-> Create New Palette for colisions tiles
-> /!\ \Make sur that for this tile the [Collider Type] is set to "Sprite", reverse from others tiles

1. Add a [Box Collider 2D] on the Player GO to react to the collider drawn, and size it to the *feet* of the Player 
![[Pasted image 20221203215559.png]]
![[Pasted image 20221203215531.png]]
2. Draw the colider tile on the Collision Layer
![[Pasted image 20221203215839.png]]
3. Add a [TileMap Collider 2D] Component to the Collisions Tile Map
 ![[Pasted image 20221203215956.png]]
4. Merge all separate collider to an unique one with the [Composite Collider 2D] (also add a rigidbody2d coponent)
-> Set the [Geometry Type] to Polygons
-> In the TileMap Collider 2D, enable [Used By Composite]
-> Set the [Rigid Body] - Body Type from Dynamic to Static 
![[Pasted image 20221203220441.png]]
(tilemap renderer has been disable here, no more visual)
5. Draw all colliders on the map 
![[Pasted image 20221203221238.png]]

## Add Scenary [6.4]

goals :
->  Add Mores Things to the scene

### Bushes
-> 
1. Create new empty GO named "Scenary" with a GO child named "Bushes"
2. Drag and drop a bush sprite on Bushes hierarchy to create a GO underneath, and set its [Sprite Sort Point] to pivot & its [Sorting Layer] to Instance
3. Add a [Box Collider 2D] to the bush, and size its collider to the bottom
![[Pasted image 20221203223252.png]]
4. Save that Bushe to Prefabs>scenary
### Trees
->
1. Create a new GO named "Trees" below the Bushes Group, and repeat the same
![[Pasted image 20221203231100.png]]

## Add Cinemachine Confiner [6.5]

goals :
-> Prevent camera of exiting the bond of the terrain with the Virtual Camera

### Create Boundaries
1. Create a GO on the Scene1_Farm with a [polygon collider ]on it which specify the boundaries of the map
-> set its [Layer] to "Ignore RayCast". 
-> enable [Is Trigger] to not interfer with collision
-> Attribute the tag "BoundsConfiner" to the GO to reference it throught code more easily (add it to prefab)
![[Pasted image 20221203232022.png]]
### Set the Camera to bondaries
-> We cannot link 2 GO in different scene, but we can make a script to counter that limitation
-> Creation of a Tags script, which reference the name of the tags used in the unity interface for manual scripts

1. Create a static class Tags.cs in Misc Folder
``` c#
public static class Tags 
{
    public const string BoundsConfiner = "BoundsConfiner";
}
```

2. Create the SwitchConfineBoundingShape.cs (to adapt polygon collider to each scene) :
``` c#
using UnityEngine;
using Cinemachine;

public class SwitchConfineBoundingShape : MonoBehaviour
{
// Start is called before the first frame update
void Start()
{
SwitchBoundingShape();
}
  
// <summary>
// Switch the collider that cinemachine uses to define the edge of the screen
// </summary>

private void SwitchBoundingShape()
{
// Get the polygon collider on the "boundsconfiner" (tag) gameobject which is used by Cinemachine to prevent the camera going beyond the screen edges
PolygonCollider2D polygonCollider2D = GameObject.FindGameObjectWithTag(Tags.BoundsConfiner).GetComponent<PolygonCollider2D>();

CinemachineConfiner cinemachineConfiner = GetComponent<cinemachineConfiner>();


cinemachineConfiner.m_BoundingShape2D = polygonCollider2D;


// since the confiner bounds have changed, need to call this to clear the cache;
cinemachineConfiner.InvalidatePathCache();
}
}
```
3. On the PlayerFollowVirtualCamera, add the extension "Cinemachine Confiner"
4. attach the SwitchConfineBoundingShape.cs Component (no need to define a specific collider)

#### Day 3 | 4/12 (9h)
## Scenary Fader [6.6]

goals :
-> Create a component that fade object when player pass behind (bushed fading)

concept :
-> Script "ObscuringItemFader" -> Fade out / Fade in Routine alpha value on collide
-> [Coroutine] split execution between a number a frame , process rythm by "yield return null;", are or type of Ienumerator

### Add Value to Settings.cs 
1. hardcode value for fade in / out routine
``` c#
[...] Settings.cs
// Obscuring Item Fading - ObscuringItemFader
public const float fadeInSeconds = 0.25f;
public const float fadeOutSeconds = 0.35f;
public const float targetAlpha = 0.45f;
```

### Create ObscuringItemFader.cs
->
1. Creation ouf our fade in / out coroutines 
``` c#
using System.Collections;
using UnityEngine;

[RequireComponent(typeof(SpriteRenderer))]

public class ObscuringItemFader : MonoBehaviour
{
private SpriteRenderer spriteRenderer;

private void Awake()
{
spriteRenderer = gameObject.GetComponent<SpriteRenderer>();
}

public void FadeOut()
{
StartCoroutine(FadeOutRoutine());
}

public void FadeIn()
{
StartCoroutine(FadeInRoutine());
}
  
private IEnumerator FadeInRoutine()
{
float currentAlpha = spriteRenderer.color.a;
float distance = 1f - currentAlpha;

while (1f - currentAlpha > 0.01f)
{
currentAlpha = currentAlpha + distance / Settings.fadeInSeconds * Time.deltaTime;
spriteRenderer.color = new Color(1f, 1f, 1f, currentAlpha);
yield return null;
}
spriteRenderer.color = new Color(1f, 1f, 1f, 1f);
}
  
private IEnumerator FadeOutRoutine()
{
// Define the current Alpha (transparence) of the object
float currentAlpha = spriteRenderer.color.a;
// distance between current opacity and alpha set in Settings.cs
float distance = currentAlpha - Settings.targetAlpha;

  
// decrease alpha in a loop by frame
while(currentAlpha - Settings.targetAlpha > 0.01f)
{
currentAlpha = currentAlpha - distance / Settings.fadeOutSeconds * Time.deltaTime;
spriteRenderer.color = new Color(1f, 1f, 1f, currentAlpha);
yield return null;
}

// ensure correct alpha after progressive decrease
spriteRenderer.color = new Color(1f, 1f, 1f, Settings.targetAlpha);
}
}
```
2. Now we need to attach an TriggerObscuringItemFader.cs component to our player to execute our ObscuringItemFader.cs component attached to object to fade. Player needs an [Collider] Component to be linked with 
``` c# 
using UnityEngine;

public class TriggerObscuringItemFader : MonoBehaviour
{
private void OnTriggerEnter2D(Collider2D collision)
{
// Get the gameobject we have collided with, and then get all the Obscuring Item Fader components on it and its children - and then trigger the fade out

ObscuringItemFader[] obscuringItemFader = collision.gameObject.GetComponentsInChildren<ObscuringItemFader>();

if (obscuringItemFader.Length > 0)
{
for(int i=0; i<obscuringItemFader.Length; i++)
{
obscuringItemFader[i].FadeOut();
}
}
}

private void OnTriggerExit2D(Collider2D collision)
{
// Get the gameobject we have collided with, and then get all the Obscuring Item Fader components on it and its children - and then trigger the fade in

ObscuringItemFader[] obscuringItemFader = collision.gameObject.GetComponentsInChildren<ObscuringItemFader>();

if (obscuringItemFader.Length > 0)
{
for (int i = 0; i < obscuringItemFader.Length; i++)
{

obscuringItemFader[i].FadeIn();
}
}

}
}
```
3. Now Create a [Box Collider 2D] with [isTrigger] enabled on bushes and items to fade (one for collision without istrigger, an other one with trigger for detection), and add the ObscuringItemFader.cs component
![[Pasted image 20221204112345.png]]
4. And add the TriggerObscuringItemFader.cs on the Player GO !
![[Pasted image 20221204112935.png]]
![[Pasted image 20221204112911.png]]