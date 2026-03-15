---
title: Parcours Basics
summary: Creating the parcours and basic jumping without minigames.
date: 2026-02-15
weight: 4
---
**Map Changes**

The first thing I had to do was transform the map. The little fields placed on the street were made with Blender. For the textures, I created a low-poly color map for all my components with gimp. I used a combination of blue and orange, since I think these colors harmonize well with each other and with the template we got. I extended the map to create more complex textures for the minigames. There is also space for further extension.

{{< figure src="images/field.png" caption="This is the field I created with Blender." >}}

{{< figure src="images/vr_colors.png" caption="This is the low poly color map I used." >}}

After creating the fields, I placed them in the map at relatively equal distances. I considered different numbers of total fields and found that 45 was a good number, so the player has to perform some minigames, but not too many, since it should only be a small game. Every field has a collider and a tag named field, so I can track if the player is standing on a field. The fields are placed in the correct order, so I can get the correct position of the current and the next field by tracking how many jumps the player has performed. I placed the coins at the highest jumping point. Thus, the player would collect all coins without doing anything special. I thought of placing them a little bit off place so that the coins had to be pushed by the controllers while jumping, but I didn't know if it would be a little bit too fast with first swinging the arms to jump and then immediately pushing them to reach the coin. Therefore, I stayed with the simple approach of collecting all coins.

{{< figure src="images/parcours_new.png" caption="This is the parcours after placing the fields and changing the positions of the coins." >}}

**Jump Algorithm**

Only playing the minigames and then the player-character will jump all the fields by itself is too boring, so I wanted that every jump has to be performed individually. I thought of mapping a real jump of the person to the player-character, but since the player-character has a fix lenght to jump to the next field, it would be a lot of computation to get the real jump perfectly matched with the jump the player is performing. I didn't test it, but I think that it would lead to a lot of motion sickness if it isn't mapped perfectly. I wanted to focus on the minigames, so I implemented an easier approach, by swinging the arms to initiate the jump, which would keep the motion sickness smaller than only pushing a button. To reduce the motion sickness even further, the player has to look to the next field; otherwise, it would be possible to jump to the side or back.

To detect the armswing, I created invisible boxes that would be placed in the direction to the next field if the player has available jumps. I tested if both controllers were inside the boxes and that they are moving fast enough. Only if everything is fullfilled the player perform the jump.
Here you can see the code I use to jump from my position to a given position (the next field).

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
