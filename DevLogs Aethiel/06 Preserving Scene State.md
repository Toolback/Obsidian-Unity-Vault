## Preserving Scene State[6.1]

goals :
-> explain concept of state management
->

concept :
-> 
![[Pasted image 20221208184240.png]]
![[Pasted image 20221208184346.png]]
![[Pasted image 20221208184540.png]]
![[Pasted image 20221208185427.png]]
![[Pasted image 20221208185450.png]]

What well do 
![[Pasted image 20221208185704.png]]


## Create Core Scene State Classes[6.2]

goals :
-> Create Prerequist class for our main logic

### Create our own Vector3Serializable.cs Class
``` c#

[System.Serializable]
// Vector 3 of unity isnt serializable so isnt savable in a data file
public class Vector3Serializable
{
public float x, y, z;
  
public Vector3Serializable(float x, float y, float z)
{
this.x = x;
this.y = y;
this.z = z;
}

public Vector3Serializable()
{
// create an instance of the class without any parameters
}
}
```

### Create SceneItems.cs Class
1. Every scene items will have one of these attached to store his position / name / itemCode
``` c#
﻿[System.Serializable]
public class SceneItem
{
    public int itemCode;
    public Vector3Serializable position;
    public string itemName;

    public SceneItem()
    {
        position = new Vector3Serializable();
    }
}
```

### Create SceneSave.cs Class
``` c#
﻿using System.Collections.Generic;

[System.Serializable]
public class SceneSave
{
// string key is an identifier name we choose for this list

public Dictionary<string, List<SceneItem>> listSceneItemDictionary;

public List<SceneItem> listSceneItem;
}
```

### Create the GameObjectSave.cs 
``` c#
﻿using System.Collections.Generic;

[System.Serializable]
public class GameObjectSave
{
    // string key = scene name
    public Dictionary<string, SceneSave> sceneData;

    public GameObjectSave()
    {
        sceneData = new Dictionary<string, SceneSave>();
    }

    public GameObjectSave(Dictionary<string, SceneSave> sceneData)
    {
        this.sceneData = sceneData;
    }
}
```

### Create the GenerateGUID.cs
``` c#
﻿using UnityEngine;

[ExecuteAlways]
public class GenerateGUID : MonoBehaviour
{
    [SerializeField]
    private string _gUID = "";

    public string GUID { get => _gUID; set => _gUID = value; }

    private void Awake()
    {
        // Only populate in the editor
        if (!Application.IsPlaying(gameObject))
        {
            // Ensure the object has a guaranteed unique id
            if (_gUID == "")
            {
                //Assign GUID
                _gUID = System.Guid.NewGuid().ToString();
            }
        }
    }
}
```

## Scene Storing and Restoring[6.3]

goals :
-> Create the main logic for storing item state of different scenes =)
->

concept :
-> 

### Create the interface ISaveable
-> interface that need to be implemented 
``` c#
﻿public interface ISaveable
{
    string ISaveableUniqueID { get; set; }

    GameObjectSave GameObjectSave { get; set; }

    void ISaveableRegister();

    void ISaveableDeregister();

    GameObjectSave ISaveableSave();

    void ISaveableLoad(GameSave gameSave);


    void ISaveableStoreScene(string sceneName);

    void ISaveableRestoreScene(string sceneName);

}
```
2. asd

### Create the SaveLoadManager.cs
->
1. asd
``` c#
﻿using System.Collections.Generic;
using System.IO;
using System.Runtime.Serialization.Formatters.Binary;
using UnityEngine;
using UnityEngine.SceneManagement;

public class SaveLoadManager : SingletonMonobehaviour<SaveLoadManager>
{

    public GameSave gameSave;
    public List<ISaveable> iSaveableObjectList;

    protected override void Awake()
    {
        base.Awake();

        iSaveableObjectList = new List<ISaveable>();
    }

    public void StoreCurrentSceneData()
    {
        // loop through all ISaveable objects and trigger store scene data for each
        foreach (ISaveable iSaveableObject in iSaveableObjectList)
        {
            iSaveableObject.ISaveableStoreScene(SceneManager.GetActiveScene().name);
        }
    }

    public void RestoreCurrentSceneData()
    {
        // loop through all ISaveable objects and trigger restore scene data for each
        foreach (ISaveable iSaveableObject in iSaveableObjectList)
        {
            iSaveableObject.ISaveableRestoreScene(SceneManager.GetActiveScene().name);
        }
    }
}
```
2. asd

