# Inventory & Items

## Item Details Class[6.1]

goals :
-> Create the ItemDetails.cs class

concept :
-> 

### Add New Item Type Enum 
1. In our Enums.cs, add a new public enum with all kind of item type in the game to be referenced in our code :
``` c#
[...] Enums.cs
public enum ItemType
{
Seed,
Commodity,
Watering_tool,
Hoeing_tool,
Chopping_tool,
Breaking_tool,
Reaping_tool,
Collecting_tool,
Reapable_scenary,
Furniture,
none,
count // count of how many item type exists in this enum
}
```
2. asd
### Create the ItemDetails.cs
->
1. asd
``` c#
using UnityEngine;

// Required to be used as Scriptable Object
[System.Serializable]
public class ItemDetails
{
	public int itemCode;
	public ItemType itemType;
	public string itemDescription;
	public Sprite itemSprite;
	public string itemLongDescription;
	public short itemUseGridRadius; // might want to chop trees 1-2 grid space away from us
	public float itemUseRadius; // if its not grid based, (ex scythe) mesure in unity units (distance based?)
	public bool isStartingItem;
	public bool canBePickedUp;
	public bool canBeDropped;
	public bool canBeEaten;
	public bool canBeCarried;
}
```

## Scriptable Object Item List[6.2]

goals :
-> Create a SO which gonna contain all items of our game with the ItemDetails class created previously
-> By using a SO asset, each Items will persist as an asset and can be added or deleted as required

concept :
-> 

### Create  SO_ItemList.cs
-> Because its a Scriptable Object, its doesnt derive from Monobehaviour but from a class named "ScriptableObject"
-> The attribute Create Asset Menu allow to create new asset from the unity UI
``` c#
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu(fileName="so_ItemList", menuName="Scriptable Objects/Item/Item List")]

public class SO_ItemList : ScriptableObject
{
	[SerializeField]
	public List<ItemDetails> itemDetails;
}
```

### Create and populate the Scriptable Object ItemList
-> Create a list of all existing items in the game to access them easily throught code !

1. For that Create>Scriptable Object>Item>ItemList
2. And populate ! 
![[Pasted image 20221204120633.png]]

## Item Class and Commodity Prefabs[6.3]

goals :
-> Create the Item Class to identify GO as items 
-> and create item prefabs for our commodities and place them in our scene

concept :
-> 

### Create the Item.cs Class
->
1. Create new scripts named "Item"
``` c#
using UnityEngine;

public class Item : MonoBehaviour
{
	[SerializeField]
	
	private int _itemCode;
	
	private SpriteRenderer spriteRenderer;
	
	public int ItemCode {get { return _itemCode;} set { _itemCode=value;}}
	
	private void Awake()
	{
		spriteRenderer = GetComponentInChildren<spriteRenderer>();
	}
	
	private void Start()
	{
		if (ItemCode != 0)
		{
			Init(ItemCode);
		}
	}
	
	public void Init(int itemCodeParam)
	{
	 // empty for now
	}

}
```
2. asd

### Create an Item Exemple Prefab
-> Base Item GO With all settings to be dup for creating new item

1. Create new GO "Items" in the scene with a child named "Item"
2. Add a [Box Collider2D] component to Item with and [Offset Y] of 0.5 and add the [Item.cs Class] as a component 
3. Create a child "ItemSprite" under Item to attach a [Sprite Renderer] component to wear the displayed image. Set its [Sprite Sort Point] to pivot and [Sorting Layer] to "Instance"
4. Drag the Item GO created to Prefab, and delete the "Item" from hierarchy 

	and now we have an exemple Item GO that we can dup !
### Create Prefab Variant for Divers Items
-> 
1. Click on the Item Prefab Create>Prefab Variant, named here "Corn"
2. set its [Sprite Renderer] to the Corn Sprite for ex
3. Size its [Box Collider 2d]
4. /!| Set its [ItemCode] (for ex to 10002) in the [Item.cs] component
5. And Repeat for all items !

## InventoryManager Class[6.4]

goals :
-> Used to manage inventory and item details
-> Used in conjonction with an Item Pickup Script, so when the player walk throught an item, details are printed over the console

concept :
-> We'll initialy populate the class with few methods;
- one that populate an item dictionnary with the item details held in the Scriptable Object Item List we've created
- And an other one that return item details for a given item code
-> Then create a pick up script for first show the description of the item as the player walks over an item

### Create the InventoryManager.cs
-> inherit from SingletonMonobehaviour for a single instance per game
1. We first start by creating and setting a dictionnary, which we'll be used to hold the inventory items created in the SO Item List. Make items more accessible, item details can be accessed from dict by item code directly
2. Create the GetItemDetails() methods which retrieve itemDetails from dictionary by given itemCode
``` c#
using System.Collections.Generic;
using UnityEngine;

public class InventoryManager : SingletonMonobehaviour<InventoryManager>
{
private Dictionary<int, ItemDetails> itemDetailsDictionary;

[SerializeField] private SO_Itemlist itemList = null;

private void Start()
{
	// Create item details dictionary
	CreateItemDetailsDictionary();
	}
	  
	// Populates the itemDetailsDictionary from the Scriptable Object Item List
	private void CreateItemDetailsDictionary()
	{
		itemDetailsDictionary = new Dictionary<int, ItemDetails>();
	
	foreach(ItemDetails itemDetails in itemList.itemDetails)
		{
			itemDetailsDictionary.Add(itemDetails.itemCode, itemDetails);
		}
	}

		// Returns the itemDetails (From SO Item List) for the itemCode, or null if the item doesn't exist
	public ItemDetails GetItemDetails(int itemCode)
	
	{
		ItemDetails itemDetails;
		
		if (itemDetailsDictionary.TryGetValue(itemCode, out itemDetails))
		{
			return itemDetails;
		}
		else
		{
			return null;
		}
	}
}
```

3. Create the InventoryManager GO In the Persistent RC>Game Object>Create Empty named "Inventory Manager", then add it the [InventoryManager.cs] Component just created
4. Set the SO_ItemList to the [Item List] Field of the Inventory Manager Component, and thats it ! Our Dictionary is now feed and may retrieve data that we create
 
