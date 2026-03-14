---
title: Minigames
summary: Here you can see which minigames I included into the game.
date: 2026-02-15
weight: 3
---

I wanted that the minigames are all based on the same mechanics. The first thing that came into my mind that can be used in many different ways, was a simple collider hitting. Thus I based all my minigames on this simple approach. Which minigame you will get is completely random with the exception of the T-shape interaction which the player has to perform once at the end of each of the three parts in the parcours.

To implement the collider hitting, I placed the objects inside the world and set them to not active in the beginning of the game. Only if they are used, they will be active. Every object got an collider and an individual tag, so that I can check in OnTriggerEnter which object I hitted and then calling the correct method for this specific object.

**Dice**

{{< figure src="images/dice.png" caption="This is the dice I created." >}}

The first minigame I included was classical for a normal board game a dice. I first created the dice with blender and then inlcuded it to unity. For placing the dice on the direction to the next field I computed the vector from the current field to the next field and placed the dice on this line. The dice got a collider, thus it could be tracked if it was hit. After hitting the dice will be set to not active and the player got a random number between 1 and 6 like a normal dice.

{{< video src="dice" type="video/mp4" caption="Here you can see me performing this minigame and jumping to the next field." >}}

**Cards**

<div style="display: flex; gap: 20px; align-items: flex-start;">
    {{< figure src="images/card_front.png" caption="This is the front side of the cards I created." >}}
    {{< figure src="images/card_back.png" caption="This is the back side of the cards I created." >}}
</div>

The second minigame is also based on a classical board game and is drawing a card. After entering the field collider and getting the random number for the card minigame four card backsides will be shown. Hitting one of the cards triggers this card to disappear and a card frontside will be placed at the same position with a text written on it how many jumps the player got. There are seven possible cards:
 - One field forward
 - Two fields forward
 - Three fields forward
 - Last roll again
 - Half of last roll
 - Double of last roll
 - One field backward

For including the cards 4-6 I had to track how much was the last roll. While starting the last roll track is set to 1, thus it won't cause an error if the player got these card at the beginning. Half of the last roll is rounded down, with the exception if the last roll was one, then you will get an one field forward card. Including the backwards card was more difficult since I had to rewrite some little parts of my jumping algorithm. I had to set the next field to the last and compute the new direction for placing the jump boxes and computing the direction the player must face to jump. And I also had to include a bool jumping, otherwise I hit the dice, card or cube while performing the jump. I fixed it that the player can't get the backwards card at the beginning or for going back to the T-shape interaction. You also can't can the cards half or double of the last roll right after this card. You will simple get an one field forward card.

{{< video src="cards" type="video/mp4" caption="Here you can see me performing this minigame and jumping to the next field." >}}

**Reaction Game**

<div style="display: flex; gap: 20px; align-items: flex-start;">
    {{< figure src="images/cube_blue.png" caption="This is the blue cube I created." >}}
    {{< figure src="images/cube_orange.png" caption="This is the orange cube I created." >}}
</div>

The previous minigames were based on board games, relativly simple and are completely random in the outcome. In this minigame the player can get more jumps the better he will perform on the game. There are two kinds of cubes: orange and blue. The orange one has to be hit by the right hand and the blue one by the left hand. After entering the field collider one random cube will be placed on a plane orthogonal to the direction to the next field, where an orange cube is be placed a little more on the right side and the blue one on the left side. After hitting one cube a new one will be placed randomly. After hitting the first cube the player has 10 seconds time to hit as many cubes as he could. The number of jumps the player get is a fourth (rounded down, but at least one) of the number of hitted cubes. The following code shows the update method.

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

{{< video src="reaction" type="video/mp4" caption="Here you can see me performing this minigame and jumping to the next field." >}}

**T-Shape Interaction**

The T-shape interaction was a given task. I included it a a minigame on the last fields of the three parcours parts and made them mandatory so you can't skip them even if you have available jumps left. The task was to place a random positionated T-shape at a target position and rotation. I thought of how I could solve this with my basic pushing approach and came to the idea that the T-shape is moving on its own and the player has to stop it a the right position and rotation. So if you start the task the T-shape will move left and right around the target position and the player can push a stop button to stop the T-shape. After this the T-shape will move up and down, and after stopping this moving it will turn around its y-axis.

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

After finishing the three steps the player has to hit the done button and then the difference to the target will be computed and based on how good the player performed the player will get jumps additionally to the jumps he had before.

{{< video src="interaction" type="video/mp4" caption="Here you can see me performing this minigame and jumping to the next field." >}}