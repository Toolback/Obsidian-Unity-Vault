## Create a Scene Controller Manager[5.1]

goals :
-> Create a scene controller manager to load and unload scenes as we move between different scenes
->

concept :
-> 
![[Pasted image 20221207223134.png]]
![[Pasted image 20221207223302.png]]
### Add New Scenes in the Enums.cs
->
1. asd
``` c#
[...] EventHandler.cs
public enum SceneName
{
Scene1_Farm,
Scene2_Cascad,
Scene3_Village
}
```
2. asd

### Add more scene change related events in EventHandler.cs
->
1. asd
``` c#
    // Scene Load Events - in the order they happen

    // Before Scene Unload Fade Out Event
    public static event Action BeforeSceneUnloadFadeOutEvent;

    public static void CallBeforeSceneUnloadFadeOutEvent()
    {
        if (BeforeSceneUnloadFadeOutEvent != null)
        {
            BeforeSceneUnloadFadeOutEvent();
        }
    }

    // Before Scene Unload Event
    public static event Action BeforeSceneUnloadEvent;

    public static void CallBeforeSceneUnloadEvent()
    {
        if (BeforeSceneUnloadEvent != null)
        {
            BeforeSceneUnloadEvent();
        }
    }

    // After Scene Loaded Event
    public static event Action AfterSceneLoadEvent;

    public static void CallAfterSceneLoadEvent()
    {
        if (AfterSceneLoadEvent != null)
        {
            AfterSceneLoadEvent();
        }
    }

    // After Scene Load Fade In Event
    public static event Action AfterSceneLoadFadeInEvent;

    public static void CallAfterSceneLoadFadeInEvent()
    {
        if (AfterSceneLoadFadeInEvent != null)
        {
            AfterSceneLoadFadeInEvent();
        }
    }
```
2. asd

### Create the GO SceneControllerManager

1. PersistentScene> New Empty GO named "SceneControllerManager"

2. Create another GO "FadeImage" as a child of our UICanvasGroup (used to draw a black image between scene). The order in the hierarchy matter, so the fact that this GO is the last draw it before everything else 
-> set Anchor Point : Alt +shift + Stretch to all screen
-> Add an [Image] component with a black color
-> set the alpha of the image to 0 to make it transparant for now 
-> Keep enable Raycast Target to get it to block any user click when the fade in appear

-> add a [Canvas Group] component (control alpha / raycast)
-> disable [ignore parent groups] to act on his own


### Create the script for the SceneControllerManager GO
-> SceneControllerManager.cs
1. asd
``` c#
using System;
using System.Collections;
using UnityEngine;
using UnityEngine.SceneManagement; // allow to control loading and unloading of scene
using UnityEngine.UI;

public class SceneControllerManager : SingletonMonobehaviour<SceneControllerManager>
{
    private bool isFading;
    [SerializeField] private float fadeDuration = 1f;
    [SerializeField] private CanvasGroup faderCanvasGroup = null;
    [SerializeField] private Image faderImage = null;
    public SceneName startingSceneName;


    private IEnumerator Fade(float finalAlpha)
    {
        // Set the fading flag to true so the FadeAndSwitchScenes coroutine won't be called again.
        isFading = true;

        // Make sure the CanvasGroup blocks raycasts into the scene so no more input can be accepted.
        faderCanvasGroup.blocksRaycasts = true;

        // Calculate how fast the CanvasGroup should fade based on it's current alpha, it's final alpha and how long it has to change between the two.
        float fadeSpeed = Mathf.Abs(faderCanvasGroup.alpha - finalAlpha) / fadeDuration;

        // While the CanvasGroup hasn't reached the final alpha yet...
        while (!Mathf.Approximately(faderCanvasGroup.alpha, finalAlpha))
        {
            // ... move the alpha towards it's target alpha.
            faderCanvasGroup.alpha = Mathf.MoveTowards(faderCanvasGroup.alpha, finalAlpha,
                fadeSpeed * Time.deltaTime);

            // Wait for a frame then continue.
            yield return null;
        }

        // Set the flag to false since the fade has finished.
        isFading = false;

        // Stop the CanvasGroup from blocking raycasts so input is no longer ignored.
        faderCanvasGroup.blocksRaycasts = false;
    }

    // This is the coroutine where the 'building blocks' of the script are put together.
    private IEnumerator FadeAndSwitchScenes(string sceneName, Vector3 spawnPosition)
    {
        // Call before scene unload fade out event
        EventHandler.CallBeforeSceneUnloadFadeOutEvent();

        // Start fading to black and wait for it to finish before continuing.
        yield return StartCoroutine(Fade(1f));

        // Set player position

        Player.Instance.gameObject.transform.position = spawnPosition;

        //  Call before scene unload event.
        EventHandler.CallBeforeSceneUnloadEvent();

        // Unload the current active scene.
        yield return SceneManager.UnloadSceneAsync(SceneManager.GetActiveScene().buildIndex);

        // Start loading the given scene and wait for it to finish.
        yield return StartCoroutine(LoadSceneAndSetActive(sceneName));

        // Call after scene load event
        EventHandler.CallAfterSceneLoadEvent();

        // Start fading back in and wait for it to finish before exiting the function.
        yield return StartCoroutine(Fade(0f));

        // Call after scene load fade in event
        EventHandler.CallAfterSceneLoadFadeInEvent();
    }


    private IEnumerator LoadSceneAndSetActive(string sceneName)
    {
        // Allow the given scene to load over several frames and add it to the already loaded scenes (just the Persistent scene at this point).
        yield return SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Additive);

        // Find the scene that was most recently loaded (the one at the last index of the loaded scenes).
        Scene newlyLoadedScene = SceneManager.GetSceneAt(SceneManager.sceneCount - 1);

        // Set the newly loaded scene as the active scene (this marks it as the one to be unloaded next).
        SceneManager.SetActiveScene(newlyLoadedScene);
    }

    private IEnumerator Start()
    {
        // Set the initial alpha to start off with a black screen.
        faderImage.color = new Color(0f, 0f, 0f, 1f);
        faderCanvasGroup.alpha = 1f;

        // Start the first scene loading and wait for it to finish.
        yield return StartCoroutine(LoadSceneAndSetActive(startingSceneName.ToString()));

        // If this event has any subscribers, call it.
        EventHandler.CallAfterSceneLoadEvent();


        // Once the scene is finished loading, start fading in.
        StartCoroutine(Fade(0f));
    }


    // This is the main external point of contact and influence from the rest of the project.
    // This will be called when the player wants to switch scenes.
    public void FadeAndLoadScene(string sceneName, Vector3 spawnPosition)
    {
        // If a fade isn't happening then start fading and switching scenes.
        if (!isFading)
        {
            StartCoroutine(FadeAndSwitchScenes(sceneName, spawnPosition));
        }
    }

}
```

