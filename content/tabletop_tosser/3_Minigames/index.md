---
title: Minigames
summary: An overview of the integrated minigames and their underlying technical implementation.
date: 2026-02-15
weight: 3
---

I wanted the minigames to be all based on the same mechanics. The first thing that came to my mind that can be used in many different ways was a simple collision-based interaction system. Thus, I based all my minigames on this simple approach. Which minigame you will get is completely random, except for the T-shape interaction, which the player has to perform once at the end of each of the three parts in the parcours. Each interactive object is equipped with a collider and a unique tag, enabling the *OnTriggerEnter* method to identify the hit object and execute the corresponding logic.

To place all objects facing to the next field, I calculated a local coordinate system after entering the new field. The vector up is only calculated once at the beginning of the game.

```c
// compute the direction facing to the next field
direction = new Vector3(nextFieldPos.x - currentFieldPos.x, 0f, nextFieldPos.z - currentFieldPos.z).normalized;
up = Vector3.up;
right = Vector3.Cross(up, direction).normalized;
```

**Dice Mechanic**

{{< figure src="images/dice.png" caption="The dice I created." >}}

The first minigame I included was a classic for a normal board game: a dice. I first created the dice with Blender and then included it in Unity. For placing the dice in the direction to the next field, I computed the vector from the current field to the next field and placed the dice on this line. After hitting, the dice will be deactivated, and the player will get a random number between 1 and 6 like a normal dice.

{{< video src="dice" type="video/mp4" caption="Demonstration of the dice mechanic and subsequent jump sequence." >}}

**Card Mechanic**

<div style="display: flex; gap: 20px; align-items: flex-start;">
    {{< figure src="images/card_front.png" caption="The front side of the cards I created." >}}
    {{< figure src="images/card_back.png" caption="The back side of the cards I created." >}}
</div>

The second minigame is also based on a classical board game, and involves drawing a card. After entering the field collider and getting the random number for the card minigame, four card backsides will be shown. Colliding with one of the cards triggers this card to disappear, and a card frontside will be placed at the same position with a text written on it showing how many jumps the player got. There are seven possible cards:
 - One field forward
 - Two fields forward
 - Three fields forward
 - Last roll again
 - Half of last roll
 - Double of last roll
 - One field backward

To include the cards 4-6, I had to track how much was the last roll. While starting the last roll track is set to 1, thus it won't cause an error if the player got this card at the beginning. *Half of the last roll* is rounded down, with the exception that if the last roll was one, you will get a *One field forward* card. Including the backwards card was more difficult since I had to rewrite some small parts of my jumping algorithm. I had to set the next field to the last and compute the new direction for placing the jump boxes and computing the direction the player must face to jump. And I also had to include a bool jumping, otherwise I hit the dice, card, or cube while performing the backward jump, since the players enters the field collider before completing the whole horizontal movement of the jump. I also included that the player can't get the backwards card at the beginning or on the field right after the T-shape interaction. You also can't get the cards half or double of the last roll right after this card. You will simply get a *One field forward* card.

{{< video src="cards" type="video/mp4" caption="Demonstration of the card mechanic and subsequent jump sequence." >}}

**Reaction Game**

<div style="display: flex; gap: 20px; align-items: flex-start;">
    {{< figure src="images/cube_blue.png" caption="The blue cube I created." >}}
    {{< figure src="images/cube_orange.png" caption="The orange cube I created." >}}
</div>

The previous minigames were based on board games, relatively simple, and were completely random in outcome. In this minigame, the player can get more jumps the better he will perform on the game. There are two kinds of cubes: orange and blue. The orange one has to be hit by the right hand and the blue one by the left hand. After entering the field collider, one random cube will be placed on a plane orthogonal to the direction to the next field, where an orange cube will be placed a little more on the right side and a blue one on the left side. After hitting one cube, a new one will be placed randomly. After hitting the first cube, the player has 10 seconds to hit as many cubes as he can. The number of jumps the player gets is a quarter (rounded down, but at least one) of the number of hit cubes. The following code shows the update method.

```c
void Update()
{
    // stop the reaction minigame 10 seconds after hitting the first cube
    // controller == OVRInput.Controller.LTouch: otherwise it will be executed for both hands which leads to some inconsistences
    if ((Time.time - parkourCounter.reactionTime) > 10 && parkourCounter.isTimerRunning && controller == OVRInput.Controller.LTouch)
    {
        CubeOrange.SetActive(false);
        CubeBlue.SetActive(false);

        // get jumps based on how many cubes are hit in the 10 seconds
        parkourCounter.availableJumps = (int)(parkourCounter.reactionCount / 4.0f);

        if (parkourCounter.availableJumps == 0)
        {
            parkourCounter.availableJumps = 1;
        }

        // remember the roll
        parkourCounter.lastRoll = parkourCounter.availableJumps;

        // reset the values for the next time
        parkourCounter.isTimerRunning = false;
        parkourCounter.reactionCount = 0;
        parkourCounter.reactionTime = 0;
    }
}
```

{{< video src="reaction" type="video/mp4" caption="Demonstration of the reaction game and subsequent jump sequence." >}}

**T-Shape Interaction**

The T-shape interaction was a specific requirement, integrated as a mandatory checkpoint. The task was to place a randomly positioned T-shape at a target position and rotation. I thought of how I could solve this with my basic pushing approach and came to the idea that the T-shape moves autonomously across different axes, and the player must stop it at the optimal position and rotation by hitting a *stop* button.

The task follows a three-step sequence:

 1. Horizontal Alignment: Movement along the X or Z axis (depending on the parcours section).
 2. Vertical Alignment: Movement along the Y-axis.
 3. Rotational Alignment: Rotation around the Y-axis.

```c
void Update()
{
    // startposition with the center as the target
    startPosition = TargetT.transform.position;

    // first round: moving left-right
    if (parkourCounter.interacting == 1)
    {
        // for the third banner: moving along x-axis
        if (loco.currentFieldCount == 45)
        {
            float newPos = Mathf.PingPong(Time.time * 2f, 3f) - 1.5f;
            transform.position = new Vector3(transform.position.x, transform.position.y, startPosition.z + newPos);
        }
        // for the first and second banner: moving along y-axis
        else
        { 
            float newPos = Mathf.PingPong(Time.time * 2f, 3f) - 1.5f;
            transform.position = new Vector3(startPosition.x + newPos, transform.position.y, transform.position.z);
        }
    }
    // second round: moving up-down
    if (parkourCounter.interacting == 2)
    {
        float newPos = Mathf.PingPong(Time.time * 2f, 3f) - 1.5f;
        transform.position = new Vector3(transform.position.x, startPosition.y + newPos, transform.position.z);
    }
    // third round: turning around y-axis
    if (parkourCounter.interacting == 3)
    {
        transform.Rotate(new Vector3(0, 1, 0));
        donePanel.SetActive(true);
    }
}
```

After finishing the three steps, the player has to hit the *done* button, and then the difference to the target will be computed, and based on how well the player performed, the player will get additional jumps to the jumps he had before.

{{< video src="interaction" type="video/mp4" caption="Demonstration of the T-shape interaction and subsequent jump sequence." >}}