### Create the Item Pick Up
-> attached to the player to test our InventoryManager Class (when collided, print the item details of the item)
1. asd
``` c#
using UnityEngine;

public class ItemPickUp : MonoBehaviour
{
	private void OnTriggerEnter2D(Collider2D collision)
	{
		Item item = collision.GetComponent<Item>();
		
		if (item != null)
		{
		// if we encounter an item, retrieve the item details
		// InventoryManager is a Singleton, so we can access its {Instance} created in the scene to access the methods GetItemDetail()
			ItemDetails itemDetails = InventoryManager.Instance.GetItemDetails(item.ItemCode);
			
			// print returned item detail
			Debug.Log(itemDetails.itemDescription);
		}
	}
}
```

## Custom Property Attribute Drawer[6.5]

goals :
-> make a custom interface in unity GUI for every Item Prefab Variant and draw its related itemDetails if the itemCode is found
->

concept :
-> Standard Unity Attributs [Range(1000,10100)] for slider in GUI  (like the [SerializeField] which make  appear in the gui the target (like itemCode for ex))
-> Unity require every editor scripts to be in an 'Editor' folder to be display on GUI

### Creation of ItemCodeDescriptionAttribute.cs
-> To Create a custom property attribute, you need to define it as a class and inherit a special class "Property Attribute"
``` c#
using UnityEngine;

public class ItemCodeDescriptionAttribute : PropertyAttribute
{
// No values need to be held for the item code description attribute (not like the range attribute which required manual input for ex)
// So the class can be empty
}
```
2. asd

### ItemCodeDescriptionDrawer.cs
->
1. asd
``` c#
using System.Collections.Generic;
using UnityEngine;
using UnityEditor; // required for editor script
using System;
  
// indicate which custom property attribute its relate to
[CustomPropertyDrawer(typeof(ItemCodeDescriptionAttribute))]
public class ItemCodeDescriptionDrawer : PropertyDrawer
{
  
	// override initial methods here because we draw item code AND item description
	public override float GetPropertyHeight(SerializedProperty property, GUIContent label)
	{
	// Change the returned property height to be double to cater for the additional item code description that we will draw
	return EditorGUI.GetPropertyHeight(property) * 2;
	}
	
	// Here we draw the property in the unity inspector
	public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
	{
	// Using BeginProperty / EndProperty on the parent property means that prefab override logic works on the entire property.
	
	EditorGUI.BeginProperty(position, label, property);
	
	if (property.propertyType == SerializedPropertyType.Integer)
	{
	  
	EditorGUI.BeginChangeCheck(); // Start of check for changed values
	
// Draw item code with 1/2 of all heigh calculated in the GUI
	var newValue = EditorGUI.IntField(new Rect(position.x, position.y, position.width, position.height / 2), label, property.intValue);
	
	// Draw item description
	EditorGUI.LabelField(new Rect(position.x, position.y + position.height / 2, position.width, position.height / 2), "Item Description", GetItemDescription(property.intValue));
	  
	// If item code value has changed, then set value to new value
	if (EditorGUI.EndChangeCheck())
	{
	property.intValue = newValue;
	}
	}
	  
	EditorGUI.EndProperty();
	}
	
	private string GetItemDescription(int itemCode)
	{
	SO_ItemList so_itemList;
	
	so_itemList = AssetDatabase.LoadAssetAtPath("Assets/Scriptable Object Assets/Item/so_ItemList.asset",typeof(SO_ItemList)) as SO_ItemList;
	
	List<ItemDetails> itemDetailsList = so_itemList.itemDetails;
	
	ItemDetails itemDetail = itemDetailsList.Find(x => x.itemCode == itemCode);
	
	if (itemDetail != null)
	{
	return itemDetail.itemDescription;
	}
	else
	{
	return "";
	}
	}
}
```
2. then add the [ItemCodeDescription] to Item.cs
``` c#
public class Item : MonoBehaviour
{
[ItemCodeDescription]
[...]
```
3. the item description should now appear on the inspector ! 
![[Pasted image 20221205000457.png]]
## Create Item Nudge Class[6.6]

goals :
-> Class which provide a wooble visual effect on item when passing by
-> Upgrade Item.cs to automatically add this script to desired item
![[Pasted image 20221205000747.png]]
concept :
-> Item has a box collider 2d, when player enter the collider, OnEnter / onExit 

### Create the ItemNudge.cs Script
->
1. we'll play with the coroutine again here 
``` c#
using System.Collections;
using UnityEngine;

public class ItemNudge : MonoBehaviour
{
private WaitForSeconds pause;
private bool isAnimating = false;

private void Awake()
{
pause = new WaitForSeconds(0.04f);
}

  private void OnTriggerEnter2D(Collider2D collision)
{
if (isAnimating == false)
{

if (gameObject.transform.position.x < collision.gameObject.transform.position.x)
{
StartCoroutine(RotateAntiClock());
}
else
{
StartCoroutine(RotateClock());
}
}
}

private void OnTriggerExit2D(Collider2D collision)
{
if (isAnimating == false)
{
if (gameObject.transform.position.x > collision.gameObject.transform.position.x)
{
StartCoroutine(RotateAntiClock());
}
else
{
StartCoroutine(RotateClock());
}
}
}

private void RotateAntiClock()
{}

private void RotateClock()
{}

}
```
2. then create the rotates methods 
``` c# 
    private IEnumerator RotateAntiClock()
    {
        isAnimating = true;
        for (int i = 0; i < 4; i++)
        {
    gameObject.transform.GetChild(0).Rotate(0f, 0f, 2f);
            yield return pause;
        }
        for (int i = 0; i < 5; i++)
        {
    gameObject.transform.GetChild(0).Rotate(0f, 0f, -2f);
            yield return pause;
        }
gameObject.transform.GetChild(0).Rotate(0f, 0f, 2f);
        yield return pause;
        isAnimating = false;
    }

    private IEnumerator RotateClock()
    {
        isAnimating = true;
        for (int i = 0; i < 4; i++)
        {
    gameObject.transform.GetChild(0).Rotate(0f, 0f, -2f);
            yield return pause;
        }
        for (int i = 0; i < 5; i++)
        {
    gameObject.transform.GetChild(0).Rotate(0f, 0f, 2f);
            yield return pause;
        }
    gameObject.transform.GetChild(0).Rotate(0f, 0f, -2f);
        yield return pause;
        isAnimating = false;
    }
```
3. ds