### Create the SceneItemsManager.cs
-> Live in the persistent scene and handle storing / loading etc 
1. 
``` c#
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;


[RequireComponent(typeof(GenerateGUID))]
public class SceneItemsManager : SingletonMonobehaviour<SceneItemsManager>, ISaveable
{
    private Transform parentItem;
    [SerializeField] private GameObject itemPrefab = null;

    private string _iSaveableUniqueID;
    public string ISaveableUniqueID { get { return _iSaveableUniqueID; } set { _iSaveableUniqueID = value; } }

    private GameObjectSave _gameObjectSave;
    public GameObjectSave GameObjectSave { get { return _gameObjectSave; } set { _gameObjectSave = value; } }

    private void AfterSceneLoad()
    {
        parentItem = GameObject.FindGameObjectWithTag(Tags.ItemsParentTransform).transform;
    }

    protected override void Awake()
    {
        base.Awake();

        ISaveableUniqueID = GetComponent<GenerateGUID>().GUID;
        GameObjectSave = new GameObjectSave();
    }

    /// <summary>
    /// Destroy items currently in the scene
    /// </summary>
    private void DestroySceneItems()
    {
        // Get all items in the scene
        Item[] itemsInScene = GameObject.FindObjectsOfType<Item>();

        // Loop through all scene items and destroy them
        for (int i = itemsInScene.Length - 1; i > -1; i--)
        {
            Destroy(itemsInScene[i].gameObject);
        }
    }

    public void InstantiateSceneItem(int itemCode, Vector3 itemPosition)
    {
        GameObject itemGameObject = Instantiate(itemPrefab, itemPosition, Quaternion.identity, parentItem);
        Item item = itemGameObject.GetComponent<Item>();
        item.Init(itemCode);
    }

    private void InstantiateSceneItems(List<SceneItem> sceneItemList)
    {
        GameObject itemGameObject;

        foreach (SceneItem sceneItem in sceneItemList)
        {
            itemGameObject = Instantiate(itemPrefab, new Vector3(sceneItem.position.x, sceneItem.position.y, sceneItem.position.z), Quaternion.identity, parentItem);

            Item item = itemGameObject.GetComponent<Item>();
            item.ItemCode = sceneItem.itemCode;
            item.name = sceneItem.itemName;
        }
    }

    private void OnDisable()
    {
        ISaveableDeregister();
        EventHandler.AfterSceneLoadEvent -= AfterSceneLoad;
    }

    private void OnEnable()
    {
        ISaveableRegister();
        EventHandler.AfterSceneLoadEvent += AfterSceneLoad;
    }

    public void ISaveableDeregister()
    {
        SaveLoadManager.Instance.iSaveableObjectList.Remove(this);
    }


    public void ISaveableRestoreScene(string sceneName)
    {
        if (GameObjectSave.sceneData.TryGetValue(sceneName, out SceneSave sceneSave))
        {
            if (sceneSave.listSceneItem != null)
            {
                // scene list items found - destroy existing items in scene
                DestroySceneItems();

                // now instantiate the list of scene items
                InstantiateSceneItems(sceneSave.listSceneItem);
            }
        }
    }

    public void ISaveableRegister()
    {
        SaveLoadManager.Instance.iSaveableObjectList.Add(this);
    }

    public GameObjectSave ISaveableSave()
    {
        // Store current scene data
        ISaveableStoreScene(SceneManager.GetActiveScene().name);

        return GameObjectSave;
    }



    public void ISaveableStoreScene(string sceneName)
    {
        // Remove old scene save for gameObject if exists
        GameObjectSave.sceneData.Remove(sceneName);

        // Get all items in the scene
        List<SceneItem> sceneItemList = new List<SceneItem>();
        Item[] itemsInScene = FindObjectsOfType<Item>();

        // Loop through all scene items
        foreach (Item item in itemsInScene)
        {
            SceneItem sceneItem = new SceneItem();
            sceneItem.itemCode = item.ItemCode;
            sceneItem.position = new Vector3Serializable(item.transform.position.x, item.transform.position.y, item.transform.position.z);
            sceneItem.itemName = item.name;

            // Add scene item to list
            sceneItemList.Add(sceneItem);
        }

        // Create list scene items in scene save and set to scene item list
        SceneSave sceneSave = new SceneSave();
        sceneSave.listSceneItem = sceneItemList;

        // Add scene save to gameobject
        GameObjectSave.sceneData.Add(sceneName, sceneSave);
    }
}
```


### Update the SceneControllerManager
->
1. 
``` c#
// This is the coroutine where the 'building blocks' of the script are put together.

private IEnumerator FadeAndSwitchScenes(string sceneName, Vector3 spawnPosition)

{

// Call before scene unload fade out event

EventHandler.CallBeforeSceneUnloadFadeOutEvent();

  

// Start fading to black and wait for it to finish before continuing.

yield return StartCoroutine(Fade(1f));

  

// Store scene data
// new here
SaveLoadManager.Instance.StoreCurrentSceneData(); 

  

// Set player position

  

Player.Instance.gameObject.transform.position = spawnPosition;

  

// Call before scene unload event.

EventHandler.CallBeforeSceneUnloadEvent();

  

// Unload the current active scene.

yield return SceneManager.UnloadSceneAsync(SceneManager.GetActiveScene().buildIndex);

  

// Start loading the given scene and wait for it to finish.

yield return StartCoroutine(LoadSceneAndSetActive(sceneName));

  

// Call after scene load event

EventHandler.CallAfterSceneLoadEvent();

  

// Restore new scene data
// new here
SaveLoadManager.Instance.RestoreCurrentSceneData();

  

// Start fading back in and wait for it to finish before exiting the function.

yield return StartCoroutine(Fade(0f));

  

// Call after scene load fade in event

EventHandler.CallAfterSceneLoadFadeInEvent();

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

  
// new here !
SaveLoadManager.Instance.RestoreCurrentSceneData();

  
  

// Once the scene is finished loading, start fading in.

StartCoroutine(Fade(0f));

}
```


### Create GO and settup execution script order
-> Create a SceneItemsManager GO and a SaveLoadManager with corresponding scripts attached, and set the execution order of the SaveLoadManager script just before our SceneControllerManager
![[Pasted image 20221208220507.png]]
1. 
``` c#

```


### Cr
->
1. 
``` c#

```


### Cr
->
1. 
``` c#

```


### Cr
->
1. 
``` c#

```


### Cr
->
1. 
``` c#

```
