## Using Tilemaps to Store Booleans[6.18]

goals :
->![[Pasted image 20221208221257.png]]
![[Pasted image 20221208221522.png]]

![[Pasted image 20221208221625.png]]
![[Pasted image 20221208221759.png]]
![[Pasted image 20221208221807.png]]
![[Pasted image 20221208221943.png]]
![[Pasted image 20221208222113.png]]


## Populate Tilemap Grid Properties[6.18]

concept :
->         tilemap.CompressBounds(); used to reduce extended tilemap (when reducing the sprites, bound are not auto reduce, but auto expand yes )

### First add some categories to Enums.cs
-> here we define different boolean categorie for our tile map grid properties
``` c#
[...] enums.cs
public enum GridBoolProperty
{
	diggable,
	canDropItem,
	canPlaceFurniture,
	isPath,
	isNPCObstacle
}
```
2. asd

### Create the GridCoordinate.cs 
->
1. Serialiazible vector field that allow us to capture the grid coordinate for the tilemap we are painting
-> the Explicit operator is used to explicit conversion of type (float to vector2)
``` c#
﻿using UnityEngine;
//can be save to data file for later functionality 
[System.Serializable]

public class GridCoordinate
{
    public int x;
    public int y;

    public GridCoordinate(int p1, int p2)
    {
        x = p1;
        y = p2;
    }

    public static explicit operator Vector2(GridCoordinate gridCoordinate)
    {
        return new Vector2((float)gridCoordinate.x, (float)gridCoordinate.y);
    }

    public static explicit operator Vector2Int(GridCoordinate gridCoordinate)
    {
        return new Vector2Int(gridCoordinate.x, gridCoordinate.y);
    }

    public static explicit operator Vector3(GridCoordinate gridCoordinate)
    {
        return new Vector3((float)gridCoordinate.x, (float)gridCoordinate.y, 0f);
    }

    public static explicit operator Vector3Int(GridCoordinate gridCoordinate)
    {
        return new Vector3Int(gridCoordinate.x, gridCoordinate.y, 0);
    }
}
```

### Create the GridProperty.cs 
-> used the previously GridCoordinate script created in this script
1. 
``` c#
[System.Serializable]
public class GridProperty
{
	// just created type 
    public GridCoordinate gridCoordinate;
    // enum type created 
    public GridBoolProperty gridBoolProperty;
    public bool gridBoolValue = false;

    public GridProperty(GridCoordinate gridCoordinate, GridBoolProperty gridBoolProperty, bool gridBoolValue)
    {
        this.gridCoordinate = gridCoordinate;
        this.gridBoolProperty = gridBoolProperty;
        this.gridBoolValue = gridBoolValue;
    }
}
```

### Create a SO GridProperty class
-> used to created a SO object for the all tilemap to enable these boolean values to be stored in a list
-> contain in a list all booleans value of a scene
``` c#
﻿using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName = "so_GridProperties", menuName = "Scriptable Objects/Grid Properties")]
public class SO_GridProperties : ScriptableObject
{
    public SceneName sceneName;
    public int gridWidth; // max bounds of tilemap
    public int gridHeight; // max bounds of tilemap
    public int originX; // origin is always the bottom left hand square of the tilemap
    public int originY;

    [SerializeField]
    public List<GridProperty> gridPropertyList; 
}
```