### Create the Init Methods in Item.cs 
-> attribute correct sprite & effect at item initialisation
1. Complete the Init() proto left empty until now
``` c#
    public void Init (int itemCodeParam)
    {
        if (itemCodeParam != 0)
        {
            ItemCode = itemCodeParam;
            ItemDetails itemDetails = InventoryManager.Instance.GetItemDetails(ItemCode);
            spriteRenderer.sprite = itemDetails.itemSprite;
            // If item type is reapable then add nudgeable component
            if (itemDetails.itemType == ItemType.Reapable_scenary)
            {           gameObject.AddComponent<ItemNudge>();
            }
        }
    }
```

### Create Reapable Scenary Prefab
1. Make a prefab variant from the Item Prefab Template (cactus and grass)

### Fix Dictionary populate order
-> Because Awake run before Start, and we need the dictionnary to fetch in required data in other class Start() methods
``` c#
%%from%%
private void Start()

{

// Create item details dictionary

CreateItemDetailsDictionary();

}

%%to%% 
protected override void Awake()

{

base.Awake();

// Create item details dictionary

CreateItemDetailsDictionary();

}
```

## Player Item Pickup[6.7]

goals :
-> Allow player to pick up item when walking over it
-> for now, print the "inventory" details in the console; item name / qty 

concept :
-> 

### Add a new Enum in Enums.cs
-> related to the inventory location, we could add multi storage location, like a storage chest, house or bank for ex
1. asd
``` c# [Enums.cs]
public enum InventoryLocation

{
player,
chest,
count
}
```
2. asd

### Create a InventoryItem struct which represent 1 item categorie of our inventory 
->contain the item code and the qty possessed by the player
``` c#
[System.Serializable] // used to cache inventory for saving games

public struct InventoryItem
{
public int itemCode;
public int itemQuantity;
}
```

### Add fields to Settings.cs
->contain the item code and the qty possessed by the player
``` c# [...] Settings.cs
// Inventory

public static int playerInitialInventoryCapacity = 24;
public static int playerMaximumInventoryCapacity = 48;
```

### Day 4 | 5/23 (10h)
### Create a new event in the EventHandler.cs class
->trigger event when inventory has been changed. when a player pickup an item the GUI may want to know about it =)
1. here we can use the built in "Action" event, unless the mvmt event with 16+ params which required 
to specified a delegate 
``` c# [...] EventHandler.cs
using System;
using System.Collections.Generic;
using UnityEngine;
// adding new namespace

// Inventory Updated Event
public static event Action<InventoryLocation, List<InventoryItem>> InventoryUpdatedEvent;

public static void CallInventoryUpdatedEvent(InventoryLocation inventoryLocation, List<InventoryItem> inventoryList)
{
if (InventoryUpdatedEvent != null)
InventoryUpdatedEvent(inventoryLocation, inventoryList);
}
```

### Update our InventoryManager.cs Class
-> 
``` c# [InventoryManager.cs]
// 2 new variables
////////////////////////////////
public List<InventoryItem>[] inventoryLists; // 0 = player, 1 = chest, etc

[HideInInspector] public int[] inventoryListCapacityIntArray; // 0 = player inventory capacity (slots)
//////////////////////////////////

// Add CreateInventoryList
////////////////////////////////
protected override void Awake()
{
base.Awake();

// Create Inventory List
CreateInventoryList();

// Create item details dictionary
CreateItemDetailsDictionary();
}
////////////////////////////////

// add new inventory methods
////////////////////////////////
/// <summary>
// Set initial inventory (numbers / slots)
/// </summary>
private void CreateInventoryList()
{
inventoryLists = new List<InventoryItem>[(int)InventoryLocation.count];

for (int i = 0; i < (int)InventoryLocation.count; i++)
{
inventoryLists[i] = new List<InventoryItem>();
// 0 = player inv, 1 chest
}

// Initialise the inventory list capacity array (array of 2)
inventoryListCapacityIntArray = new int[(int)InventoryLocation.count];

// Initialize player inventory list capacity
inventoryListCapacityIntArray[(int)InventoryLocation.player] = Settings.playerInitialInventoryCapacity;
}

/// <summary>
/// Add an item to the inventory list for the inventoryLocation and then destroy the gameObjectToDelete (overloaded methods, additional parameters)
/// </summary>
public void AddItem(InventoryLocation inventoryLocation, Item item, GameObject gameObjectToDelete)
{
AddItem(inventoryLocation, item);

Destroy(gameObjectToDelete);
}

/// <summary>
// Add an item to the inventory list for the inventory location
/// </summary>
public void AddItem(InventoryLocation inventoryLocation, Item item)
{
int itemCode = item.ItemCode;
List<InventoryItem> inventoryList = inventoryLists[(int)inventoryLocation];

// check if inventory already contain the item
int itemPosition = FindItemInInventory(inventoryLocation, itemCode);

if(itemPosition != -1)
{
AddItemAtPosition(inventoryList, itemCode, itemPosition);
}
else
{
AddItemAtPosition(inventoryList, itemCode);
}

// Send event that inventory has been updated;
EventHandler.CallInventoryUpdatedEvent(inventoryLocation, inventoryLists[(int)inventoryLocation]);
}

/// <summary>
/// Add item to the end of the inventory
/// </summary>
private void AddItemAtPosition(List<InventoryItem> inventoryList, int itemCode)
{
InventoryItem inventoryItem = new InventoryItem();

inventoryItem.itemCode = itemCode;
inventoryItem.itemQuantity = 1;
inventoryList.Add(inventoryItem);

//DebugPrintInventoryList(inventoryList);
}

/// <summary>
/// Add item to position in the inventory
/// </summary>
private void AddItemAtPosition(List<InventoryItem> inventoryList, int itemCode, int position)
{
InventoryItem inventoryItem = new InventoryItem();

int quantity = inventoryList[position].itemQuantity + 1;
inventoryItem.itemQuantity = quantity;
inventoryItem.itemCode = itemCode;
inventoryList[position] = inventoryItem;
//DebugPrintInventoryList(inventoryList);
}

/// <summary>
/// Find if an itemCode is already in the inventory. Returns the item position
/// in the inventory list, or -1 if the item is not in the inventory
/// </summary>

public int FindItemInInventory(InventoryLocation inventoryLocation, int itemCode)
{
List<InventoryItem> inventoryList = inventoryLists[(int)inventoryLocation];
  
for (int i = 0; i < inventoryList.Count; i++)
{
if (inventoryList[i].itemCode == itemCode)
{
return i;
}
}

return -1;
}
////////////////////////////////

// Debug Methods to print the item in console
private void DebugPrintInventoryList(List<InventoryItem> inventoryList)
{
foreach (InventoryItem inventoryItem in inventoryList)
{
Debug.Log("Item Description:" + InventoryManager.Instance.GetItemDetails(inventoryItem.itemCode).itemDescription + " Item Quantity: " + inventoryItem.itemQuantity);
}
Debug.Log("******************************************************************************");
}

}
```

