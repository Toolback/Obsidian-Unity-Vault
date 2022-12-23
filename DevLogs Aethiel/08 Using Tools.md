## Using the Hoe Animation [6.18]

goals :
-> allow the player to select the hoe and clic on a digeable area to active the hoe animation
->

concept :
-> 

### Add Items to Settings.cs
->
1. asd
``` c#
public static Vector2 cursorSize = Vector2.one;
public static float useToolAnimationPause = 0.25f;
public static float afterUseToolAnimationPause = 0.2f;
```
2. asd

### Change the GridCursor.cs
->
1. add a test case ItemType.Hoeing.Tool
![[Pasted image 20221209221607.png]]
2. add the is CursorValidForTool() to GridCursor.cs
``` c# 
// / <summary>
// / Sets the cursor as either valid or invalid for the tool for the target gridPropertyDetails. Returns true if valid or false if invalid
// / </summary>

private bool IsCursorValidForTool(GridPropertyDetails gridPropertyDetails, ItemDetails itemDetails)
{
// Switch on tool
switch (itemDetails.itemType)
{
case ItemType.Hoeing_tool:
if (gridPropertyDetails.isDiggable == true && gridPropertyDetails.daysSinceDug == -1)
{
#region Need to get any items at location so we can check if they are reapable
  
// Get world position for cursor
Vector3 cursorWorldPosition = new Vector3(GetWorldPositionForCursor().x + 0.5f, GetWorldPositionForCursor().y + 0.5f, 0f);
  
// Get list of items at cursor location
List<Item> itemList = new List<Item>();

HelperMethods.GetComponentsAtBoxLocation<Item>(out itemList, cursorWorldPosition, Settings.cursorSize, 0f);  

#endregion Need to get any items at location so we can check if they are reapable  

// Loop through items found to see if any are reapable type - we are not going to let the player dig where there are reapable scenary items
bool foundReapable = false;  

foreach (Item item in itemList)
{
if (InventoryManager.Instance.GetItemDetails(item.ItemCode).itemType == ItemType.Reapable_scenary)
{
foundReapable = true;
break;
}
}
  
if (foundReapable)
{
return false;
}
else
{
return true;
}
}
else
{
return false;
}

default:
return false;
}
}


public Vector3 GetWorldPositionForCursor()
{
	return grid.CellToWorld(GetGridPositionForCursor());
}
```

### Create the HelperMethods.cs
->
1. asd
``` c#
﻿using System.Collections.Generic;
using UnityEngine;

public static class HelperMethods
{   
/// <summary>
    /// Gets Components of type T at box with centre point and size and angle.  Returns true if at least one found and the found components are returned in the list
    /// </summary>
    public static bool GetComponentsAtBoxLocation<T>(out List<T> listComponentsAtBoxPosition, Vector2 point, Vector2 size, float angle)
    {
        bool found = false;
        List<T> componentList = new List<T>();

        Collider2D[] collider2DArray = Physics2D.OverlapBoxAll(point, size, angle);

        // Loop through all colliders to get an object of type T
        for (int i = 0; i < collider2DArray.Length; i++)
        {
            T tComponent = collider2DArray[i].gameObject.GetComponentInParent<T>();
            if (tComponent != null)
            {
                found = true;
                componentList.Add(tComponent);
            }
            else
            {
                tComponent = collider2DArray[i].gameObject.GetComponentInChildren<T>();
                if (tComponent != null)
                {
                    found = true;
                    componentList.Add(tComponent);
                }
            }
        }

        listComponentsAtBoxPosition = componentList;

        return found;
    }
}
```

