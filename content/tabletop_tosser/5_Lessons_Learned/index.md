---
title: Technical Challenges and Lessons Learned
summary: An analysis of the technical obstacles encountered during development and the corresponding solutions.
date: 2026-02-15
weight: 1
---

**Hardware Constraints and Debugging**

The primary challenge during development was a significant hardware limitation: my laptop's GPU was insufficient to support Meta Quest Link. Therefore, I had to build and deploy the entire project for every minor change, installing it on the headset via SideQuest. The main disadvantage was the inability to use real-time debugging; I could not see live error messages or logs, which made troubleshooting considerably more difficult.

**Logic Errors and System Initialization**

A problem that would have been much easier to resolve with access to error logs was a *NullReferenceException* during the creation of the card minigame. Upon hitting a card, I intended to deactivate the card back and instantiate a white card at the same position. Initially, the white card appeared in the wrong spot. I later discovered that this was due to a different rotation setting used to ensure the text faced the player correctly. However, during my initial troubleshooting, I deleted and re-imported the card asset into the scene but forgot to re-assign it in the Inspector. Consequently, while the card back was disabled correctly, the new white card failed to appear. Without logs, I mistakenly assumed it was a positioning error. Only after realizing the game was halting at that point, I used AI to help identify potential logic gaps. This taught me a vital lesson: whenever a new GameObject is added to a script, the first step must be to verify the assignment in the Unity Inspector. I also learned to pay closer attention to layers, as using the wrong layer on the dice initially prevented collision detection from working as intended.

Another issue was that the application occasionally failed to launch on the headset, even without changes to the code. This was caused by a race condition during startup. I resolved this by manually loading the XR subsystems using a custom script (provided by AI) attached to an empty object at the top of the Unity hierarchy:

```c
public class XRManualLoader : MonoBehaviour
{
    // manuelles Laden, damit es nicht zu race-conditions kommt und das Spiel korrekt startet

    void Start()
    {
        // Starte die Initialisierung mit einer kleinen Verzögerung
        StartCoroutine(StartXRCoroutine());
    }

    public IEnumerator StartXRCoroutine()
    {
        Debug.Log("Initialisiere XR...");

        // Initialisiert den in den Settings gewählten Loader (z.B. Oculus/OpenXR)
        yield return XRGeneralSettings.Instance.Manager.InitializeLoader();

        if (XRGeneralSettings.Instance.Manager.activeLoader == null)
        {
            Debug.LogError("XR Initialisierung fehlgeschlagen. Prüfe die Logs!");
        }
        else
        {
            Debug.Log("XR initialisiert. Starte Subsysteme...");
            XRGeneralSettings.Instance.Manager.StartSubsystems();
        }
    }

    // Optional: XR beim Beenden sauber herunterfahren
    void OnDisable()
    {
        if (XRGeneralSettings.Instance.Manager.isInitializationComplete)
        {
            XRGeneralSettings.Instance.Manager.StopSubsystems();
            XRGeneralSettings.Instance.Manager.DeinitializeLoader();
            Debug.Log("XR sauber gestoppt.");
        }
    }
}
```

I suspect the issue arose because the player and the field markers were being initialized in an inconsistent order, sometimes leaving the player without a valid starting position.

**Minigame Refinement**

In the reaction minigame, I encountered a bug where the jump count was calculated twice at the end of the ten-second timer. The first calculation would reset the hit count to zero, and the second would then default to the minimum of one jump. This happened because the script was running on both controllers simultaneously. I fixed this by adding a check to ensure the logic only executes for the left controller.

In the T-shape interaction, the object initially moved in only one direction instead of oscillating. I had omitted the *-1.5f* offset in the *Mathf.PingPong* method, which prevented it from returning to a negative range. Additionally, the oscillation center was off-target; I corrected this by setting the startPosition to *TargetT.transform.position* instead of the script's own transform. Finally, to prevent accidental double-triggers of the *stop* button—often caused by the player's hand passing through the collider twice during a single motion—I extended the collider's depth to the rear.

**Version Control Challenges**

During the deployment of this blog, a large video file exceeded GitHub's file size limits. Even after local compression, the original large file remained in the Git history, causing subsequent pushes to fail. Resolving this required a git revert (or git filter-repo) of the specific commit to purge the large file from the history before re-adding the compressed version.

**Conclusion**

I am very satisfied with the project's modular design, which allowed me to implement and test individual components without having to plan every minigame from the start. The system is easily expandable; it could be modified with new minigames using the same collision mechanics, different maps, or even a multiplayer mode.