### Update our InventoryManager.cs Class
-> add a canBePickUp check, and if yes it call InventoryManager.Instance.AddItem() and destroy the GO
``` c# [ItemPickup.cs]
using UnityEngine;

public class ItemPickUp : MonoBehaviour
{
private void OnTriggerEnter2D(Collider2D collision)
{
Item item = collision.GetComponent<Item>();
  
if (item != null)
{
// if we encounter an item, retrieve the item details
// InventoryManager is a Singleton, so we can access its {Instance} created in the scene to access the methods GetItemDetail()
ItemDetails itemDetails = InventoryManager.Instance.GetItemDetails(item.ItemCode);

// if item can be picked up
if (itemDetails.canBePickedUp == true)
{
// Add item to inventory
InventoryManager.Instance.AddItem(InventoryLocation.player, item, collision.gameObject);

}
// // print returned item detail
// Debug.Log(itemDetails.itemDescription);
}
}
}
```

and the pickup system is now active ! 

## Player Inventory Bar UI[6.8]
Lets design some gui first
goals :
-> draw the inventory bar at the bottom of the screen
-> when bottom bound of the map, move the bar to top for clear visibility

### Create the Inventory Bar
-> Main Canvas Set up
1. in Persistent Scene, Create>Game Object named "UI", underneath as a child Create>UI>Canvas named "MainGameUICanvas"
-> Enable the [Pixel Perfect Mode] of the canvas
-> Change its [UI Scale Mode] from " Constant Pixel Size" to "Scale With Screen Size" to stretch
-> Its [Reference Resolution] to X:480 Y:270
-> its [ Screen Match Mode] to Expand
-> [Reference Pixels Per Unit] 16px


-> Canvas Group / Components

1. Under MainGameUICanvas, create an empty GO named "UICanvasGroup"
-> add a [Canvas Group] component
-> and and [Aspect Ratio Filter] set to "Fit in Parent" and aspect ratio to 1.777778  (16/9)
2. Under UICanvasGroup, create an other empty GO named "UIInventoryBar" , 
->add an [Image] component to it with desired sprite (InventoryBar here)
->disable [Cull Transparent Mesh]
->and click to [Set Native Size] to resize the image correctly
3. Then Position the bar within the rect transform of the canvas (game view)
-> shift + alt + bottom center of Anchor
-> increase the pos Y to 2.5 to up a bit from screen
![[Pasted image 20221205145241.png]]
4. as

### Create the custom UIInventoryBar.cs
-> moves the bar in fonction of the player position and the map bouding
-> should update the player class to always retrieve its current position accessible by any script
1. Player.cs class, PlayerPosition on viewport method (from camera, convert position to viewport)
``` c# 
// [... Player.cs]
private Camera mainCamera; // new camera var

protected override void Awake()
{
base.Awake();

rigidBody2D = GetComponent<Rigidbody2D>();

// get reference to main camera
mainCamera = Camera.main;
}

public Vector3 GetPlayerViewportPosition()
{
// Vector3 viewport position for player ((0,0) viewport bottom left, (1,1) viewport top right)
return mainCamera.WorldToViewportPoint(transform.position);
}
```
2. Create the UIInventoryBar.cs class
-> call the methods just created on Player.cs class to get the play viewport position
-> if the position has reached the bottom of the map, change to position of the UI Inventory bar to top. Else, change the position to its initial state

``` c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class UIInventoryBar : MonoBehaviour
{

    private RectTransform rectTransform;

    private bool _isInventoryBarPositionBottom = true;

    public bool IsInventoryBarPositionBottom { get => _isInventoryBarPositionBottom; set => _isInventoryBarPositionBottom = value; }

    private void Awake()
    {
        rectTransform = GetComponent<RectTransform>();
    }

	private void Update()
	{
		// Switch inventory bar position depending on player position
		SwitchInventoryBarPosition();
	}

    private void SwitchInventoryBarPosition()
    {
        Vector3 playerViewportPosition = Player.Instance.GetPlayerViewportPosition();

        if (playerViewportPosition.y > 0.3f && IsInventoryBarPositionBottom == false)
        {
            // transform.position = new Vector3(transform.position.x, 7.5f, 0f); // this was changed to control the recttransform see below
            rectTransform.pivot = new Vector2(0.5f, 0f);
            rectTransform.anchorMin = new Vector2(0.5f, 0f);
            rectTransform.anchorMax = new Vector2(0.5f, 0f);
            rectTransform.anchoredPosition = new Vector2(0f, 2.5f);

            IsInventoryBarPositionBottom = true;
        }
        else if (playerViewportPosition.y <= 0.3f && IsInventoryBarPositionBottom == true)
        {
            //transform.position = new Vector3(transform.position.x, mainCamera.pixelHeight - 120f, 0f);// this was changed to control the recttransform see below
            rectTransform.pivot = new Vector2(0.5f, 1f);
            rectTransform.anchorMin = new Vector2(0.5f, 1f);
            rectTransform.anchorMax = new Vector2(0.5f, 1f);
            rectTransform.anchoredPosition = new Vector2(0f, -2.5f);

            IsInventoryBarPositionBottom = false;
        }
    }

}
```
3. then apply our UIInventoryBar.cs script as component to our UIInventoryBar GO to have our moving GUI
![[Pasted image 20221205152221.png]]

## Add Collected Items To Inventory Bar[6.9]

goals :
-> display our current inventory and update picked up item on the GUI bottom bar with sprite and qty
-> Create a series of slots positions over the inventory bar graphics, and in code manipulate image & qty displayed

concept :
-> to display a canva out of a scene, place your cursos in the Scene View (not hierarchy) and press F while having selected the desired canvas>related image
![[Pasted image 20221205154122.png]]