### Day 8 | 09/12 (6h)
### Create the TileMapGridProperties.cs
-> The GridProperties with the boolean tilemap is initialy disable in our hierarchy.
![[Pasted image 20221209124656.png]]
-> we'r going to enable/disable them during the line time set up of the tilemap. A script will run on each GridProperties object in the hierarchy, 
initialy clear all the properties out theses SO, and then allow us to draw our boolean values on tilemap.
-> when disable, run and scan throught the all tilemap and create thoses variables of grid property type for each grid square that you can find a bool property for and then added to that SO grid property list
-> Create a "TileMapGridProperties.cs" component to attach to each of these tilemap to do that
1. 
``` c#
﻿using UnityEditor;
using UnityEngine;
using UnityEngine.Tilemaps;

// run by default when editing the game and when game run
[ExecuteAlways]
public class TilemapGridProperties : MonoBehaviour
{
#if UNITY_EDITOR
    private Tilemap tilemap;
    [SerializeField] private SO_GridProperties gridProperties = null;
    [SerializeField] private GridBoolProperty gridBoolProperty = GridBoolProperty.diggable;

    private void OnEnable()
    {
        // here whe specify that : Only populate in the editor
        if (!Application.IsPlaying(gameObject))
        {
            tilemap = GetComponent<Tilemap>();

            if (gridProperties != null)
            {
                gridProperties.gridPropertyList.Clear();
            }
        }
    }

    private void OnDisable()
    {        // Only populate in the editor
        if (!Application.IsPlaying(gameObject))
        {
            UpdateGridProperties();

            if (gridProperties != null)
            {
                // This is required to ensure that the updated gridproperties gameobject gets saved when the game is saved - otherwise they are not saved.
                EditorUtility.SetDirty(gridProperties);
            }
        }
    }

    private void UpdateGridProperties()
    {
        // Compress timemap bounds
        tilemap.CompressBounds();

        // Only populate in the editor
        if (!Application.IsPlaying(gameObject))
        {
            if (gridProperties != null)
            {
	            // bottom left corner
                Vector3Int startCell = tilemap.cellBounds.min;
                // top right corner
                Vector3Int endCell = tilemap.cellBounds.max;
				// scan the all tile map
                for (int x = startCell.x; x < endCell.x; x++)
                {
                    for (int y = startCell.y; y < endCell.y; y++)
                    {
                        TileBase tile = tilemap.GetTile(new Vector3Int(x, y, 0));

                        if (tile != null)
                        {
                            gridProperties.gridPropertyList.Add(new GridProperty(new GridCoordinate(x, y), gridBoolProperty, true));
                        }
                    }
                }
            }
        }
    }

    private void Update()
    {        // Only populate in the editor
        if (!Application.IsPlaying(gameObject))
        {
            Debug.Log("DISABLE PROPERTY TILEMAPS");
        }
    }
#endif
}
```


### Create the SO Grid Properties assets to store saved bools
1. create a new Scriptable Object Asset "so_GridProperties_Scene1_Farm" Create>New>ScriptableObject>GridProperty
-> set its coordinate and corresponding scene (origin => bottom left coordinate)
-> The grid Property List will get populated by our script TileMapGridProeperties
![[Pasted image 20221209131037.png]]
2. then do the same for each scene
![[Pasted image 20221209131655.png]]
3. then Attache the TileMap Grid Properties script to all Bool tile map with corresponding goal and attach te corresponding SO just created for each scene 
![[Pasted image 20221209132103.png]]
![[Pasted image 20221209132624.png]]
4. Now you can Enable the GridProperty GO in the hierarchy, paint tiles for custom bools value (instance layer with a blue tile ?), disable the GridProperty GO, and the coordinates of the booleans value will be saved in our so_GridProperties_"correspondingScene" ! 
5. 
``` c#
[...] 
```

## Only Drop Items Where Permitted[6.18]

goals :
-> Ensure that the player can only act on the bool tile map drawn on dev mode
->

concept :
-> 

### refactoring 
->
1. SceneSave.cs
``` c#
// change 
public Dictionary<string, List<SceneItem>> listSceneItemDictionary;
// to 
﻿using System.Collections.Generic;

[System.Serializable]
public class SceneSave
{
    public Dictionary<string, int> intDictionary;
    public Dictionary<string, bool> boolDictionary;    // string key is an identifier name we choose for this list
    public Dictionary<string, string> stringDictionary;
    public Dictionary<string, Vector3Serializable> vector3Dictionary;
    public Dictionary<string, int[]> intArrayDictionary;
    public List<SceneItem> listSceneItem;
    public Dictionary<string, GridPropertyDetails> gridPropertyDetailsDictionary;
    public List<InventoryItem>[] listInvItemArray;
}
```
2. SceneItemManager.cs
``` c#
asd
```
### create the GridPropertyDetails.cs class
->
``` c#
﻿[System.Serializable]
public class GridPropertyDetails
{
    public int gridX;
    public int gridY;
    public bool isDiggable = false;
    public bool canDropItem = false;
    public bool canPlaceFurniture = false;
    public bool isPath = false;
    public bool isNPCObstacle = false;
    public int daysSinceDug = -1;
    public int daysSinceWatered = -1;
    public int seedItemCode = -1;
    public int growthDays = -1;
    public int daysSinceLastHarvest = -1;

    public GridPropertyDetails()
    {
    }
}
```

