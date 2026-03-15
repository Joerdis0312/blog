---
title: Minigames
summary: An overview of the integrated minigames and their underlying technical implementation.
date: 2026-02-15
weight: 3
---

I designed the minigames to be based on the same core mechanics. My goal was to create a versatile system, so I implemented a simple collision-based interaction system. Each interactive object is equipped with a collider and a unique tag, enabling the *OnTriggerEnter* method to identify the object and execute the corresponding logic.

Which minigame a player encounters is determined randomly, with the exception of the T-shape interaction, which must be performed at the end of each of the three parcour sections. To ensure all objects face the next field, I calculate a local coordinate system upon entering a new field. The "up" vector is calculated only once at the start of the game.

```c
// compute the direction facing to the next field
direction = new Vector3(nextFieldPos.x - currentFieldPos.x, 0f, nextFieldPos.z - currentFieldPos.z).normalized;
up = Vector3.up;
right = Vector3.Cross(up, direction).normalized;
```

**Dice Mechanic**

{{< figure src="images/dice.png" caption="The dice asset created in Blender." >}}

The first minigame is a board game classic: the dice. I modeled the dice in Blender and imported it into Unity. To position the dice toward the next field, I calculate the vector between the current and the next field and place the dice along this line. Upon collision, the dice is deactivated, and the player receives a random number between 1 and 6.

{{< video src="dice" type="video/mp4" caption="Demonstration of the dice mechanic and subsequent jump sequence." >}}

**Card Mechanic**

<div style="display: flex; gap: 20px; align-items: flex-start;">
    {{< figure src="images/card_front.png" caption="The front side of the cards I created." >}}
    {{< figure src="images/card_back.png" caption="The back side of the cards I created." >}}
</div>

The second minigame involving drawing a card is also inspired by traditional board games. When a player enters the field collider and the card minigame is triggered, four card backs are displayed. Colliding with a card causes it to disappear, revealing a card front that displays the number of jumps awarded. There are seven possible cards:
 - One field forward
 - Two fields forward
 - Three fields forward
 - Last roll again
 - Half of last roll
 - Double of last roll
 - One field backward

To implement cards 4–6, I track the value of the previous roll. This value is initialized to 1 to prevent errors if a player draws these cards at the start. *Half of last roll* is rounded down, except when the last roll was 1, in which case it defaults to *One field forward*.

Implementing the *One field backward* card was more challenging, as it required modifications to the jumping algorithm. I had to set the next field to the previous one and recalculate the direction for the jump boxes and the player's orientation. I also implemented a isJumping boolean to prevent accidental collisions with interactive objects (dice, cards, or cubes) during the backward jump, as the player might enter a field collider before the horizontal movement is fully completed. Additionally, I added logic to ensure players cannot receive a backward card at the very beginning or immediately after a T-shape interaction.

{{< video src="cards" type="video/mp4" caption="Demonstration of the card mechanic and subsequent jump sequence." >}}

**Reaction Game**

<div style="display: flex; gap: 20px; align-items: flex-start;">
    {{< figure src="images/cube_blue.png" caption="The blue cube I created." >}}
    {{< figure src="images/cube_orange.png" caption="The orange cube I created." >}}
</div>

Unlike the previous luck-based games, the reaction game allows players to earn more jumps through performance. There are two types of cubes: orange (to be hit with the right hand) and blue (to be hit with the left hand).

Upon entering the field, a cube is spawned on a plane orthogonal to the direction of the next field. After the first hit, a 10-second timer starts, and new cubes appear randomly. The number of jumps awarded is one-quarter of the total hits (rounded down, with a minimum of one).

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

The T-shape interaction serves as a mandatory checkpoint. The task requires aligning a T-shape with a target position and rotation. I adapted my pushing interaction style by having the T-shape move autonomously across different axes, requiring the player to hit a *stop* button at the optimal moment.

The interaction follows a three-step sequence:

 1. Horizontal Alignment: Movement along the X or Z axis.
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

After finishing the three steps, the player must hit the *done* button. The system then computes the difference between the player's final placement and the target transformation. Based on the accuracy of the alignment, the player is awarded additional jumps, which are added to their existing jump count.

{{< video src="interaction" type="video/mp4" caption="Demonstration of the T-shape interaction and subsequent jump sequence." >}}