### Creatings GO UIInventorySlot (contain highlight and text)
-> prefabs for all others slots 

1. On the UIInventoryBar, create New Empty GO named "UIInventorySlot"
-> set its Anchor to ALT + middle left 
-> add an [Image] component with a transparant sprite, x16, enable [Preserve Aspect] and click on [Set Native Size]
-> add a [Canvas Group] Component
-> Set its [Pos X] to 0 (rect transform)

##### Creatings Inventory Bar Highlight & Text 
-> create childs under the UIInventorySlot previously created

1.  Create an empty GO child under UInventorySlot, named "*InventoryHighlight*" (for later dev to red highlight the item selected on click) 
-> add a [Image] component with the InventoryHighlight sprite (red circle)
-> Enable the [Preserve Aspect] of the Image compo
-> and click on [Set Native Size]
-> change the alpha of the higlight to 0 to make it invisible (to activate by code)

2. Create a new TextMeshPro Text child under UInventorySlot (import tmp essentials)
-> we'll create a font asset used by textmesh pro for our game named "Peepo"
(Window>TextMeshPro>FontAssetCreator, select the font you want, and generate a font atlas. then save the font !)
-> Now we can set our TMP text typo to Peepo !
-> Font style => Bold
-> Font Size => 4
=> Vertex color => Black
=> In the rectTranform => Anchor point alignement to Shift + Alt +Top Left
=> Set its position to Pos X : -1 PosY: 2 PosZ: 0
=> Width : 16 Height : 16
=> Delete Text (will be set by code)

3. Add then drap the UIInventoryBar from hierarchy to your prefabs to dup it for other slots ! 

### Create 12 GO slots of inventory 

1. dupliquate the UIInventory renamed  UIInventory0, and place them in the bar 
![[Pasted image 20221205161636.png]]

### Create the UIInventorySlot.cs script to add at Slots as component
-> contain data about each slots and set the sprite and qty field
1. Create some field in the UIInventorySlot.cs
``` c#
using TMPro;

using UnityEngine;

using UnityEngine.EventSystems;

using UnityEngine.UI;
public class UIInventorySlot : MonoBehaviour
{

// correspond to GO
public Image inventorySlotHighlight;
public Image inventorySlotImage;
public TextMeshProUGUI textMeshProUGUI;

[HideInInspector] public ItemDetails itemDetails;
[HideInInspector] public int itemQuantity;
}
```
2. Set corresponding component in the unity inspector

### Modify the UIInventoryBar.cs 
-> Populate the inventory slot with the inventory the player is actually carrying 
``` c#
// [...]UIInventoryBar.cs 

[SerializeField] private Sprite blank16x16sprite = null;
[SerializeField] private UIInventorySlot[] inventorySlot = null;
```
-> then respond to pickup event 
``` c# 
//[...] UIInventoryBar.cs

// subscribe/unsubscribe to update inventory event

private void OnDisable()
{
EventHandler.InventoryUpdatedEvent -= InventoryUpdated;
}

private void OnEnable()
{
EventHandler.InventoryUpdatedEvent += InventoryUpdated;
}

    private void ClearInventorySlots()
    {
        if (inventorySlot.Length > 0)
        {
            // loop through inventory slots and update with blank sprite
            for (int i = 0; i < inventorySlot.Length; i++)

            {
                inventorySlot[i].inventorySlotImage.sprite = blank16x16sprite;
                inventorySlot[i].textMeshProUGUI.text = "";
                inventorySlot[i].itemDetails = null;
                inventorySlot[i].itemQuantity = 0;
                %%SetHighlightedInventorySlots(i);%%
            }
        }
    }

    private void InventoryUpdated(InventoryLocation inventoryLocation, List<InventoryItem> inventoryList)
    {
        if (inventoryLocation == InventoryLocation.player)
        {
            ClearInventorySlots();

            if (inventorySlot.Length > 0 && inventoryList.Count > 0)
            {
                // loop through inventory slots and update with corresponding inventory list item
                for (int i = 0; i < inventorySlot.Length; i++)
                {
                    if (i < inventoryList.Count)
                    {
                        int itemCode = inventoryList[i].itemCode;

                        // ItemDetails itemDetails = InventoryManager.Instance.itemList.itemDetails.Find(x => x.itemCode == itemCode);
                        ItemDetails itemDetails = InventoryManager.Instance.GetItemDetails(itemCode);

                        if (itemDetails != null)
                        {
                            // add images and details to inventory item slot
                            inventorySlot[i].inventorySlotImage.sprite = itemDetails.itemSprite;
                            inventorySlot[i].textMeshProUGUI.text = inventoryList[i].itemQuantity.ToString();
                            inventorySlot[i].itemDetails = itemDetails;
                            inventorySlot[i].itemQuantity = inventoryList[i].itemQuantity;
                            %%SetHighlightedInventorySlots(i);%%

                        }
                    }
                    else
                    {
                        break;
                    }
                }
            }
        }
    }

```

### Day 5 | 6/12 (5h)
## Drop Items From Inventory Bar[6.10]

goals :
-> Drop items from inventory bar to the scene
-> When dropping and item to the scene, we'll create a GO parented with the Items GO group

concept :
-> Ray cast with the UIInventorySlot to detect where the mouse is and so the click

(Make sur that in [Image] component the [Raycast Target] is enable, but not in for the InventoryHighlight and the textmeshpro ! (extra settings))

### Add a new tag 
-> identify the GO group (Items, which contain all items of the scene) by a tag to reference it by code [unity interface], "ItemsParentTransform", and also add the tag to our Tags.cs 
1. asd
``` c#
[Tags.cs]
public const string ItemsParentTransform = "ItemsParentTransform";
```
2. asd

