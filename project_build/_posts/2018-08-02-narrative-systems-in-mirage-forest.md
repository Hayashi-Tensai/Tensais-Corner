---
layout: post
title:  "Mirage Forest - Narrative Systems"
permalink: blog/2018/08/02/narrative-systems-in-mirage-forest
date:   2018-08-02 10:46:00 +0800
categories: game-development mirage-forest
---

Full script can be viewed at:  
[https://github.com/kwlim94/Mirage-Forest/blob/master/Mirage%20Forest/Assets/Scripts/Databases/NarrativeDatabaseScript.cs](https://github.com/kwlim94/Mirage-Forest/blob/master/Mirage%20Forest/Assets/Scripts/Databases/NarrativeDatabaseScript.cs)

[https://github.com/kwlim94/Mirage-Forest/blob/master/Mirage%20Forest/Assets/Scripts/Global%20Controllers/NarrativeControlScript.cs](https://github.com/kwlim94/Mirage-Forest/blob/master/Mirage%20Forest/Assets/Scripts/Global%20Controllers/NarrativeControlScript.cs)  

This system composed of two scripts the Narrative Database Script and the Narrative Control Script.

Narrative Database Script:  
Overview of the script

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public class Dialogue
{
    public string name;
    public int chracterID;
    public string speech;
    public float duration;
    public float wantedAngle;
    public CharacterAnimation characterAnimation;
}

[System.Serializable]
public class NarrativeDatabase
{
    public string name;
    public int IdNumber;
    public string Description; //HT For refrence usage only
    public List<Dialogue> DialogueList;
}

public class NarrativeDatabaseScript : MonoBehaviour
{
    public static NarrativeDatabaseScript Instance {get; set;}

    void Awake()
    {
        if(Instance!= null && Instance != this)
                Destroy(gameObject);
        else
                Instance = this;
    }

    public List<NarrativeDatabase> NarrativeDatabaseList;
}
```

This script stores all the data required for the Narrative. First, there is a <Dialogue> class that stores the required data for each dialogue.

```csharp
[System.Serializable]
public class Dialogue
{
    public string name;
    public int chracterID;
    public string speech;
    public float duration;
    public float wantedAngle;
    public CharacterAnimation characterAnimation;
}
```
It stores the name of the character that is speaking, ID that identifies that character, the content of the speech, duration on how long the dialogue should be, the angle of the camera and as well the animation played.

Next, there is a <NarrativeDatabase> class that stores the data required for each conversation.

```csharp
[System.Serializable]
public class NarrativeDatabase
{
    public string name;
    public int IdNumber;
    public string Description; //HT For refrence usage only
    public List<Dialogue> DialogueList;
}
```

It includes the name of the conversation, the ID number of the conversation, the description of the conversation for internal uses and as well a list of <Dialougue> that forms the whole conversation.

Last but not least, there is a list of <NarrativeDatabase> to store multiple conversations

```csharp
public List<NarrativeDatabase> NarrativeDatabaseList;
```

Narrative Control Script:  
Overview of the script

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class NarrativeControlScript : MonoBehaviour
{
    List <GameObject> characterList;
    int pageNumber;
    int currentCharacterIndex;
    public Image speechBubble;
    public Image picture;
    List<Dialogue> tempDialogueList;
    public List<Sprite> pictureList;
    float timeElasped;
    GameObject toDeactivate; //! GameObjects to Reactivate after deactivated
    public bool isCompleted_L; //! Turns to true when a set of dialougue is completed

    public static NarrativeControlScript Instance {get; set;}

    void Awake()
    {
        if(Instance!= null && Instance != this)
                Destroy(gameObject);
        else
                Instance = this;
    }

    void Start ()
    {
        tempDialogueList = new List<Dialogue>();
    }

    void Update ()
    {
        if(speechBubble.gameObject.activeSelf)
        {
            if(Input.GetMouseButtonDown(0))
            {
                timeElasped = 0;
                NextPage ();
            }

            if(timeElasped > tempDialogueList[pageNumber].duration)
            {
                timeElasped = 0;
                NextPage ();
            }
            timeElasped += Time.deltaTime;
        }
    }

    void FindCharacters ()
    {
        characterList = new List<GameObject>();
        characterList.AddRange(GameObject.FindGameObjectsWithTag("Player"));
        characterList.AddRange(GameObject.FindGameObjectsWithTag("Characters"));
    }


    public void LoadConversation (int IdNumber)
    {
        toDeactivate = null;
        speechBubble.gameObject.SetActive(true);
        picture.gameObject.SetActive(true);
        //isCompleted_L = false;

        List<NarrativeDatabase> tempList = NarrativeDatabaseScript.Instance.NarrativeDatabaseList;

        FindCharacters ();
        characterList[0].GetComponent<CharacterControlScript>().enabled = false;
        characterList[0].transform.GetChild(1).GetComponent<Animator>().SetBool("Walk", false);

        for (int i = 0; i < tempList.Count; i++)
        {
            if(tempList[i].IdNumber == IdNumber)
            {
                tempDialogueList = tempList[i].DialogueList;
                break;
            }
        }

        pageNumber = -1;
        timeElasped = 0;
        NextPage ();
    }

    public void LoadConversation (int IdNumber, GameObject toDeactivate)
    {
        LoadConversation (IdNumber);
        this.toDeactivate = toDeactivate;
    }

    public void NextPage ()
    {
        if (pageNumber < tempDialogueList.Count - 1)
        {
            pageNumber ++;
            speechBubble.transform.GetChild(0).GetComponent<Text>().text
            = tempDialogueList[pageNumber].speech;

            speechBubble.transform.GetChild(1).GetChild(0).GetComponent<Text>().text
            = tempDialogueList[pageNumber].name;

            for (int i = 0; i < characterList.Count; i++)
            {
                if(characterList[i].GetComponent<CharacterIDTagScript>().ID == tempDialogueList[pageNumber].chracterID)
                {
                        currentCharacterIndex = i;
                        break;
                }
            }

            picture.sprite = pictureList[tempDialogueList[pageNumber].chracterID - 1];

            //speechBubble.transform.position = Camera.main.WorldToScreenPoint(characterList[currentCharacterIndex].transform.GetChild(0).position);
            characterList[0].GetComponent<CharacterControlScript> ().RotateCamera(tempDialogueList[pageNumber].wantedAngle);
            characterList[currentCharacterIndex].
            GetComponent<CharacterAnimationScript> ().ChangeAnimation(tempDialogueList[pageNumber].characterAnimation);
        }
        else
        {
            characterList[0].GetComponent<CharacterControlScript>().enabled = true;
            speechBubble.gameObject.SetActive(false);
            picture.gameObject.SetActive(false);
            isCompleted_L = true;
            if(toDeactivate != null)
            {
                    toDeactivate.SetActive(false);
            }
        }
    }
}
```

Firstly, let’s look into function called Load Conversation( ), this function handles the extraction of data from Narrative Database Script to a temporary list to handle the current conversation.

```csharp
public void LoadConversation (int IdNumber)
{
    toDeactivate = null;
    speechBubble.gameObject.SetActive(true);
    picture.gameObject.SetActive(true);
    //isCompleted_L = false;

    List<NarrativeDatabase> tempList = NarrativeDatabaseScript.Instance.NarrativeDatabaseList;

    FindCharacters ();
    characterList[0].GetComponent<CharacterControlScript>().enabled = false;
    characterList[0].transform.GetChild(1).GetComponent<Animator>().SetBool("Walk", false);

    for (int i = 0; i < tempList.Count; i++)
    {
        if(tempList[i].IdNumber == IdNumber)
        {
            tempDialogueList = tempList[i].DialogueList;
            break;
        }
    }

    pageNumber = -1;
    timeElasped = 0;
    NextPage ();
}
```

On Update, the next dialogue can be triggered by click, or when the duration for that dialogue is up.

```csharp
void Update ()
{
    if(speechBubble.gameObject.activeSelf)
    {
        if(Input.GetMouseButtonDown(0))
        {
            timeElasped = 0;
            NextPage ();
        }

        if(timeElasped > tempDialogueList[pageNumber].duration)
        {
            timeElasped = 0;
            NextPage ();
        }
        timeElasped += Time.deltaTime;
    }
}
```