### create the GridPropertiesManager.cs class
-> read throught all the SO object created related to booleans value on tilemap, and populate that using the grid property details into the dicitonary
``` c#
using System.Collections.Generic;
using System.Linq;
using UnityEngine;
using UnityEngine.SceneManagement;
using UnityEngine.Tilemaps;

[RequireComponent(typeof(GenerateGUID))]
public class GridPropertiesManager : SingletonMonobehaviour<GridPropertiesManager>, ISaveable
{
    private Grid grid; // define in AfterSceneLoaded()
    private Dictionary<string, GridPropertyDetails> gridPropertyDictionary; // key is coordinate locations
    [SerializeField] private SO_GridProperties[] so_gridPropertiesArray = null; 

    private string _iSaveableUniqueID;
    public string ISaveableUniqueID { get { return _iSaveableUniqueID; } set { _iSaveableUniqueID = value; } }

    private GameObjectSave _gameObjectSave;
    public GameObjectSave GameObjectSave { get { return _gameObjectSave; } set { _gameObjectSave = value; } }

    protected override void Awake()
    {
        base.Awake();

        ISaveableUniqueID = GetComponent<GenerateGUID>().GUID;
        GameObjectSave = new GameObjectSave();
    }

    private void OnEnable()
    {
        ISaveableRegister(); // register this GO in the save load manager

        EventHandler.AfterSceneLoadEvent += AfterSceneLoaded;
    }

    private void OnDisable()
    {
        ISaveableDeregister();

        EventHandler.AfterSceneLoadEvent -= AfterSceneLoaded;
    }

    private void Start()
    {
        InitialiseGridProperties();
    }


    /// <summary>
    /// This initialises the grid property dictionary with the values from the SO_GridProperties assets and stores the values for each scene in
    /// GameObjectSave sceneData
    /// </summary>
    private void InitialiseGridProperties()
    {
        // Loop through all gridproperties in the array
        foreach (SO_GridProperties so_GridProperties in so_gridPropertiesArray)
        {
            // Create dictionary of grid property details
            Dictionary<string, GridPropertyDetails> gridPropertyDictionary = new Dictionary<string, GridPropertyDetails>();

            // Populate grid property dictionary - Iterate through all the grid properties in the so gridproperties list
            foreach (GridProperty gridProperty in so_GridProperties.gridPropertyList)
            {
                GridPropertyDetails gridPropertyDetails;

                gridPropertyDetails = GetGridPropertyDetails(gridProperty.gridCoordinate.x, gridProperty.gridCoordinate.y, gridPropertyDictionary);

                if (gridPropertyDetails == null)
                {
                    gridPropertyDetails = new GridPropertyDetails();
                }

                switch (gridProperty.gridBoolProperty)
                {
                    case GridBoolProperty.diggable:
                        gridPropertyDetails.isDiggable = gridProperty.gridBoolValue;
                        break;

                    case GridBoolProperty.canDropItem:
                        gridPropertyDetails.canDropItem = gridProperty.gridBoolValue;
                        break;

                    case GridBoolProperty.canPlaceFurniture:
                        gridPropertyDetails.canPlaceFurniture = gridProperty.gridBoolValue;
                        break;

                    case GridBoolProperty.isPath:
                        gridPropertyDetails.isPath = gridProperty.gridBoolValue;
                        break;

                    case GridBoolProperty.isNPCObstacle:
                        gridPropertyDetails.isNPCObstacle = gridProperty.gridBoolValue;
                        break;

                    default:
                        break;
                }

                SetGridPropertyDetails(gridProperty.gridCoordinate.x, gridProperty.gridCoordinate.y, gridPropertyDetails, gridPropertyDictionary);
            }

            // Create scene save for this gameobject
            SceneSave sceneSave = new SceneSave();

            // Add grid property dictionary to scene save data
            sceneSave.gridPropertyDetailsDictionary = gridPropertyDictionary;

            // If starting scene set the gridPropertyDictionary member variable to the current iteration
            if (so_GridProperties.sceneName.ToString() == SceneControllerManager.Instance.startingSceneName.ToString())
            {
                this.gridPropertyDictionary = gridPropertyDictionary;
            }

            // Add bool dictionary and set first time scene loaded to true
            sceneSave.boolDictionary = new Dictionary<string, bool>();
            sceneSave.boolDictionary.Add("isFirstTimeSceneLoaded", true);


            // Add scene save to game object scene data
            GameObjectSave.sceneData.Add(so_GridProperties.sceneName.ToString(), sceneSave);
        }
    }

    private void AfterSceneLoaded()
    {
        // Get Grid after loaded scene
        grid = GameObject.FindObjectOfType<Grid>();
    }

    /// <summary>
    /// Returns the gridPropertyDetails at the gridlocation for the supplied dictionary, or null if no properties exist at that location.
    /// </summary>
    public GridPropertyDetails GetGridPropertyDetails(int gridX, int gridY, Dictionary<string, GridPropertyDetails> gridPropertyDictionary)
    {
        // Construct key from coordinate
        string key = "x" + gridX + "y" + gridY;

        GridPropertyDetails gridPropertyDetails;

        // Check if grid property details exist forcoordinate and retrieve
        if (!gridPropertyDictionary.TryGetValue(key, out gridPropertyDetails))
        {
            // if not found
            return null;
        }
        else
        {
            return gridPropertyDetails;
        }
    }


    /// <summary>
    /// Get the grid property details for the tile at (gridX,gridY).  If no grid property details exist null is returned and can assume that all grid property details values are null or false
    /// </summary>
    public GridPropertyDetails GetGridPropertyDetails(int gridX, int gridY)
    {
        return GetGridPropertyDetails(gridX, gridY, gridPropertyDictionary);
    }

    /// <summary>
    /// for sceneName this method returns a Vector2Int with the grid dimensions for that scene, or Vector2Int.zero if scene not found
    /// </summary>

    public bool GetGridDimensions(SceneName sceneName, out Vector2Int gridDimensions, out Vector2Int gridOrigin)
    {
        gridDimensions = Vector2Int.zero;
        gridOrigin = Vector2Int.zero;

        // loop through scenes
        foreach (SO_GridProperties so_GridProperties in so_gridPropertiesArray)
        {
            if (so_GridProperties.sceneName == sceneName)
            {
                gridDimensions.x = so_GridProperties.gridWidth;
                gridDimensions.y = so_GridProperties.gridHeight;

                gridOrigin.x = so_GridProperties.originX;
                gridOrigin.y = so_GridProperties.originY;

                return true;
            }
        }

        return false;
    }



    public void ISaveableDeregister()
    {
        SaveLoadManager.Instance.iSaveableObjectList.Remove(this);
    }

    public void ISaveableRegister()
    {
        SaveLoadManager.Instance.iSaveableObjectList.Add(this);
    }

    public void ISaveableRestoreScene(string sceneName)
    {
        // Get sceneSave for scene - it exists since we created it in initialise
        if (GameObjectSave.sceneData.TryGetValue(sceneName, out SceneSave sceneSave))
        {
            // get grid property details dictionary - it exists since we created it in initialise
            if (sceneSave.gridPropertyDetailsDictionary != null)
            {
                gridPropertyDictionary = sceneSave.gridPropertyDetailsDictionary;
            }
        }
    }

    public GameObjectSave ISaveableSave()
    {
        // Store current scene data
        ISaveableStoreScene(SceneManager.GetActiveScene().name);

        return GameObjectSave;
    }


    public void ISaveableStoreScene(string sceneName)
    {
        // Remove sceneSave for scene
        GameObjectSave.sceneData.Remove(sceneName);

        // Create sceneSave for scene
        SceneSave sceneSave = new SceneSave();

        // create & add dict grid property details dictionary
        sceneSave.gridPropertyDetailsDictionary = gridPropertyDictionary;

        // Add scene save to game object scene data
        GameObjectSave.sceneData.Add(sceneName, sceneSave);
    }

    /// <summary>
    /// Set the grid property details to gridPropertyDetails for the tile at (gridX,gridY) for current scene
    /// </summary>
    public void SetGridPropertyDetails(int gridX, int gridY, GridPropertyDetails gridPropertyDetails)
    {
        SetGridPropertyDetails(gridX, gridY, gridPropertyDetails, gridPropertyDictionary);
    }

    /// <summary>
    /// Set the grid property details to gridPropertyDetails for the tile at (gridX,gridY) for the gridpropertyDictionary.
    /// </summary>
    public void SetGridPropertyDetails(int gridX, int gridY, GridPropertyDetails gridPropertyDetails, Dictionary<string, GridPropertyDetails> gridPropertyDictionary)
    {
        // Construct key from coordinate
        string key = "x" + gridX + "y" + gridY;

        gridPropertyDetails.gridX = gridX;
        gridPropertyDetails.gridY = gridY;

        // Set value
        gridPropertyDictionary[key] = gridPropertyDetails;
    }
}
```