### Set up the GO
-> Attach the SceneControllerManager to its GO and link appropriate references

### Update the UIInventorySlot.cs
-> when player change scene, 
1. suppress the following line, then add the rest below
``` c# 
//suppress this line in the Start Methods
parentItem = GameObject.FindGameObjectWithTag(Tags.ItemsParentTransform).transform; 
```
then add 
``` c#
private void OnDisable()
{
	EventHandler.AfterSceneLoadEvent -= SceneLoaded;
}

private void OnEnable()
{
	EventHandler.AfterSceneLoadEvent += SceneLoaded;
}

public void SceneLoaded()
{
	parentItem = GameObject.FindGameObjectWithTag(Tags.ItemsParentTransform).transform;
}

```
2. asd

### Update the SwitchConfineBoundingShape.cs
-> Again, when cant assume the GO is present until loaded scene, so change the Start Method
1. asd
``` c# [...] 
SwitchConfineBoundingShape.cs

// suppress the start method and replace with => 

private void OnDisable()
{
	EventHandler.AfterSceneLoadEvent -= SwitchBoundingShape;
}

private void OnEnable()
{
	EventHandler.AfterSceneLoadEvent += SwitchBoundingShape;
}
```
2. asd

### Add code to Player.cs
-> check if scene loaded 
1. asd
``` c# [...] 
player.cs

// suppress the start method and replace with => 
private void PlayerTestInput()
{
[...]
	// Test Scene unload / load
	if (Input.GetKeyDown(KeyCode.L))
	
	{
		SceneControllerManager.Instance.FadeAndLoadScene(SceneName.Scene1_Farm.ToString(), transform.position);
	}
}
```

### Modify the Script Execution Order 
-> In project settings make sur that the scene as loaded before item try to access it
1. edit > projet settings > script execution order
-> + button, find SceneControllerManager and add it to the bottom with -90

2. Unload the scene from the hierarchy since the SceneControllerManager we'll do it !
## Create a New Field Scene[5.2]

Create a second scene additively added to the hierarchy, and then unload it !

#### Day 7 | 08/12 (6h) 
## Move Between Scenes[5.3]

goals :
-> Move between the Farm scene and the new one
->

concept :
-> farm -22;-31, -16;-31
-> Cascad -22;30 -17;30

### Create a SceneTeleport GO
-> under Scene1 Farm, create an empty GO named "SceneTeleport", and create a custom script "SceneTeleport"
1. asd
``` c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(BoxCollider2D))]
public class SceneTeleport : MonoBehaviour
{
    [SerializeField] private SceneName sceneNameGoto = SceneName.Scene1_Farm;
    [SerializeField] private Vector3 scenePositionGoto = new Vector3();


    private void OnTriggerStay2D(Collider2D collision)
    {
        Player player = collision.GetComponent<Player>();

        if (player != null)
        {
            //  Calculate players new position

            float xPosition = Mathf.Approximately(scenePositionGoto.x, 0f) ? player.transform.position.x : scenePositionGoto.x;

            float yPosition = Mathf.Approximately(scenePositionGoto.y, 0f) ? player.transform.position.y : scenePositionGoto.y;

            float zPosition = 0f;

            // Teleport to new scene
            SceneControllerManager.Instance.FadeAndLoadScene(sceneNameGoto.ToString(), new Vector3(xPosition, yPosition, zPosition));

        }

    }
}
```
2. asd

### Set up the GO
-> Assign the position and scene you want to go in the GO script, and voila !

## Create a New Farmhouse Cabin Scene[5.4]