### Update the Player.cs
->
1. to respond when player click to hoe with tool selected
``` c#
using System.Collections; // new namespace

private WaitForSeconds afterUseToolAnimationPause;

private WaitForSeconds useToolAnimationPause;

private bool playerToolUseDisabled = false;

// Update the Start method to populate above fields
private void Start()
{
gridCursor = FindObjectOfType<GridCursor>();
useToolAnimationPause = new WaitForSeconds(Settings.useToolAnimationPause);
afterUseToolAnimationPause = new WaitForSeconds(Settings.afterUseToolAnimationPause);
}

private void PlayerClickInput()
{
if (!playerToolUseDisabled)
{
if (Input.GetMouseButton(0))
{
//if (gridCursor.CursorIsEnabled || cursor.CursorIsEnabled)
if (gridCursor.CursorIsEnabled) {

// Get Cursor Grid Position
Vector3Int cursorGridPosition = gridCursor.GetGridPositionForCursor();

// Get Player Grid Position
Vector3Int playerGridPosition = gridCursor.GetGridPositionForPlayer();

ProcessPlayerClickInput(cursorGridPosition, playerGridPosition);
}
}
}
}

  

private Vector3Int GetPlayerClickDirection(Vector3Int cursorGridPosition, Vector3Int playerGridPosition)
{
if (cursorGridPosition.x > playerGridPosition.x)
{
return Vector3Int.right;
}
else if (cursorGridPosition.x < playerGridPosition.x)
{
return Vector3Int.left;
}
else if (cursorGridPosition.y > playerGridPosition.y)
{
return Vector3Int.up;
}
else
{
return Vector3Int.down;
}
}

// Rework ProcessPlayerClickInput()
// to add the hoe case
private void ProcessPlayerClickInput(Vector3Int cursorGridPosition, Vector3Int playerGridPosition)

{

ResetMovement();

  

Vector3Int playerDirection = GetPlayerClickDirection(cursorGridPosition, playerGridPosition);

  

// Get Grid property details at cursor position (the GridCursor validation routine ensures that grid property details are not null)

GridPropertyDetails gridPropertyDetails = GridPropertiesManager.Instance.GetGridPropertyDetails(cursorGridPosition.x, cursorGridPosition.y);

  

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

  

// case ItemType.Watering_tool:

// case ItemType.Breaking_tool:

// case ItemType.Chopping_tool:

case ItemType.Hoeing_tool:

// case ItemType.Reaping_tool:

// case ItemType.Collecting_tool:

ProcessPlayerClickInputTool(gridPropertyDetails, itemDetails, playerDirection);

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

    private void ProcessPlayerClickInputTool(GridPropertyDetails gridPropertyDetails, ItemDetails itemDetails, Vector3Int playerDirection)
    {
        // Switch on tool
        switch (itemDetails.itemType)
        {
            case ItemType.Hoeing_tool:
                if (gridCursor.CursorPositionIsValid)
                {
                    HoeGroundAtCursor(gridPropertyDetails, playerDirection);
                }
                break;
                
            default:
                break;
        }
    }

    private void HoeGroundAtCursor(GridPropertyDetails gridPropertyDetails, Vector3Int playerDirection)
    {
        // Trigger animation
        StartCoroutine(HoeGroundAtCursorRoutine(playerDirection, gridPropertyDetails));
    }

    private IEnumerator HoeGroundAtCursorRoutine(Vector3Int playerDirection, GridPropertyDetails gridPropertyDetails)
    {
        PlayerInputIsDisabled = true;
        playerToolUseDisabled = true;

        // Set tool animation to hoe in override animation
        toolCharacterAttribute.partVariantType = PartVariantType.hoe;
        characterAttributeCustomisationList.Clear();
        characterAttributeCustomisationList.Add(toolCharacterAttribute);
        animationOverrides.ApplyCharacterCustomisationParameters(characterAttributeCustomisationList);

        if (playerDirection == Vector3Int.right)
        {
            isUsingToolRight = true;
        }
        else if (playerDirection == Vector3Int.left)
        {
            isUsingToolLeft = true;
        }
        else if (playerDirection == Vector3Int.up)
        {
            isUsingToolUp = true;
        }
        else if (playerDirection == Vector3Int.down)
        {
            isUsingToolDown = true;
        }

        yield return useToolAnimationPause;

        // Set Grid property details for dug ground
        if (gridPropertyDetails.daysSinceDug == -1)
        {
            gridPropertyDetails.daysSinceDug = 0;
        }

        // Set grid property to dug
        GridPropertiesManager.Instance.SetGridPropertyDetails(gridPropertyDetails.gridX, gridPropertyDetails.gridY, gridPropertyDetails);




        // After animation pause
        yield return afterUseToolAnimationPause;

        PlayerInputIsDisabled = false;
        playerToolUseDisabled = false;
    }
```
2. asd


## Digging the Ground [7.2]

goals :
-> display dug ground when player clic to hoe the tile 
->

concept :
-> 
![[Pasted image 20221210134517.png]]
![[Pasted image 20221210134534.png]]
![[Pasted image 20221210134603.png]]
![[Pasted image 20221210134642.png]]
![[Pasted image 20221210134706.png]]
![[Pasted image 20221210134719.png]]
![[Pasted image 20221210134731.png]]
![[Pasted image 20221210134737.png]]
![[Pasted image 20221210134756.png]]
![[Pasted image 20221210134759.png]]
![[Pasted image 20221210134800.png]]
![[Pasted image 20221210134909.png]]
![[Pasted image 20221210134937.png]]
![[Pasted image 20221210134947.png]]
![[Pasted image 20221210135102.png]]
### Tags.cs
->
1. add new tags
``` c#
[...] Tags.cs
public const string GroundDecoration1 = "GroundDecoration1";

public const string GroundDecoration2 = "GroundDecoration2";
```
2. asd