### Update Settings.cs
1. add 
``` c#
public const float gridCellSize = 1f; // grid cell size in unity units
```


### Update UIInventorySlot.cs
->
1. and the methods DropSelectedItemAtMousePosition() (instantiate item dropped by player)
``` c#
  
[...]UIInventorySlot
/// <summary>
/// Drops the item (if selected) at the current mouse position. Called by the DropItem event.
/// </summary>
private void DropSelectedItemAtMousePosition()
{
if (itemDetails != null && isSelected)
{
Vector3 worldPosition = mainCamera.ScreenToWorldPoint(new Vector3(Input.mousePosition.x, Input.mousePosition.y, -mainCamera.transform.position.z));

// if can drop item here
Vector3Int gridPosition = GridPropertiesManager.Instance.grid.WorldToCell(worldPosition);
GridPropertyDetails gridPropertyDetails = GridPropertiesManager.Instance.GetGridPropertyDetails(gridPosition.x, gridPosition.y);
if (gridPropertyDetails != null && gridPropertyDetails.canDropItem)
{
// Create item from prefab at mouse position
GameObject itemGameObject = Instantiate(itemPrefab, new Vector3(worldPosition.x, worldPosition.y - Settings.gridCellSize / 2f, worldPosition.z), Quaternion.identity, parentItem); // drop at exact cursor pointer, defaut is grid placed
Item item = itemGameObject.GetComponent<Item>();
item.ItemCode = itemDetails.itemCode;

// Remove item from players inventory
InventoryManager.Instance.RemoveItem(InventoryLocation.player, item.ItemCode);

// If no more of item then clear selected
if (InventoryManager.Instance.FindItemInInventory(InventoryLocation.player, item.ItemCode) == -1)
{
ClearSelectedItem();
}
}
}
}
```

