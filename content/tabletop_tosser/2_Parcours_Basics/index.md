---
title: Parcours Basics
summary: Implementation of the environment and the core jumping algorithm without mini-game integration.
date: 2026-02-15
weight: 4
---
**Map Adjustments and Asset Creation**

The first step involved modifying the existing environment. The small fields placed on the track were created using Blender. To ensure visual consistency, I developed a low-poly color map in GIMP, using a combination of blue and orange. I chose these colors because they harmonize well with each other and the provided template. I also extended the map to create more complex textures for the minigames, leaving sufficient space for future expansions.

{{< figure src="images/field.png" caption="The field I created with Blender." >}}

{{< figure src="images/vr_colors.png" caption="The low-poly color map used for texturing." >}}

After modeling the assets, I placed them along the path at relatively equal intervals. I experimented with different total field counts and settled on 45 fields to balance gameplay duration—ensuring the player engages with several minigames without the experience becoming repetitive. Each field has a collider and a tag named "field," allowing me to track if the player is standing on it. By naming the fields sequentially (e.g., "Field_0"), I can determine the correct position of the current and the next field by tracking the number of jumps the player has performed.

I placed the collectible coins at the peak of the jump trajectory. Consequently, the player collects them automatically during the jump. I initially considered placing them slightly off-center so they would have to be "pushed" by the controllers mid-air. However, I was concerned that the transition from swinging arms to initiate the jump to immediately reaching for coins might be too fast or taxing. Therefore, I opted for the simpler approach of automatic collection.

{{< figure src="images/parcours_new.png" caption="The updated parcour after placing the fields and adjusting the coin placements." >}}

**Jump Algorithm and Locomotion**

To ensure an engaging experience, I decided that each jump must be initiated manually rather than moving the character automatically. I considered mapping a physical jump by the user to the player-character. However, since the character moves a fixed distance to the next field, this would require complex calculations to align the physical movement with the virtual jump. Any discrepancy here would likely induce motion sickness. To focus on the minigames, I implemented a more accessible approach: swinging the arms to initiate the jump. This method reduces motion sickness more effectively than simply pressing a button. To further enhance comfort, the jump is only triggered if the player is looking toward the next field, preventing disorienting sideways or backward movements.

To detect the arm swing, I created invisible trigger boxes that are positioned toward the next field when a jump is available. The system checks if both controllers are within these boxes and moving at a sufficient speed. The jump is only performed if both conditions are met.

Below is the code used to jump from the current position to a target position (the next field). The logic was adapted from an AI suggestion on how to jump to a specific coordinate; since it performed well during testing, I kept the implementation as is.

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

To test the basic jumping mechanic before integrating minigames, I initially added one "available jump" whenever the player entered a field collider. This allowed me to verify the core mechanics before gradually including the minigames.