### Works on the GridPropertiesManager.cs
-> 
1. asd
``` c#
using UnityEngine.Tilemaps;

private Grid grid; // from public to private
private Tilemap groundDecoration1;
private Tilemap groundDecoration2;
// selected dug ground tiles from the inspector
[SerializeField] private Tile[] dugGround = null;

private void ClearDisplayGroundDecoration()
{
	// remove ground decorations
	groundDecoration1.ClearAllTiles();
	groundDecoration2.ClearAllTiles();
}

private void ClearDisplayGridPropertyDetails()
{
	// TODO : clear crops call
	ClearDisplayGroundDecoration();
}

private void DisplayDugGround()
{
	// Dug
	if (GetGridPropertyDetails.daysSinceDug > -1)
	{
		ConnectDugGround(gridPropertyDetails);
	}
}



    private void ConnectDugGround(GridPropertyDetails gridPropertyDetails)
    {
        // Select tile based on surrounding dug tiles

        Tile dugTile0 = SetDugTile(gridPropertyDetails.gridX, gridPropertyDetails.gridY);
        groundDecoration1.SetTile(new Vector3Int(gridPropertyDetails.gridX, gridPropertyDetails.gridY, 0), dugTile0);

        // Set 4 tiles if dug surrounding current tile - up, down, left, right now that this central tile has been dug

        GridPropertyDetails adjacentGridPropertyDetails;

        adjacentGridPropertyDetails = GetGridPropertyDetails(gridPropertyDetails.gridX, gridPropertyDetails.gridY + 1);
        if (adjacentGridPropertyDetails != null && adjacentGridPropertyDetails.daysSinceDug > -1)
        {
            Tile dugTile1 = SetDugTile(gridPropertyDetails.gridX, gridPropertyDetails.gridY + 1);
            groundDecoration1.SetTile(new Vector3Int(gridPropertyDetails.gridX, gridPropertyDetails.gridY + 1, 0), dugTile1);
        }

        adjacentGridPropertyDetails = GetGridPropertyDetails(gridPropertyDetails.gridX, gridPropertyDetails.gridY - 1);
        if (adjacentGridPropertyDetails != null && adjacentGridPropertyDetails.daysSinceDug > -1)
        {
            Tile dugTile2 = SetDugTile(gridPropertyDetails.gridX, gridPropertyDetails.gridY - 1);
            groundDecoration1.SetTile(new Vector3Int(gridPropertyDetails.gridX, gridPropertyDetails.gridY - 1, 0), dugTile2);
        }
        adjacentGridPropertyDetails = GetGridPropertyDetails(gridPropertyDetails.gridX - 1, gridPropertyDetails.gridY);
        if (adjacentGridPropertyDetails != null && adjacentGridPropertyDetails.daysSinceDug > -1)
        {
            Tile dugTile3 = SetDugTile(gridPropertyDetails.gridX - 1, gridPropertyDetails.gridY);
            groundDecoration1.SetTile(new Vector3Int(gridPropertyDetails.gridX - 1, gridPropertyDetails.gridY, 0), dugTile3);
        }

        adjacentGridPropertyDetails = GetGridPropertyDetails(gridPropertyDetails.gridX + 1, gridPropertyDetails.gridY);
        if (adjacentGridPropertyDetails != null && adjacentGridPropertyDetails.daysSinceDug > -1)
        {
            Tile dugTile4 = SetDugTile(gridPropertyDetails.gridX + 1, gridPropertyDetails.gridY);
            groundDecoration1.SetTile(new Vector3Int(gridPropertyDetails.gridX + 1, gridPropertyDetails.gridY, 0), dugTile4);
        }
    }




    private Tile SetDugTile(int xGrid, int yGrid)
    {
        //Get whether surrounding tiles (up,down,left, and right) are dug or not

        bool upDug = IsGridSquareDug(xGrid, yGrid + 1);
        bool downDug = IsGridSquareDug(xGrid, yGrid - 1);
        bool leftDug = IsGridSquareDug(xGrid - 1, yGrid);
        bool rightDug = IsGridSquareDug(xGrid + 1, yGrid);

        #region Set appropriate tile based on whether surrounding tiles are dug or not

        if (!upDug && !downDug && !rightDug && !leftDug)
        {
            return dugGround[0];
        }
        else if (!upDug && downDug && rightDug && !leftDug)
        {
            return dugGround[1];
        }
        else if (!upDug && downDug && rightDug && leftDug)
        {
            return dugGround[2];
        }
        else if (!upDug && downDug && !rightDug && leftDug)
        {
            return dugGround[3];
        }
        else if (!upDug && downDug && !rightDug && !leftDug)
        {
            return dugGround[4];
        }
        else if (upDug && downDug && rightDug && !leftDug)
        {
            return dugGround[5];
        }
        else if (upDug && downDug && rightDug && leftDug)
        {
            return dugGround[6];
        }
        else if (upDug && downDug && !rightDug && leftDug)
        {
            return dugGround[7];
        }
        else if (upDug && downDug && !rightDug && !leftDug)
        {
            return dugGround[8];
        }
        else if (upDug && !downDug && rightDug && !leftDug)
        {
            return dugGround[9];
        }
        else if (upDug && !downDug && rightDug && leftDug)
        {
            return dugGround[10];
        }
        else if (upDug && !downDug && !rightDug && leftDug)
        {
            return dugGround[11];
        }
        else if (upDug && !downDug && !rightDug && !leftDug)
        {
            return dugGround[12];
        }
        else if (!upDug && !downDug && rightDug && !leftDug)
        {
            return dugGround[13];
        }
        else if (!upDug && !downDug && rightDug && leftDug)
        {
            return dugGround[14];
        }
        else if (!upDug && !downDug && !rightDug && leftDug)
        {
            return dugGround[15];
        }

        return null;

        #endregion Set appropriate tile based on whether surrounding tiles are dug or not
    }



    private bool IsGridSquareDug(int xGrid, int yGrid)
    {
        GridPropertyDetails gridPropertyDetails = GetGridPropertyDetails(xGrid, yGrid);

        if (gridPropertyDetails == null)
        {
            return false;
        }
        else if (gridPropertyDetails.daysSinceDug > -1)
        {
            return true;
        }
        else
        {
            return false;
        }
    }


    private void DisplayGridPropertyDetails()
    {
        // Loop through all grid items
        foreach (KeyValuePair<string, GridPropertyDetails> item in gridPropertyDictionary)
        {
            GridPropertyDetails gridPropertyDetails = item.Value;

            DisplayDugGround(gridPropertyDetails);

  //          DisplayWateredGround(gridPropertyDetails);

//            DisplayPlantedCrop(gridPropertyDetails);
        }
    }



/////// update the : 
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

            // If grid properties exist
            if (gridPropertyDictionary.Count > 0)
            {
                // grid property details found for the current scene destroy existing ground decoration
                ClearDisplayGridPropertyDetails();
                // Instantiate grid property details for current scene
                DisplayGridPropertyDetails();
            }

        }
    }


/////// update the : 
    private void AfterSceneLoaded()
    {
        // Get Grid
        grid = GameObject.FindObjectOfType<Grid>();

        // Get tilemaps
        groundDecoration1 = GameObject.FindGameObjectWithTag(Tags.GroundDecoration1).GetComponent<Tilemap>();
        groundDecoration2 = GameObject.FindGameObjectWithTag(Tags.GroundDecoration2).GetComponent<Tilemap>();

    }

```


