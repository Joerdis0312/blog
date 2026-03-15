---
title: Problems and Lessons Learned
summary: Here you can see which problems I had and how I dealt with them.
date: 2026-02-15
weight: 1
---

My first problem was that the graphic card of my laptop I used for creating this project was good enough to run unity without problems but not good enough to connect my VR-headset with the laptop via Meta Horizon Link. Therefor I had to build the whole project for every small change I did and install it to the headset with SideQuest. The main disadveantage was that I could't debug the game and didn't see the error messages which made it a lot more difficult to find some problems.

One problem that would be a lot easier to find with an error message was a null pointer exception while creating the card minigame. After pushing the one card I wanted to set this card to not active and place the white card at the same position. I had the problem that the white card was not at the exact same spot. Later I found that the problem was, that I turned the white card different than the others so that the writing is shown correctly to the player. But I didn't figure it out at the beginning and deleted the card and import it new to unity and into the scene, but I forgot to set it new in the inspector, so while pushing the card the card was deleted correctly but the new white card wasn't showing. I thought it was a problem with the positioning since my card was previously shown. It took me two days to figure it out. With an appropriate error message this would have been a lot easier. I learned if I include a new GameObject in any script the first thing to do for me was to go to the unity inspector and select the correct object for the script. I also learned to pay attention to the layer of the objects, since wo got the project with some layers and I didn't know what the differences were and hitting of the dice didn't work, since I used the wrong layer on the dice.

In my reaction minigame I had the problem, that the T-shape was not moving back and forth, but starting to fly in one directionand never came back. I missed to add the *-1.5f* after the ping-pong method, so it never gets a negative push to come back. After adding this I had the problem, that the shape had the wrong middle point. I fixed this by setting: *startPosition = TargetT.transform.position;* instead of *startPosition = transform.position;*.
So the T-shape was now oscillating correcty, but sometimes I hit the stop button accidentaly twice, because I hit to far and while taking back my hand I entered the collider a second time. This could easily be fixed, by extending the collider further to the back.

Another problem was that sometimes my game died while starting. It was random if the apk would start correct or not even without changing anything. The problem was due to a race condition and could be cured with manually loading XR with this script on an empty object at the top of the unity hierachy:

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

While uploading this blog to git I had the problem, that the longvideo was too big, so I had to compress it. But it also didn't work. After some time of searching I realised that the commit with the big video is still in the history and so github was still trying to upload it. So I had to revert this commit and add the video new to the repository.

**Conclusion**
I really like my idea since it is very easy to extend the game, and I could start with implementing some parts without planning all minigames in the beginning. This game can be modified in different ways, more/different minigames (with same collider-hitting mechanic or other mechanics), different maps or a multiplayer mode were the first things that came to my mind.