### Create our GamePropertiesManager GO In the hierarchy
->
1. Create an empty GO "GamePropertiesManager" and add the "Grid Properties Manager" script, and populate so_grid Properties from different scenes created previously (tilemap boolean) 

## Implement a grid Cursor[6.18]

goals :
-> Implement a cursor to visualy represent where the player may place item (green / red square)
->

concept :
-> 

### Work on the InventoryManager.cs
->
1. add 2 methods :
``` c#
[...] InventoryManager.cs

/// <summary>
/// Returns the itemDetails (from the SO_ItemList) for the currently selected item in the inventoryLocation , or null if an item isn't selected
/// </summary>

public ItemDetails GetSelectedInventoryItemDetails(InventoryLocation inventoryLocation)
{
	int itemCode = GetSelectedInventoryItem(inventoryLocation);
	
	if (itemCode == -1)
	{
		return null;
	}
	else
	{
		return GetItemDetails(itemCode);
	}
}

/// <summary>
/// Get the selected item for inventoryLocation - returns itemCode or -1 if nothing is selected
/// </summary>

private int GetSelectedInventoryItem(InventoryLocation inventoryLocation)
{
	return selectedInventoryItem[(int)inventoryLocation];
}
```
2. asd

### Add the GridCursor.cs Class
->
1. asd
``` c#
﻿using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class GridCursor : MonoBehaviour
{
    private Canvas canvas; // use to store the canvas used by the grid cursor (ui canvas)
    private Grid grid; // grid of the actual tilemap
    private Camera mainCamera; 
    // populated in the inspector from assets in hierarchy 
    [SerializeField] private Image cursorImage = null;
    [SerializeField] private RectTransform cursorRectTransform = null;
    [SerializeField] private Sprite greenCursorSprite = null;
    [SerializeField] private Sprite redCursorSprite = null;
    [SerializeField] private SO_CropDetailsList so_CropDetailsList = null;

	// bool ref for cursor status + getter / setter
    private bool _cursorPositionIsValid = false;
    public bool CursorPositionIsValid { get => _cursorPositionIsValid; set => _cursorPositionIsValid = value; }
	//populated with the grid radius using of the selected item
    private int _itemUseGridRadius = 0;
    public int ItemUseGridRadius { get => _itemUseGridRadius; set => _itemUseGridRadius = value; }

    private ItemType _selectedItemType;
    public ItemType SelectedItemType { get => _selectedItemType; set => _selectedItemType = value; }
	// if the cursor is display or not
    private bool _cursorIsEnabled = false;
    public bool CursorIsEnabled { get => _cursorIsEnabled; set => _cursorIsEnabled = value; }

	// subscribe to afterSceneLoadEvent to find the and populate the grid ref 
    private void OnDisable()
    {
        EventHandler.AfterSceneLoadEvent -= SceneLoaded;
    }

    private void OnEnable()
    {
        EventHandler.AfterSceneLoadEvent += SceneLoaded;
    }

    // Start is called before the first frame update
    private void Start()
    {
        mainCamera = Camera.main;
        canvas = GetComponentInParent<Canvas>();
    }

    // Update is called once per frame
    private void Update()
    {
        if (CursorIsEnabled)
        {
            DisplayCursor();
        }
    }

    private Vector3Int DisplayCursor()
    {
        if (grid != null)
        {
            // Get grid position for cursor
            Vector3Int gridPosition = GetGridPositionForCursor();

            // Get grid position for player
            Vector3Int playerGridPosition = GetGridPositionForPlayer();

            // Set cursor sprite
            SetCursorValidity(gridPosition, playerGridPosition);

            // Get rect transform position for cursor
            cursorRectTransform.position = GetRectTransformPositionForCursor(gridPosition);

            return gridPosition;
        }
        else
        {
            return Vector3Int.zero;
        }
    }

    private void SceneLoaded()
    {
        grid = GameObject.FindObjectOfType<Grid>();
    }

    private void SetCursorValidity(Vector3Int cursorGridPosition, Vector3Int playerGridPosition)
    {
        SetCursorToValid();

        // Check item use radius is valid
        if (Mathf.Abs(cursorGridPosition.x - playerGridPosition.x) > ItemUseGridRadius
            || Mathf.Abs(cursorGridPosition.y - playerGridPosition.y) > ItemUseGridRadius)
        {
            SetCursorToInvalid();
            return;
        }

        // Get selected item details
        ItemDetails itemDetails = InventoryManager.Instance.GetSelectedInventoryItemDetails(InventoryLocation.player);

        if (itemDetails == null)
        {
            SetCursorToInvalid();
            return;
        }

        // Get grid property details at cursor position
        GridPropertyDetails gridPropertyDetails = GridPropertiesManager.Instance.GetGridPropertyDetails(cursorGridPosition.x, cursorGridPosition.y);

        if (gridPropertyDetails != null)
        {
            // Determine cursor validity based on inventory item selected and grid property details
            switch (itemDetails.itemType)
            {
                case ItemType.Seed:
                    if (!IsCursorValidForSeed(gridPropertyDetails))
                    {
                        SetCursorToInvalid();
                        return;
                    }
                    break;

                case ItemType.Commodity:

                    if (!IsCursorValidForCommodity(gridPropertyDetails))
                    {
                        SetCursorToInvalid();
                        return;
                    }
                    break;

                case ItemType.none:
                    break;

                case ItemType.count:
                    break;

                default:
                    break;
            }
        }
        else
        {
            SetCursorToInvalid();
            return;
        }
    }

    /// <summary>
    /// Set the cursor to be invalid
    /// </summary>
    private void SetCursorToInvalid()
    {
        cursorImage.sprite = redCursorSprite;
        CursorPositionIsValid = false;
    }

    /// <summary>
    /// Set the cursor to be valid
    /// </summary>
    private void SetCursorToValid()
    {
        cursorImage.sprite = greenCursorSprite;
        CursorPositionIsValid = true;
    }

    /// <summary>
    /// Test cursor validity for a commodity for the target gridPropertyDetails. Returns true if valid, false if invalid
    /// </summary>
    private bool IsCursorValidForCommodity(GridPropertyDetails gridPropertyDetails)
    {
        return gridPropertyDetails.canDropItem;
    }

    /// <summary>
    /// Set cursor validity for a seed for the target gridPropertyDetails. Returns true if valid, false if invalid
    /// </summary>
    private bool IsCursorValidForSeed(GridPropertyDetails gridPropertyDetails)
    {
        return gridPropertyDetails.canDropItem;
    }

    public void DisableCursor()
    {
        cursorImage.color = Color.clear;

        CursorIsEnabled = false;
    }

    public void EnableCursor()
    {
        cursorImage.color = new Color(1f, 1f, 1f, 1f);
        CursorIsEnabled = true;
    }

    public Vector3Int GetGridPositionForCursor()
    {
        Vector3 worldPosition = mainCamera.ScreenToWorldPoint(new Vector3(Input.mousePosition.x, Input.mousePosition.y, -mainCamera.transform.position.z));  // z is how far the objects are in front of the camera - camera is at -10 so objects are (-)-10 in front = 10
        return grid.WorldToCell(worldPosition);
    }

    public Vector3Int GetGridPositionForPlayer()
    {
        return grid.WorldToCell(Player.Instance.transform.position);
    }

    public Vector2 GetRectTransformPositionForCursor(Vector3Int gridPosition)
    {
        Vector3 gridWorldPosition = grid.CellToWorld(gridPosition);
        Vector2 gridScreenPosition = mainCamera.WorldToScreenPoint(gridWorldPosition);
        return RectTransformUtility.PixelAdjustPoint(gridScreenPosition, cursorRectTransform, canvas);
    }
}
```