### Pimp the Player.cs script
-> When dragging item from inventory bar, we dont want the player to be able to move
1. Update the Update() methods to run only "if (!PlayerInputIsDisabled)"
``` c#
[Player.cs]

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
EventHandler.CallMovementEvent([...]);
}
#endregion Player Input
}
```
2. then add a couple of functions :
``` c#
[...]Player.cs

private void ResetMouvement()
{
	// Reset Mouvement
	xInput = 0f;
	yInput = 0f;
	isRunning = false;
	isWalking = false;
	isIdle = true;
}

public void DisablePlayerInputAndResetMouvement()
{
	DisablePlayerInput();
	ResetMouvement();
	
	// Send Event to any listeners for player movement input with changed parameters
	EventHandler.CallMovementEvent(xInput, yInput, isWalking, isRunning, isIdle, isCarrying, toolEffect,
	isUsingToolRight, isUsingToolLeft, isUsingToolUp, isUsingToolDown,
	isLiftingToolRight, isLiftingToolLeft, isLiftingToolUp, isLiftingToolDown,
	isPickingRight, isPickingLeft, isPickingUp, isPickingDown,
	isSwingingToolRight, isSwingingToolLeft, isSwingingToolUp, isSwingingToolDown,
	false, false, false, false);
}

public void DisablePlayerInput()
{
	PlayerInputIsDisabled = true;
}

public void EnablePlayerInput()
{
	PlayerInputIsDisabled = false;
}

```
3. asd

### Create a Prefab Reference for item drop (new items created)
-> d
1. add a GO reference to UIInventoryBar.cs, populate it within the inventorySlot as we dragg item out
``` c#
[...] UIInventoryBar.cs
public GameObject inventoryBarDraggedItem;
```

### Create the  dragged item Prefab GO
-> GO Created when item dragged out of Inventory Bar

1. Create a new empty GO under UIInventoryBar in the Hierarchy, and name it "InventoryDraggedItem"
-> Set its [width] and [height] to 16
-> Add a [Canvas] Component with :
-> [Pixel Perfect] : On
-> [Override Sorting] : Enable (Ensure the dragged item appear before the other items in the bar)
-> [Sort Order] : 2
-> and Add a [Canvas Renderer] Component
![[Pasted image 20221206233026.png]]
-> then create a child "Item" under InventoryDraggedItem :
-> set its [height] and [width] to 16
-> Add an [Image] component that will be set by our dragged item

2. Create a prefab for the InventoryDraggedItem and delete it from the scene ! 
-> in the prefab folder, clic on the InventoryDraggedItem, and on its [Canvas] component, set its [Render Mode] to : Screen Space - Overlay and enable its [Pixel Perfect]

3. then drag the prefab InventoryDraggedItem to the [UIInventoryBar] script component of the UIInventoryBar in the hierarchy, to be referenced when item dragged out
![[Pasted image 20221206233631.png]]

### Add the logic to UIInventorySlot.cs
-> Play with the unity event system 

1. add a new namespace and few variables
``` c#
[...] UIInventorySlot
using UnityEngine.EventSystems;

public class UIInventorySlot : MonoBehaviour
{

	private Camera mainCamera; // calling a methods named "screenToWorldPoint" static methods on main camera which convert from screen point to world point vector
	private Transform parentItem; // set to the parent GO in the scene that we tag
	public GameObject draggedItem; // for GO created when dragged

	[SerializeField] private UIInventoryBar inventoryBar = null; // Inventory bar referenced from the unity inspector
	[SerializeField] private GameObject itemPrefab = null;
```
2. initiate var with a Start() method
``` c#
[...]UIInventorySlot.cs
private void Start()
{
mainCamera = Camera.main;

parentItem = GameObject.FindGameObjectWithTag(Tags.ItemsParentTransform).transform;

}
```
3. Create the events methods 
-> the Unity event system works in conjonction with interface for detecting drag and drop and click. Those interface are call "IBeginDragHandler", "IDragHandler", "IEndDragHandler"
``` c#
[...]UIInventorySlot.cs
public void OnBeginDrag(PointerEventData eventData)
{
if (itemDetails != null)
{
// Disable keyboard input
Player.Instance.DisablePlayerInputAndResetMovement();
  
// Instatiate gameobject as dragged item
draggedItem = Instantiate(inventoryBar.inventoryBarDraggedItem, inventoryBar.transform);

// Get image for dragged item
Image draggedItemImage = draggedItem.GetComponentInChildren<Image>();
draggedItemImage.sprite = inventorySlotImage.sprite;

}
}

public void OnDrag(PointerEventData eventData)
{
// move game object as dragged item
if (draggedItem != null)
{
draggedItem.transform.position = Input.mousePosition;
}
}

public void OnEndDrag(PointerEventData eventData)
{
	// Destroy game object as dragged item
	if (draggedItem != null)
	{
	Destroy(draggedItem);
	  
	// If drag ends over inventory bar, get item drag is over and swap them
	
		if (eventData.pointerCurrentRaycast.gameObject != null && eventData.pointerCurrentRaycast.gameObject.GetComponent<UIInventorySlot>() != null)
		
		{
		 // to fill later
		}
		// else attempt to drop the item if it can be dropped
	else
		{
		if (itemDetails.canBeDropped)
			{
			DropSelectedItemAtMousePosition();
			}
		}
		
		// Enable player input
		Player.Instance.EnablePlayerInput();
		}
}

/// <summary>
/// Drops the item (if selected) at the current mouse position. Called by the DropItem event.
/// </summary>
private void DropSelectedItemAtMousePosition()
{
	if (itemDetails != null && isSelected)
	{

			Vector3 worldPosition = mainCamera.ScreenToWorldPoint(new Vector3(Input.mousePosition.x, Input.mousePosition.y, -mainCamera.transform.position.z));
			
			// Create item from prefab at mouse position
			GameObject itemGameObject = Instantiate(itemPrefab, new Vector3(worldPosition.x, worldPosition.y - Settings.gridCellSize / 2f, worldPosition.z), Quaternion.identity, parentItem);
			Item item = itemGameObject.GetComponent<Item>();
			item.ItemCode = itemDetails.itemCode;
			
			// Remove item from players inventory	
			InventoryManager.Instance.RemoveItem(InventoryLocation.player, item.ItemCode);
		
	}
}
```
4. add the RemoveItem() and other methods to InventoryManager.cs
``` c#
[...] InventoryManager.cs

/// <summary>
/// Remove an item from the inventory, and create a game object at the position it was dropped
/// </summary>
public void RemoveItem(InventoryLocation inventoryLocation, int itemCode)
{
	List<InventoryItem> inventoryList = inventoryLists[(int)inventoryLocation];
	
	// Check if inventory already contains the item
	int itemPosition = FindItemInInventory(inventoryLocation, itemCode);
	
	if (itemPosition != -1)
	{
		RemoveItemAtPosition(inventoryList, itemCode, itemPosition);
	}
	
	// Send event that inventory has been updated
	EventHandler.CallInventoryUpdatedEvent(inventoryLocation, inventoryLists[(int)inventoryLocation]);

}

private void RemoveItemAtPosition(List<InventoryItem> inventoryList, int itemCode, int position)
{
	InventoryItem inventoryItem = new InventoryItem();
	
	int quantity = inventoryList[position].itemQuantity - 1;
	
	if (quantity > 0)
	{
		inventoryItem.itemQuantity = quantity;
		inventoryItem.itemCode = itemCode;
		inventoryList[position] = inventoryItem;
	}
	else
	{
		inventoryList.RemoveAt(position);
	}
}
```
5. add the corresponding field in the UIInventorySlot Prefab !

