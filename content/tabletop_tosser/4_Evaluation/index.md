---
title: Evaluation and Results
subtitle: ""
summary: Analysis of the user study and performance metrics.
date: 2026-02-15
weight: 2
---
**Game Result**

{{< video src="all" type="video/mp4" caption="Here you can see me performing a whole run of the game." >}}

**User Study**

To evaluate the gameplay experience and technical implementation, I conducted a user study with a small group of participants (friends and family). Each participant was given a brief orientation period to familiarize themselves with the Meta Quest 2 and the game mechanics before completing a full run.

The study focused on two primary categories:

 1. Gameplay Mechanics: Ease of understanding, performance in skill-based minigames (reaction and T-shape), and overall entertainment value.
 2. VR Experience: Motion sickness, sense of presence, and physical/visual fatigue.

Notably, one participant with a history of vertigo experienced significant motion sickness after the first jump and was unable to continue.

**Part 1: Mechanics and Engagement**

As shown in the first figure, the game mechanics were easy to understand. However, the entertainment factor varied among participants. While the core mechanics were well-received, long-term motivation was limited by a lack of substantial rewards. While players could compare their completion time or the number of minigames played with previous runs, the high impact of randomness made direct competition difficult.

{{< figure src="images/part_1_1.png" caption="User study results regarding mechanics and engagement." >}}

A key objective was to ensure that skill-based minigames felt more rewarding than purely stochastic dice rolls (which have an expected value of 3.5). As illustrated below, both the reaction game and the T-shape interaction yielded higher average jump rewards, confirming that player effort effectively influences the outcome.

{{< figure src="images/part_1_2.png" caption="Average results of the two skill-based minigames." >}}

**Part 2: VR Experience**

Susceptibility to motion sickness varied significantly among players and largely aligned with their self-reported sensitivity prior to the test. Fatigue was primarily attributed to visual flickering or extended sessions. As the developer, I did not experience motion sickness even after multiple runs, though I did note significant fatigue after two or three consecutive games. While the sense of presence was rated as relatively good, the environment was not yet immersive enough to fully captivate all players.

{{< figure src="images/part_2.png" caption="Here are the results of part 2." >}}

**Feedback**

Participants provided the following constructive feedback:

 - The concept is solid, but would benefit from a higher proportion of skill-based minigames.
 - Increase motivation for replayability (e.g., global rankings for "fewest rolls needed" or detailed minigame statistics).
 - Dynamic height adjustment: Place interactive objects based on the user's headset height.
 - Scale adjustment: Some objects were perceived as too large.
 - UI/UX: Add an acoustic or visual countdown timer for the reaction game.
 - Visual Polish: Reduce the flickering within the virtual environment.