### Modify the UIInventorySlot.cs 
->
1. asd
``` c#
[...] UIInventorySlot.cs

private GridCursor gridCursor;

private void Start()
{
	mainCamera = Camera.main;
	gridCursor = FindObjectOfType<GridCursor>();
}

  

private void ClearCursors()
{
// Disable cursor
gridCursor.DisableCursor();
cursor.DisableCursor();

// Set item type to none
gridCursor.SelectedItemType = ItemType.none;
cursor.SelectedItemType = ItemType.none;
}

  
// update
/// <summary>
/// Sets this inventory slot item to be selected
/// </summary>

private void SetSelectedItem()
{
// Clear currently highlighted items
inventoryBar.ClearHighlightOnInventorySlots();

// Highlight item on inventory bar
isSelected = true;

// Set highlighted inventory slots
inventoryBar.SetHighlightedInventorySlots();

// here
// Set use radius for cursors
gridCursor.ItemUseGridRadius = itemDetails.itemUseGridRadius;

// If item requires a grid cursor then enable cursor
if (itemDetails.itemUseGridRadius > 0)
{
gridCursor.EnableCursor();
}
else
{
gridCursor.DisableCursor();
}

// If item requires a cursor then enable cursor
if (itemDetails.itemUseRadius > 0f)
{
cursor.EnableCursor();
}
else
{
cursor.DisableCursor();
}

// Set item type
gridCursor.SelectedItemType = itemDetails.itemType;
// cursor.SelectedItemType = itemDetails.itemType;

// Set item selected in inventory
InventoryManager.Instance.SetSelectedInventoryItem(InventoryLocation.player, itemDetails.itemCode);

if (itemDetails.canBeCarried == true)
{
// Show player carrying item
Player.Instance.ShowCarriedItem(itemDetails.itemCode);
}
else // show player carrying nothing
{
Player.Instance.ClearCarriedItem();
}
}


public void ClearSelectedItem()
{
ClearCursors(); // here new

// Clear currently highlighted items
inventoryBar.ClearHighlightOnInventorySlots();

isSelected = false;  

// set no item selected in inventory
InventoryManager.Instance.ClearSelectedInventoryItem(InventoryLocation.player);

// Clear player carrying item
Player.Instance.ClearCarriedItem();
}

/// <summary>
/// Drops the item (if selected) at the current mouse position. Called by the DropItem event.
/// </summary>

private void DropSelectedItemAtMousePosition()
{
if (itemDetails != null && isSelected)
{
// If a valid cursor position
if (gridCursor.CursorPositionIsValid)
{
Vector3 worldPosition = mainCamera.ScreenToWorldPoint(new Vector3(Input.mousePosition.x, Input.mousePosition.y, -mainCamera.transform.position.z));

// Create item from prefab at mouse position
GameObject itemGameObject = Instantiate(itemPrefab, new Vector3(worldPosition.x, worldPosition.y - Settings.gridCellSize / 2f, worldPosition.z), Quaternion.identity, parentItem);
Item item = itemGameObject.GetComponent<Item>();
item.ItemCode = itemDetails.itemCode;

// Remove item from players inventory
InventoryManager.Instance.RemoveItem(InventoryLocation.player, item.ItemCode);

// If no more of item then clear selected
if (InventoryManager.Instance.FindItemInInventory(InventoryLocation.player, item.ItemCode) == -1)
{
ClearSelectedItem();
}
}
}
}
```