## Reorder Items in Inventory Bar [6.11]

goals :
-> Allow player to swap item in the inventory bar by dragging them

### Update the UIInventorySlot.cs 
-> add field to identify slot number this ui slot is

1. to update in the inspector
``` c#
[...] UIInventorySlot.cs
[SerializeField] private int slotNumber = 0;
```
2. Update the OnEndDragMethod()
``` c#
public void OnEndDrag(PointerEventData eventData)
{
// Destroy game object as dragged item
if (draggedItem != null)
{
Destroy(draggedItem);

// If drag ends over inventory bar, get item drag is over and swap them
if (eventData.pointerCurrentRaycast.gameObject != null && eventData.pointerCurrentRaycast.gameObject.GetComponent<UIInventorySlot>() != null)
{
// get the slot number where the drag ended
int toSlotNumber = eventData.pointerCurrentRaycast.gameObject.GetComponent<UIInventorySlot>().slotNumber;

// Swap inventory items in inventory list

InventoryManager.Instance.SwapInventoryItems(InventoryLocation.player, slotNumber, toSlotNumber);

}
// else attempt to drop the item if it can be dropped
else
{
if (itemDetails.canBeDropped)
{
DropSelectedItemAtMousePosition();
}
}

// Enable player input
Player.Instance.EnablePlayerInput();
}
}
```

### Update the InventoryManager.cs 

1. add the SwapInventoryItems()
``` c#
[...] InventoryManager.cs
///<summary>
///Swap item at fromItem index with item at toItem index in inventoryLocation inventory list
///</summary>

public void SwapInventoryItems(InventoryLocation inventoryLocation, int fromItem, int toItem)
{
	// if fromItem index and toItemIndex are within the bounds of the list, not the same, and greater than or equal to zero
	if (fromItem < inventoryLists[(int)inventoryLocation].Count && toItem < inventoryLists[(int)inventoryLocation].Count
	&& fromItem != toItem && fromItem >= 0 && toItem >= 0)
	{
		InventoryItem fromInventoryItem = inventoryLists[(int)inventoryLocation][fromItem];
		InventoryItem toInventoryItem = inventoryLists[(int)inventoryLocation][toItem];
		
		inventoryLists[(int)inventoryLocation][toItem] = fromInventoryItem;
		inventoryLists[(int)inventoryLocation][fromItem] = toInventoryItem;
		
		// Send event that inventory has been updated
		EventHandler.CallInventoryUpdatedEvent(inventoryLocation, inventoryLists[(int)inventoryLocation]);
	}
}
```

2. Then assign each InventorySlot in the Hierarchy to its corresponding slot number !  
![[Pasted image 20221207010928.png]]
## Item Description Pop ups[6.12]

goals :
-> When hover on the inventory bar, a text box display with description from the item details
->

concept :
-> Create a Prefab GO for the Pop up text box, and we're gonna populate it in script with the description from item detail

### Create the GO
1. under UICanvasGroup, create the "InventoryTextBox" empty GO ("f" in scene view to go to view)
-> add a [Content Size Fitter] component to it, which automaticaly rezise the GO based on its content 
-> set the [Vertical Fit] to : Preferred Size, which set the vertical size of the inventory text box based on the elements its contains
-> add a [Vertical Layout Group] Component, which layout the child GO of the inventory text box verticaly
	-> set its Padding>[Spacing] in to -4
	-> [Control Child Size] enable Width & Height
	-> [Child Force Expand] Enable Width & Height

2. Add a child UI image GO,
-> right click Inventory Text Box > UI > Image, named "TextBoxImage1"
-> set its [Source Image] to (what u want) "text box" here

Below this TextBoxImage1 GO, we're going to have some text game object and we want them layout sorted vertically aswell, so
-> add a [Vertical Layout Group] component
	Left : 10
	Right : 10
	Top : 5
	Bottom: 5
	Spacing : 1
	ControleChild Size : Enable Width & Height

3. Add text GO
-> Right click on TextBoxImage1
-> UI>Text - TextMeshPro named "Text1"
	-> set the [font size] to 10
	->	[Vertex Color] => black
4. duplique and name it Text2
-> set its [font size] to 8
-> [Vertex Color] => E00000 (red)
5. Dup text 3
-> [Vertex Color] => black
6. Dup TextBox Image1 and name it TextBoxImage2
![[Pasted image 20221207014508.png]]

then delete all Predefine text, drag the InventoryTextBox GO to Prefabs and delete it from the scene !

### Prefab GO Access Text Script
-> Create a new class "UIInventoryTextBox.cs" to attach to text box to easily access the each texts fields created
1. Define 6 variables to define in the inspector for each text, and a methods to set those texts once defined with item details on hover
``` c#
﻿using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class UIInventoryTextBox : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI textMeshTop1 = null;
    [SerializeField] private TextMeshProUGUI textMeshTop2 = null;
    [SerializeField] private TextMeshProUGUI textMeshTop3 = null;
    [SerializeField] private TextMeshProUGUI textMeshBottom1 = null;
    [SerializeField] private TextMeshProUGUI textMeshBottom2 = null;
    [SerializeField] private TextMeshProUGUI textMeshBottom3 = null;



    // Set text values
    public void SetTextboxText(string textTop1, string textTop2, string textTop3, string textBottom1, string textBottom2, string textBottom3)
    {
        textMeshTop1.text = textTop1;
        textMeshTop2.text = textTop2;
        textMeshTop3.text = textTop3;
        textMeshBottom1.text = textBottom1;
        textMeshBottom2.text = textBottom2;
        textMeshBottom3.text = textBottom3;
    }

}
```
2. then attach the script to the prefab and populate the variable in the inspector with appropriate text GO

