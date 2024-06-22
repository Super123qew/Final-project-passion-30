# All modules found within the code

## Main Script


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
- If it reaches zero it triggers the "gameOver" event and displays a different text to indicate it
- Connects to the "base.UpdateHealth" function so the base health is updated whenever the event is fired


## Mob Module

- Retrieves necessary game services
- Sets up event binding
- Moves a mob along waypoints that are defined on the map
- Once it reaches the final waypoint it is destroyed and its remaining health is subtracted from the base health through the event in the base module
- Spawning is achieved by cloning a mob from the "ServerStorage" and positioning it at the starting point on the map
- Collisions are turned off by assigning it a collision group (The player can't touch them during gameplay)
- Mobs are destroyed after death once defeated
- Prints a warning message if the mob type doesn't exist within the "ServerStorage"

  ## Tower Module

  - Retrieves necessary game services
  - Sets up event bindings for spawning towers
  - Tower finds the nearest enemy within the given range of a tower
  - If a target is found and is alive, the tower then turns toward the enemy and deals damage
  - Set wait cooldown is added to simulate how fast a tower should attack
  - If the target is killed, the player is awarded gold based on the target's max health
  - The function recursively repeats for continued attack
  - Spawning is done first by checking if a selected tower exists
  - If it does, clone it and set a position for it
  - Create a "BodyGyro" to facilitate the turning of troops toward enemies
  - Collisions are turned off by setting towers in a collision group that cant touch players
  - Start the tower's attack behavior using a coroutine
 
## GameController Local Script

- Retrieves necessary game services
- Sets up references for the player camera and GUI among other things
- Multiple variables are created here stating conditions for player I.E limiting 10 towers for the player
- Mouse ray cast is used for tower placement determining where it can be spawned
- A temporary tower is used as a placeholder for spawning (A semi-transparent "tower" that helps with tower placement)
- Removed once the actual tower is placed
- GUI setup is done here creating buttons allowing for separate troop types
- Inputs are handled such as with left click placing towers if in a valid position on the map
- The placeholder tower changes colour to red if the ray cast result is not part of the designated "Tower" areas.


## MobAnimation Local Script

- Handles animation for newly spawned mobs
- Waits until the mob to have a "humanoid" and "Animations" folder
- If present retrieves a "walk animation"
- Creates an "ANimation" object if it doesn't already exist in the "humanoid"
- Loads and plays the animation on the mob
- Is connected so that it ensures whenever a mob is spawned the function is called to animate it

  ##  DataStore script (WIP)

  - Retrieves necessary game services
  - 

  ## OnPLayerAdded Script

  - Retrieves necessary game services
  - 