### Setup the GO for the grid cursor
->
1. In the UI>UICanvasGroup create a UI>Panel named "UIPanel"
-> move it above UIInventoryBar in the hierarchy
-> set the Source Image to None
-> set its alpha value to 0
-> and add a [Canvas Group] and disable [Blocks Raycast]
-> add the [Grid Cursor] script

2. Add Child GO under UIPanel and populate diff component to grid cursor script
-> create empty "GridCursor"
-> add an [Image] component
-> set source image to GreenGridCursors
-> SetNativeSize
-> set color alpha to 0
-> set pivot X : 0 / Y : 0
-> pos X / Y : -8

-> then set up the grid cursor component of the UI Panel
![[Pasted image 20221209204938.png]]


## Click Drop Items[6.18]

goals :
-> allow player to drop item selected not only dragged from inventory bar
->

concept :
-> Add new functionality to the Player.cs class to handle Player Click Input. That click input will generate an event (created in our eventhandler class) and the ui slot script will listen to this item to drop the item

### EventHandler.cs
->
1. add the new event
``` c#
// Drop selected item event
public static event Action DropSelectedItemEvent;

public static void CallDropSelectedItemEvent()
{
if (DropSelectedItemEvent != null)
	DropSelectedItemEvent();
}
```
2. asd