### Update Settings.cs
-> Add Strings Constants used for description box for different type of tools
``` c#
[...] Settings.cs

//Tools
public const string HoeingTool = "Hoe";
public const string ChoppingTool = "Axe";
public const string BreakingTool = "Pickaxe";
public const string ReapingTool = "Scythe";
public const string WateringTool = "Watering Can";
public const string CollectingTool = "Basket";
```

### Update InventoryManager.cs
-> New methods to get item description with item type passed in
``` c#
[...] InventoryManager.cs

/// <summary>
/// Get the item type description for an item type - returns the item type description as a string for a given ItemType
/// </summary>

public string GetItemTypeDescription(ItemType itemType)
{
	string itemTypeDescription;
	switch (itemType)
	{
		case ItemType.Breaking_tool:
		itemTypeDescription = Settings.BreakingTool;
		break;
		
		case ItemType.Chopping_tool:
		itemTypeDescription = Settings.ChoppingTool;
		break;
		
		case ItemType.Hoeing_tool:
		itemTypeDescription = Settings.HoeingTool;
		break;
		
		case ItemType.Reaping_tool:
		itemTypeDescription = Settings.ReapingTool;
		break;
		
		case ItemType.Watering_tool:
		itemTypeDescription = Settings.WateringTool;
		break;
		
		case ItemType.Collecting_tool:
		itemTypeDescription = Settings.CollectingTool;
		break;
		
		default:
		itemTypeDescription = itemType.ToString();
		break;
	}
	return itemTypeDescription;
}
```

### Update UIInventorySlot.cs
-> when creating our item description textbox GO, we going to parent it, underneath his parent canvas. 

``` c#
[...] UIInventorySlot.cs

private Canvas parentCanvas;

[SerializeField] private GameObject inventoryTextBoxPrefab = null;

private void Awake()
{
	parentCanvas = GetComponentInParent<Canvas>();
}

```

-> when the cursor enter and exit a GO and the interface, we need to do all these

``` c#
[...] UIInventorySlot.cs

public class UIInventorySlot : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler, IPointerEnterHandler, IPointerEnterExit
{
[...]
}

```

-> create the methods to pop up text
``` c#
[...] UIInventorySlot.cs

    public void OnPointerEnter(PointerEventData eventData)
    {
        // Populate text box with item details
        if (itemQuantity != 0)
        {
            // Instantiate inventory text box
            inventoryBar.inventoryTextBoxGameobject = Instantiate(inventoryTextBoxPrefab, transform.position, Quaternion.identity);
            inventoryBar.inventoryTextBoxGameobject.transform.SetParent(parentCanvas.transform, false);

            UIInventoryTextBox inventoryTextBox = inventoryBar.inventoryTextBoxGameobject.GetComponent<UIInventoryTextBox>();

            // Set item type description
            string itemTypeDescription = InventoryManager.Instance.GetItemTypeDescription(itemDetails.itemType);

            // Populate text box
            inventoryTextBox.SetTextboxText(itemDetails.itemDescription, itemTypeDescription, "", itemDetails.itemLongDescription, "", "");

            // Set text box position according to inventory bar position
            if (inventoryBar.IsInventoryBarPositionBottom)

            {
                inventoryBar.inventoryTextBoxGameobject.GetComponent<RectTransform>().pivot = new Vector2(0.5f, 0f);
                inventoryBar.inventoryTextBoxGameobject.transform.position = new Vector3(transform.position.x, transform.position.y + 50f, transform.position.z);
            }
            else
            {
                inventoryBar.inventoryTextBoxGameobject.GetComponent<RectTransform>().pivot = new Vector2(0.5f, 1f);
                inventoryBar.inventoryTextBoxGameobject.transform.position = new Vector3(transform.position.x, transform.position.y - 50f, transform.position.z);
            }
        }
    }

    public void OnPointerExit(PointerEventData eventData)
    {
        DestroyInventoryTextBox();
    }


```
-> update the OnEndDrag() to clean instance of popup text
``` c#
[...] UIInventorySlot.cs
// If drag ends over inventory bar, get item drag is over and swap them

public void OnEndDrag(PointerEventData eventData)
{
// Destroy game object as dragged item
if (draggedItem != null)
{
Destroy(draggedItem);

// If drag ends over inventory bar, get item drag is over and swap them
if (eventData.pointerCurrentRaycast.gameObject != null && eventData.pointerCurrentRaycast.gameObject.GetComponent<UIInventorySlot>() != null)
{
// get the slot number where the drag ended
int toSlotNumber = eventData.pointerCurrentRaycast.gameObject.GetComponent<UIInventorySlot>().slotNumber;

// Swap inventory items in inventory list
InventoryManager.Instance.SwapInventoryItems(InventoryLocation.player, slotNumber, toSlotNumber);

// Destroy inventory text box
DestroyInventoryTextBox();
[...]
}
```

Finly Add a GO Field to UIInventoryBar.cs
``` c#
[...] UIIventoryBar.cs
[HideInInspector] public GameObject inventoryTextBoxGameobject;
```

and assign textbox field in UIInventorySlot  prefab !

## Select Items in the Inventory Bar[6.14]

goals :
->
->

concept :
-> 

### Goals 1
->
1. asd
``` c#
asd
```
2. asd

### Goals 2
->
1. asd
``` c#
asd
```
2. asd

## Carry Item Animation Overrides[6.15]

goals :
->
->

concept :
-> 

### Goals 1
->
1. asd
``` c#
asd
```
2. asd

### Goals 2
->
1. asd
``` c#
asd
```
2. asd

## Title[6.16]

goals :
->
->

concept :
-> 

### Goals 1
->
1. asd
``` c#
asd
```
2. asd

### Goals 2
->
1. asd
``` c#
asd
```
2. asd

## Title[6.17]

goals :
->
->

concept :
-> 

### Goals 1
->
1. asd
``` c#
asd
```
2. asd

### Goals 2
->
1. asd
``` c#
asd
```
2. asd

## Title[6.18]

goals :
->
->

concept :
-> 

### Goals 1
->
1. asd
``` c#
asd
```
2. asd

### Goals 2
->
1. asd
``` c#
asd
```
2. asd