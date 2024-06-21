# All modules found within the main code



## Main code module


- Gather necessary game services such as the server storage where our enemies are stored in
- Set up initial game parameters I.E base health
- Game over event is turned on to be true when base health is 0 effectively ending the enemy spawning loop
- The main wave system consists of 10 waves
- Each wave progressively gets more difficult, spawning more enemies and different ones against the player
- Waits during each wave until the player loses or beats the wave
- If the wave is beaten it progresses to the next one while if the player loses it prints out a loser message in the console


## Base module

- Retrieves necessary game services such as the sever storage where towers are stored
- Has a "base" table that holds functions and data for it
- Initializes this base with a model from the workspace (What the player sees) with a set current and max health
- Updates the health according to what enemy health passes into it effectively dealing damage and changing the current health
- If reaches zero it triggers the "gameOver" event and displays a different text to indicate it
- Connects to the "base.UpdateHealth" function so the base health is updated whenever the event is fired


## Mob Module