### Modify Player.cs to handle click input
->
1. asd
``` c#
// create a start methods
private void Start()
{
gridCursor = FindObjectOfType<GridCursor>();
}

  
// update this one
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

PlayerClickInput(); // new call here

PlayerTestInput();
[...]
}
#endregion Player Input
}


private void PlayerClickInput()
{
	if (Input.GetMouseButton(0))
	{
		if (gridCursor.CursorIsEnabled)
		{
			ProcessPlayerClickInput();
		}
	}
}


private void ProcessPlayerClickInput()
{
	ResetMovement();
	// Get Selected item details
	ItemDetails itemDetails = InventoryManager.Instance.GetSelectedInventoryItemDetails(InventoryLocation.player);
	if (itemDetails != null)
	{
		switch (itemDetails.itemType)
		{
			case ItemType.Seed:
			if (Input.GetMouseButtonDown(0))
			{
			ProcessPlayerClickInputSeed(itemDetails);
			}
			break;
			
			case ItemType.Commodity:
			if (Input.GetMouseButtonDown(0))
			{
			ProcessPlayerClickInputCommodity(itemDetails);
			}
			break;
			
			case ItemType.none:
			break;
			
			case ItemType.count:
			break;
			
			default:
			break;
		}
	}
}


private void ProcessPlayerClickInputSeed(ItemDetails itemDetails)
{
	if (itemDetails.canBeDropped && gridCursor.CursorPositionIsValid)
	{
		EventHandler.CallDropSelectedItemEvent();
	}
}

private void ProcessPlayerClickInputCommodity(ItemDetails itemDetails)
{
	if (itemDetails.canBeDropped && gridCursor.CursorPositionIsValid)
	{
		EventHandler.CallDropSelectedItemEvent();
	}
}
```


### UIInventorySlot.cs update
-> on enable subscribe to event 
1. asd
``` c#
  

private void OnDisable()
{
	EventHandler.AfterSceneLoadEvent -= SceneLoaded;
	EventHandler.DropSelectedItemEvent -= DropSelectedItemAtMousePosition;
}

private void OnEnable()
{
	EventHandler.AfterSceneLoadEvent += SceneLoaded;
	EventHandler.DropSelectedItemEvent += DropSelectedItemAtMousePosition;
}
```


