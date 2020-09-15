---
layout: post
title:  "Enchanter - Ifrit's Attack Algorithm"
author: Hayashi Tensai
excerpt: This one of the first scripts that I wrote in Unity 3 years ago in our first game project together as a team, it's quite a simple algorithm that controls Ifrit's attack pattern and it is split into 3 phases.
permalink: blog/2019/01/10/ifrit-attack-algorithm-in-enchanter
date:   2019-01-10 20:58:00 +0800
categories: game-development enchanter
slideshow:
- https://raw.githubusercontent.com/Hayashi-Tensai/Tensais-Corner/master/assets/blog-images/game-dev/enchanter-phase-3-1.jpg
- https://raw.githubusercontent.com/Hayashi-Tensai/Tensais-Corner/master/assets/blog-images/game-dev/enchanter-phase-3-2.jpg
- https://raw.githubusercontent.com/Hayashi-Tensai/Tensais-Corner/master/assets/blog-images/game-dev/enchanter-phase-3-3.jpg
- https://raw.githubusercontent.com/Hayashi-Tensai/Tensais-Corner/master/assets/blog-images/game-dev/enchanter-phase-3-4.jpg
---

View the full script at: 
[https://drive.google.com/open?id=1kxwx-zIiyVCkFe4fCjeH1WTfvwLTE6EL](https://drive.google.com/open?id=1kxwx-zIiyVCkFe4fCjeH1WTfvwLTE6EL)

This one of the first scripts that I wrote in Unity 3 years ago in our first game project together as a team, it's quite a simple algorithm that controls Ifrit's attack pattern and it is split into 3 phases.

```csharp
void PhaseController() {
    if (phaseCounter == 1) {
            PhaseOne ();
            if (resetFlag) {
                    counterFloat = 0.0f;
                    resetFlag = false;
            }
    } 
    else if (phaseCounter == 2) {
            PhaseOne ();
            PhaseTwo ();
    } 
    else if (phaseCounter == 3) {
            PhaseThree ();
    }
}
```

First there is this PhaseController() that takes control which phase to call.

Now's let look into phase 1. In phase 1 if the player stays on one platform for more than 5 seconds Ifrit will start lighting the platform on fire.

![enchanter-phase-1](https://raw.githubusercontent.com/Hayashi-Tensai/Tensais-Corner/master/assets/blog-images/game-dev/enchanter-phase-1.jpg)

```csharp
void PhaseOne () {
    //! This is to prevent the fire pillar to spawn during the fire wave *Thianchai*
    if (isSpawnFireWave) {
        playerStayTime += Time.deltaTime;
    }
    else 
    {
        isPlayBurnAudio = true;
        playerStayTime = 0;
        isPillar = false;
        currentPlatform.gameObject.GetComponent <SpriteRenderer> ().sprite = defaultSprite;

    }

    // this loop only runs before the platform lights up, once it lights up, dont search for current platform anymore //Thianchai
    for (int i = 0; i < RandomedWordsComparerIntegrated.Instance.platformList.Count; i++) {
        if (playerCoordinate.x == RandomedWordsComparerIntegrated.Instance.platformList [i].gameObject.transform.position.x && playerStayTime < 5) {
            PlatformSpriteDefaulter (currentPlatform);
            currentPlatform = RandomedWordsComparerIntegrated.Instance.platformList [i].gameObject;
        }
    }
```
In this first part of function, the for loop checks for each and every platform for the player and will continue to lock on the player's position until the player stays on one platform for more than 5 seconds.

```csharp
    if (playerStayTime >= 5 && playerStayTime < 8) {
        if (isPlayBurnAudio) {
            SoundManagerScript.Instance.PlaySFXNoPitchChange (AudioClipID.SFX_FIRE_BURN);
            isPlayBurnAudio = false;
        }

        currentPlatform.gameObject.GetComponent <SpriteRenderer> ().sprite = preFireSprite;
    }

    if (playerStayTime >= 8 && playerStayTime < 11) {
        if (!isPillar) {
            currentPlatform.gameObject.GetComponent <SpriteRenderer> ().sprite = defaultSprite;
            isPlayBurnAudio = true;
            SoundManagerScript.Instance.PlaySFXNoPitchChange (AudioClipID.SFX_FIRE_PILLAR);
            Instantiate (firePillar,new Vector2(currentPlatform.transform.position.x, currentPlatform.transform.position.y + 50), Quaternion.identity, this.transform);
            isPillar = true;
        }
    } 
    else if (playerStayTime >= 11) {
        isPlayBurnAudio = true;
        playerStayTime = 0;
        isPillar = false;
        currentPlatform.gameObject.GetComponent <SpriteRenderer> ().sprite = defaultSprite;
    }


    if (gameObject.GetComponent <BossHealthManager> ().currentHealth <= (gameObject.GetComponent <BossHealthManager> ().startingHealth / 2) && gameObject.GetComponent <BossHealthManager> ().currentHealth > 0) {

        phaseCounter = 2;
        resetFlag = true;

        if(gameObject.GetComponent<Animator>().runtimeAnimatorController.name == "Ifrit")
        {
            gameObject.GetComponent<Animator>().runtimeAnimatorController = ifritOverride;
            gameObject.GetComponent<IfritAnimation>().PlayAnimation(IfritAnimationType.IA_TRANSFORM);
        }
    }

    BloodSpurt ();
}
```

Once if goes beyond it stops checking on the player's current position and proceeds to lock on that platform regardless whether the player is there. It then starts a precursor animation to warn the player on the incoming fire.

After the 8th second, the pillar of fire comes up and stays for 3 seconds before setting the timer back to zero and repeating the function loop.

In phase 2, Ifrit remains the fire pillar attack, and adds two more attack types to its attack pattern, the fire meteor and fire wave. Phase 2 starts when Ifrit's health falls below 50%.
![enchanter-phase-2-fireball](https://raw.githubusercontent.com/Hayashi-Tensai/Tensais-Corner/master/assets/blog-images/game-dev/enchanter-phase-2-fireball.jpg)
Above: The fire meteor

![enchanter-phase-2-wave](https://raw.githubusercontent.com/Hayashi-Tensai/Tensais-Corner/master/assets/blog-images/game-dev/enchanter-phase-2-wave.jpg)
Above: The fire wave

```csharp
void PhaseTwo () {
    if (counter < 3) {
        phase2Randomization = Random.Range (1, 9);
    }
    else if (counter >= 3 && counter < 16) {
        if (phase2Randomization >= 1 && phase2Randomization <= 7) {
            meteorCount += Time.deltaTime;
            if (meteorCount > 5) {
                gameObject.GetComponent<IfritAnimation>().PlayAnimation(IfritAnimationType.IA_METEOR);
                SoundManagerScript.Instance.PlaySFXNoPitchChange(AudioClipID.SFX_IFRIT_METEOR);
                SpawnMeteor ();
                meteorCount = 0;
            }
        }
        else if (phase2Randomization == 8) 
        {
            if (isSpawnFireWave) 
            {
                isSpawnFireWave = false;
                //! Turning on blackout *Thianchai*
                blackout.SetActive(true);
                gameObject.GetComponent<IfritAnimation>().PlayAnimation(IfritAnimationType.IA_FIREWAVE);
                SoundManagerScript.Instance.PlaySFX (AudioClipID.SFX_IFRIT_CAST_FIREWAVE);
                Instantiate (fireWave, new Vector2 ((Screen.width / 3 * 2) + 50, (-Screen.height / 3) + 175), Quaternion.identity, this.transform);

                //! Teleporting player to safety :) *Thianchai*
                SoundManagerScript.Instance.PlaySFX (AudioClipID.SFX_TELEPORT);
                player.transform.position = new Vector2 (firstPlatform.transform.position.x, firstPlatform.transform.position.y);

                //! Deactivating all platforms *Thianchai*
                for (int i = 0; i < RandomedWordsComparerIntegrated.Instance.platformList.Count; i++) 
                {
                    RandomedWordsComparerIntegrated.Instance.platformList [i].transform.GetChild (0).gameObject.SetActive (false);
                    RandomedWordsComparerIntegrated.Instance.platformList [i].GetComponent<SpriteRenderer> ().color = Color.black;
                    RandomedWordsComparerIntegrated.Instance.platformList [i].GetComponent<TypableObject> ().RemoveTypableObjectFromObjectList ();
                }
            }
        }
    }

    if (counter > 20) {
        isSpawnFireWave = true;
        counterFloat = 0;
    }
```

In addition with the earlier fire pillar algorithm in phase 1 still running, the PhaseTwo() will randomize one of the two above attack patterns every three seconds.

If the randomization comes out with a value less than or equals to seven (87.5% chance) It will summon a meteor.

Last but not least there is phase 3, it triggers when Ifrit losses all its health.
```csharp
    if (gameObject.GetComponent <BossHealthManager> ().currentHealth <= 0) {

        phaseCounter = 3;
        exorcismWords.SetActive (true);
        resetFlag = true;
        if (counter > 5) {
            PlatformSpriteDefaulter (currentPlatform);
        }
    }
}

void PhaseThree () {
    if (counter >= 35) 
    {
        phaseCounter = 2;
        exorcismWords.GetComponent<ExorcismTypableObject> ().ResetPhraseCount ();
        exorcismWords.SetActive (false);
        resetFlag = true;
        gameObject.GetComponent <BossHealthManager> ().currentHealth = (gameObject.GetComponent <BossHealthManager> ().startingHealth) / 10;
    }
}
```
In the last part of PhaseTwo() it has a condition to check if Ifrit's health is at zero, and if the condition holds true, it will update the phaseCounter to 3.

In phase 3, several short sentences will appear and the player will have to finish typing all of it within 35 seconds to clear the stage , if the player fails to do so, it will regain 10% of its health and revert back to phase 2.
<div>{%- include slideshow.html images=page.slideshow -%}</div>
Above: The final type sequence of phase 3.