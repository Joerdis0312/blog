---
title: Parcours Basics
summary: Implementation of the environment and the core jumping algorithm without mini-game integration.
date: 2026-02-15
weight: 4
---
**Map Adjustments and Asset Creation**

The first step involved modifying the existing environment. The little fields placed on the street were made with Blender. To ensure visual consistency, I created a low-poly color map using GIMP. I used a combination of blue and orange, since I think these colors harmonize well with each other and with the provided template. I extended the map to create more complex textures for the minigames. There is also space for further extension.

{{< figure src="images/field.png" caption="The field I created with Blender." >}}

{{< figure src="images/vr_colors.png" caption="The low poly color map used for texturing." >}}

After modeling, I placed them in the map at relatively equal intervals along the path. I considered different numbers of total fields and settled on 45 fields to balance gameplay duration — ensuring the player engages with several minigames without making the experience overly repetitive. Every field has a collider and a tag named "field", so I can track if the player is standing on a field. By giving the fields names like "Field_0" in a sequential order, I can get the correct position of the current and the next field by tracking how many jumps the player has performed.

I placed the collectible coins at the highest jumping point. Thus, the player would collect all coins without doing anything special. I thought of placing them a little bit off place so that the coins had to be pushed by the controllers while jumping, but I didn't know if it would be a little bit too fast with first swinging the arms to jump and then immediately pushing them to reach the coin. Therefore, I stayed with the simple approach of automatic collecting.

{{< figure src="images/parcours_new.png" caption="The updated parcours after placing the fields and adjust the coin placements." >}}

**Jump Algorithm and Locomotion**

To ensure an engaging experience, I decided that each jump must be initiated manually rather than having the character move automatically. I thought of mapping a real jump of the person to the player-character, but since the player-character has a fix lenght to jump to the next field, it would need complex calculations to get the real jump perfectly matched with the jump the player is performing. Any discrepancy here would likely induce motion sickness. I wanted to focus on the minigames, so I implemented an easier approach, by swinging the arms to initiate the jump, which would keep the motion sickness smaller than only pushing a button. To further enhance comfort, the jump is only triggered if the player is looking toward the next field, preventing disorienting sideways or backward movements.

To detect the armswing, I created invisible boxes that would be placed in the direction to the next field when jumps are available. I tested if both controllers were inside the boxes and that they are moving fast enough. Only if everything is fullfilled the player perform the jump.

Here you can see the code I use to jump from my position to a given position (the next field). The code was provided by AI by asking how I can jump to a desired position, and since it worked very good, I didn't change anything.

```c
// values for the jumps
[SerializeField] float jumpHeight = 5;
public float jumpDuration = 2.83f;

IEnumerator JumpCoroutine(Vector3 target)
{
    jumping = true;
    Vector3 startPos = transform.position;
    float elapsed = 0f;

    while (elapsed < jumpDuration)
    {
        elapsed += Time.deltaTime;
        float normalizedTime = elapsed / jumpDuration;

        // linear interpolation for horizontal moving
        Vector3 currentPos = Vector3.Lerp(startPos, target, normalizedTime);

        // parabel for the height
        currentPos.y += Mathf.Sin(normalizedTime * Mathf.PI) * jumpHeight;

        transform.position = currentPos;
        yield return null;
    }

    // place the player at the exact correct position so that it is not slightly off
    transform.position = target;
    jumping = false;
}
```

To first test if the basic jump without minigames worked, I add one jump to the available jumps at entering the field collider. Thus, I first checked the core mechanics and included the minigames one-by-one.