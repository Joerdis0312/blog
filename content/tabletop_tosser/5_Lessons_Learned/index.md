---
title: Technical Challenges and Lessons Learned
summary: An analysis of the technical obstacles encountered during development and the corresponding solutions.
date: 2026-02-15
weight: 1
---

**Hardware Constraints and Debugging**

The primary challenge during development was a hardware limitation: My laptop's GPU was insufficient to support Meta Horizon Link. Therefor I had to build the whole project for every small change I made and install it on the headset with SideQuest. The main disadvantage was that I could't debug the game and didn't see the error messages, which made it a lot more difficult to find some problems.

**Logic Errors and System Initialization**

One problem that would be a lot easier to find with an error message was a null pointer exception while creating the card minigame. After pushing the one card, I wanted to deactivate this card and place the white card at the same position. I had the problem that the white card was not in the same spot. Later, I found that the problem was that I turned the white card different than the others so that the writing is shown correctly to the player. But I didn't figure it out at the beginning and deleted the card and imported it new to Unity and into the scene, but I forgot to set it again in the inspector, so while pushing, the card backside was deleted correctly, but the new white card wasn't showing. I thought it was a problem with the positioning since my card was previously shown. It took me two days to figure it out. With an appropriate error message, this would have been a lot easier. I learned that if I include a new GameObject in any script, the first thing to do for me was to go to the Unity inspector and select the correct object for the script. I also learned to pay attention to the layers of the objects, since we got the project with some layers, and I didn't know what the differences were, and hitting the dice didn't work as I wished, since I used the wrong layer on the dice.

Another problem was that sometimes my game failed during startup. It was random if the .apk would start correctly or not, even without changing anything. The problem was due to a race condition and could be cured with manually loading XR with this script (given by AI) on an empty object at the top of the Unity hierachy:

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

I guess that the problem occured due to the fact that I placed the player at the position of the first field. And sometimes the field would be initialised first and sometimes the player which then had no correct initial position.

**Minigame Refinement**

In the reaction minigame I first had the problem, that the number of jumps was computed two times, after the ten seconds, which leads to a one each time. The first calculation, sets the number of hit cubes to zero and the second sees it and since the player gets at least one jump, the player got one jump each time. The problem was that the script runs on both controllers, so I included a check if the current controller is the left one, and only then I computed the number of jumps.

In my T-shape interaction minigame, I had the problem that the T-shape was not moving back and forth, but starting to fly in one directionand never came back. I missed adding the *-1.5f* after the ping-pong method, so it never gets a negative push to come back. After adding this, I had the problem that the shape had the wrong middle point. I fixed this by setting: *startPosition = TargetT.transform.position;* instead of *startPosition = transform.position;*.

So the T-shape was now oscillating correctly, but sometimes I hit the stop button accidentally twice, because I hit too far, and while taking my hand back, I entered the collider a second time. This could easily be fixed by extending the collider further to the back.

**Version Control Challenges**

During the deployment of this blog, a large video file exceeded GitHub's upload limits. Even after local compression, the large file remained in the Git history, causing subsequent push attempts to fail. Resolving this required a *git revert* of the specific commit to clean the history before re-adding the compressed version.

**Conclusion**

I really like my idea since it is very easy to extend the game, and I could start by implementing some parts without planning all the minigames in the beginning. This game can be modified in different ways, more/different minigames (with the same collider-hitting mechanic or other mechanics), different maps, or a multiplayer mode were the first things that came to my mind.