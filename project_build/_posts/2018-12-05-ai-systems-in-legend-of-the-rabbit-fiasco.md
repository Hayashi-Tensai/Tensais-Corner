---
layout: post
title:  "Legend of the Rabbit Fiasco - AI Systems"
permalink: blog/2018/12/05/ai-systems-in-legend-of-the-rabbit fiasco
date:   2018-12-05 13:22:00 +0800
categories: game-development legend-of-the-rabbit-fiasco
---


The full script can viewed here:  
[https://github.com/Hayashi-Tensai/Legend-of-the-rabbit-fiasco/blob/master/Legend%20of%20the%20Rabbit%20Fiasco/Assets/Scripts/Rabbit/RabbitAIScript.cs](https://github.com/Hayashi-Tensai/Legend-of-the-rabbit-fiasco/blob/master/Legend%20of%20the%20Rabbit%20Fiasco/Assets/Scripts/Rabbit/RabbitAIScript.cs)

In this section I will be featuring the AI Systems for the rabbits in the game. Firstly, all three rabbits share the same AI script, differentiating them by using an enum:

```csharp
public enum RabbitType
{
   NORMAL_RABBIT,
   CARROT_RABBIT,
   BIG_RABBIT,
}
```
This script is then split into two parts, the movement and attack:

**Movement**

```csharp
void MovementAI()
{
    if (transform.position.x <= -8.5f)
        direction = Vector3.right;
    else if (transform.position.x >= 8.5f)
        direction = Vector3.left;

    if (rabbitType != RabbitType.CARROT_RABBIT)
    {
        if (lastPos.x < transform.position.x)
        {
            transform.GetChild(0).GetComponent<SpriteRenderer>().flipX = true;
        }
        else
        {
            transform.GetChild(0).GetComponent<SpriteRenderer>().flipX = false;
        }
    }

    lastPos = transform.position;
    transform.Translate(direction * speed * Time.deltaTime);
}
```
It's a simple algorithm that moves the rabbit left and right, except the carrot rabbit, that one stays stationary.

**Attack**

```csharp
void AttackAI()
{
    isAttack = true;
    if(rabbitType == RabbitType.NORMAL_RABBIT)
    {
        //! Chasing Player
        if (leftAttack)
        {
            transform.GetChild(0).GetComponent<SpriteRenderer>().flipX = false;
            transform.position = Vector3.MoveTowards(transform.position, PlayerControllerScript.instance.transform.position 
                + new Vector3(0.5f, 0.0f), speed * Time.deltaTime);
            atttackCollider.enabled = true;
        }
        else if(RightAttack)
        {
            transform.GetChild(0).GetComponent<SpriteRenderer>().flipX = true;
            transform.position = Vector3.MoveTowards(transform.position, PlayerControllerScript.instance.transform.position
                + new Vector3(-0.5f, 0.0f), speed * Time.deltaTime);
            atttackCollider.enabled = true;
        }
        else if (UpAttack)
        {
            isJump = true;
        }
    }
```
This part is the algorithm where the normal rabbit chases the player when the left or right raycast hits the player and​ activates the invisible hit box when they are in range of the player.

```csharp
    if(rabbitType == RabbitType.CARROT_RABBIT && intervalTime >= damageInterval)
    {
        anim.Play("Idle No Carrot");
        if (leftAttack)
        {
            carrot.GetComponent<CarrotScript>().InitSpawn(Vector3.left);
            carrot.GetComponent<SpriteRenderer>().flipX = false;
            Instantiate(carrot, transform.position + new Vector3(0.0f, 0.2f, 0.0f), Quaternion.identity);
            transform.GetChild(0).GetComponent<SpriteRenderer>().flipX = false;
        }
        else if(RightAttack)
        {
            carrot.GetComponent<CarrotScript>().InitSpawn(Vector3.right);
            carrot.GetComponent<SpriteRenderer>().flipX = true;
            Instantiate(carrot, transform.position + new Vector3(0.0f, 0.2f, 0.0f), Quaternion.identity);
            transform.GetChild(0).GetComponent<SpriteRenderer>().flipX = true;
        }
        else
        {
            int random = Random.Range(0, 2);
            if(random == 1)
            {
                carrot.GetComponent<CarrotScript>().InitSpawn(Vector3.right);
                carrot.GetComponent<SpriteRenderer>().flipX = true;
                transform.GetChild(0).GetComponent<SpriteRenderer>().flipX = true;
            }
            else
            {
                carrot.GetComponent<CarrotScript>().InitSpawn(Vector3.left);
                carrot.GetComponent<SpriteRenderer>().flipX = false;
                transform.GetChild(0).GetComponent<SpriteRenderer>().flipX = false;
            }

            Instantiate(carrot, transform.position + new Vector3(0.0f, 0.2f, 0.0f), Quaternion.identity);
        }

        intervalTime = 0.0f;
    }

    if (rabbitType != RabbitType.CARROT_RABBIT)
    {
        //! Dealing Attacks
        if (intervalTime >= damageInterval && isHit)
        {
            PlayerControllerScript.instance.currentHealth -= damage;
            intervalTime = 0.0f;

            if (rabbitType == RabbitType.NORMAL_RABBIT)
                anim.Play("Attack");
        }
    }     
}
```
As for the carrot rabbit, there is two conditions as you can see, checking the booleans, "leftAttack" and "rightAttack", these are turned true when the player is its respective direction, and it will shoot the carrot to the direction of the player. Else, it would shoot the carrot at a random direction either left or right.

The last condition at the bottom is for the normal rabbit and the big rabbit, where isHit is true (isHit checks for collision of the attack hit box with the player), it will damage the player on an interval until the player leaves the hitBox.