### Update the Player.cs
->
1. asd
``` c#
  private IEnumerator HoeGroundAtCursorRoutine(Vector3Int playerDirection, GridPropertyDetails gridPropertyDetails)
    {
        PlayerInputIsDisabled = true;
        playerToolUseDisabled = true;

        // Set tool animation to hoe in override animation
        toolCharacterAttribute.partVariantType = PartVariantType.hoe;
        characterAttributeCustomisationList.Clear();
        characterAttributeCustomisationList.Add(toolCharacterAttribute);
        animationOverrides.ApplyCharacterCustomisationParameters(characterAttributeCustomisationList);

        if (playerDirection == Vector3Int.right)
        {
            isUsingToolRight = true;
        }
        else if (playerDirection == Vector3Int.left)
        {
            isUsingToolLeft = true;
        }
        else if (playerDirection == Vector3Int.up)
        {
            isUsingToolUp = true;
        }
        else if (playerDirection == Vector3Int.down)
        {
            isUsingToolDown = true;
        }

        yield return useToolAnimationPause;

        // Set Grid property details for dug ground
        if (gridPropertyDetails.daysSinceDug == -1)
        {
            gridPropertyDetails.daysSinceDug = 0;
        }
		
        // Set grid property to dug
        GridPropertiesManager.Instance.SetGridPropertyDetails(gridPropertyDetails.gridX, gridPropertyDetails.gridY, gridPropertyDetails);

// NEW HERE !
        // Display dug grid tiles
        GridPropertiesManager.Instance.DisplayDugGround(gridPropertyDetails);


        // After animation pause
        yield return afterUseToolAnimationPause;

        PlayerInputIsDisabled = false;
        playerToolUseDisabled = false;
    }

```

### Create the Dug Ground Tiles 
-> used in the Grid Properties Manager

1. Create new Ground Palette, and drop every Tile in Dug Ground field in the GridPropetiesManager
![[Pasted image 20221210151241.png]]

2. add new tags
-> Tags>AddTags>+ " GroundDecoration1" & "GroundDecoration2"
3. tags our tilemaps in different scenes in the tilemap grid
## Using the watering can animation [7.3]

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

## Watering dug ground [7.4]

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

## Implemant Non Grid-Based Cursor [7.5]

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

## Using the Scythe [7.6]

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

## Only [6.18]

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