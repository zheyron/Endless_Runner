# Endless_Runner_p5js
 Endless Runner created for p5js engine for my University Project

# Usage
You may use this code and or its assets openly for educational or commercial projects as long as you attribute me in credits. 

# Endless Runner - Cube Ascension

# Credits on Assets
* Code: Nicolas Martin Rodriguez
* Music: Nicolas Martin Rodriguez
* Background: downloaded background animation from a open ended library. You may not use this background asset on commercial projects as I was not the creator.

# Purpose
The purpose of this project is to develop an endless runner game with physics-based mechanics using the p5.js library. The game aims to provide an engaging experience where the player controls a character that can jump, glide, and change shape to avoid obstacles and enemies. The inclusion of bosses, projectiles, and increasing difficulty levels adds depth and challenge to the gameplay.

# Scope
This documentation covers the overall structure and functionality of the game code, providing insights into how various components interact within the game. It includes:
An overview of the global variables that manage game state and configurations.
Descriptions of the main functions that control game flow, rendering, and user input.
An outline of the classes used to represent game entities such as the player, enemies, boss, projectiles, and environmental elements like clouds.
Explanations of the game mechanics, including physics calculations, collision detection, and state management.
Note: Detailed explanations of each class, their properties, methods, and a UML diagram will be provided in subsequent sections.

# Script Overview
The game is structured using the p5.js framework and utilizes object-oriented programming concepts to organize code into classes and functions. Below is an overview of the script's main components:
## Global Variables
These variables maintain the game's state and configurations:
### Player and Entities:
* player: The main character controlled by the player.
* enemies: An array holding instances of Enemy objects.
* boss: An instance of the Boss class representing the boss enemy.
* projectiles: An array for storing projectiles fired by the boss.
* clouds: An array of CloudLayer objects for background visuals.
### Game State Management:
* gameOver: A boolean indicating if the game has ended.
* gameStarted: Tracks if the game has begun to manage initial timing.
score and highScore: Variables for tracking the player's score and the highest score achieved.
### Timers and Intervals:
* bossCooldown and bossSpawnDelay: Manage the timing for boss appearances.
* enemySpawnInterval and lastEnemySpawnTime: Control the spawning rate of regular enemies.
* startTime: Records the game's start time for score calculation.
### User Interface Elements:
* playAgainButton: A button to restart the game after a game over.
### Audio and Visual Assets:
* bgMusic: The background music object.
* musicStarted: Ensures music starts only once.
* bgImage: The background image for the game.
## Preload Function
```javascript
function preload() {
  bgMusic = loadSound('Ascendancy of the Cube.mp3');
  bgImage = loadImage('synthwave_sunset.gif');
}
```
* Loads the background music and image before the game starts.
## Setup Function
```javascript
function setup() {
  createCanvas(800, 400);
  player = new Player();
  for (let i = 0; i < 3; i++) {
    clouds.push(new CloudLayer(i + 1));
  }
  playAgainButton = createButton('Play Again');
  playAgainButton.position(width / 2 - 50, height / 2 + 30);
  playAgainButton.mousePressed(restartGame);
  playAgainButton.hide();
  bgMusic.setVolume(0.5);
}
```
* Initializes the game canvas.
* Creates the player object.
* Generates cloud layers for the background.
* Sets up the "Play Again" button but keeps it hidden initially.
* Adjusts the background music volume.

## Draw Function
```javascript
function draw() {
  // Display the background image
  image(bgImage, 0, 0, width, height);

  let deltaTimeSec = deltaTime / 1000; // Convert deltaTime to seconds

  for (let cloudLayer of clouds) {
    cloudLayer.update(deltaTimeSec);
    cloudLayer.show();
  }

  if (!gameOver) {
    if (!gameStarted) {
      startTime = millis();
      gameStarted = true;
    }
    player.update(deltaTimeSec);
    player.show();
    handleEnemies(deltaTimeSec);
    handleBoss(deltaTimeSec);
    handleProjectiles(deltaTimeSec);
    displayScore();
  } else {
    displayGameOver();
  }
}
```

* Renders the background image each frame.
* Calculates deltaTimeSec for consistent movement across different frame rates.
* Updates and displays cloud layers.
* Manages game flow based on gameOver and gameStarted states.
* Updates and renders the player, enemies, boss, and projectiles.
* Displays the current score or game over screen.

## Input Handling Functions
### Key Pressed
```javascript
function keyPressed() {
  if (!musicStarted) {
    bgMusic.loop();
    musicStarted = true;
  }
  if (!gameOver) {
    if (keyCode === CONTROL) {
      player.fallFast();
      player.stopGlide();
    } else if (keyCode === 32) { // Space key
      if (player.onGround()) {
        player.jump();
      } else {
        player.startGlide();
      }
    } else {
      player.jump(); // Alternative jump with any other key
    }
  }
}
```

* Starts background music on the first key press.
* Allows the player to jump, glide, or fall faster based on the key pressed.
* Uses the Control key to initiate a fast fall.
* Uses the Space key for jumping and gliding mechanics.
### Key Released
```javascript
function keyReleased() {
  if (keyCode === 32) { // Space key
    player.stopGlide();
  }
}
```
* Stops the gliding action when the Space key is released.
### Mouse Pressed
```javascript
function mousePressed() {
  if (!musicStarted) {
    bgMusic.loop();
    musicStarted = true;
  }
  if (!gameOver && player.contains(mouseX, mouseY)) {
    player.changeShape();
  }
}
```
* Starts background music on the first mouse click.
* Allows the player to change shape (from square to circle) when the character is clicked.
## Game Logic Functions
### Handling Enemies
```javascript
function handleEnemies(deltaTimeSec) {
  let adjustedEnemySpawnInterval = max(500, enemySpawnInterval - score * 8.33);

  if (boss === null && millis() - lastEnemySpawnTime > adjustedEnemySpawnInterval) {
    let enemyType = random(['ground', 'flying']);
    enemies.push(new Enemy(enemyType, score));
    lastEnemySpawnTime = millis();
  }

  for (let i = enemies.length - 1; i >= 0; i--) {
    let enemy = enemies[i];
    enemy.update(deltaTimeSec);
    enemy.show();

    if (player.collidesWith(enemy)) {
      gameOver = true;
      if (score > highScore) {
        highScore = score;
      }
      playAgainButton.show();
    }

    if (enemy.offscreen()) {
      enemies.splice(i, 1);
    }
  }

  score = floor((millis() - startTime) / 1000);
}
```
* Adjusts the enemy spawn interval based on the player's score to increase difficulty over time.
* Spawns new enemies if the boss is not active.
* Updates and displays each enemy.
* Checks for collisions between the player and enemies.
* Removes offscreen enemies.
* Updates the player's score.
### Handling the Boss
```javascript
function handleBoss(deltaTimeSec) {
  if (boss === null && bossCooldown <= 0 && score >= 15) {
    bossCount++;
    boss = new Boss(bossCount);
  }

  if (boss !== null) {
    boss.update(deltaTimeSec);
    boss.show();

    if (boss.isDestroyed) {
      boss = null;
      bossCooldown = bossSpawnDelay;
    }
  } else if (bossCooldown > 0) {
    bossCooldown -= deltaTimeSec * 1000;
  }
}
```
* Spawns a new boss after a certain score is reached and cooldown has passed.
* Updates and displays the boss.
* Handles the boss's destruction and cooldown for the next spawn.
### Handling Projectiles
```javascript
function handleProjectiles(deltaTimeSec) {
  for (let i = projectiles.length - 1; i >= 0; i--) {
    let proj = projectiles[i];
    proj.update(deltaTimeSec);
    proj.show();

    if (player.collidesWithProjectile(proj)) {
      gameOver = true;
      if (score > highScore) {
        highScore = score;
      }
      playAgainButton.show();
    }

    if (proj.offscreen()) {
      projectiles.splice(i, 1);
    }
  }
}
```

* Updates and displays each projectile fired by the boss.
* Checks for collisions between the player and projectiles.
* Removes offscreen projectiles.

## Display Functions
### Displaying Score
```javascript
function displayScore() {
  fill(250, 250, 0);
  textSize(16);
  textAlign(LEFT);
  text('Score: ' + score, 10, 20);
  textAlign(RIGHT);
  text('High Score: ' + highScore, width - 10, 20);
}
```

### Displaying Game Over Screen
```javascript
function displayGameOver() {
  fill(255, 0, 0);
  textSize(48);
  textAlign(CENTER);
  text('Game Over', width / 2, height / 2);
}
```

* Displays a "Game Over" message when the player loses.
## Game Restart Function
```javascript
function restartGame() {
  gameOver = false;
  gameStarted = false;
  enemies = [];
  projectiles = [];
  boss = null;
  bossCount = 0;
  player.reset();
  playAgainButton.hide();
}
```

* Resets all game variables and states to their initial values.
* Hides the "Play Again" button.

## Player Class
### Purpose
* The Player class represents the main character controlled by the player. It handles the player's position, movement mechanics (including jumping and gliding), shape-shifting abilities, collision detection with enemies and projectiles, and rendering on the canvas.
### Properties
* pos: A p5.Vector representing the player's position on the canvas.
* size: The size of the player when in square form.
* circleSize: The size of the player when transformed into a circle.
* vel: A p5.Vector representing the player's velocity.
* gravity: A p5.Vector representing the gravity force applied to the player.
* lift: A numeric value representing the upward force when the player jumps.
* isCircle: A boolean indicating whether the player is currently in circle form.
* shapeChangeTime: Stores the timestamp when the player last changed shape.
* maxFallSpeed: The maximum speed at which the player can fall.
* isGliding: A boolean indicating whether the player is currently gliding.
* glideGravity: A p5.Vector representing reduced gravity when the player is gliding.
### Methods
* constructor(): Initializes the player's properties.
* update(dt): Updates the player's position and handles physics calculations like gravity and velocity.
* show(): Renders the player on the canvas, including drawing wings if gliding.
* drawWings(): Draws wings on the player when gliding.
* jump(): Initiates the player's jump by applying an upward velocity.
* fallFast(): Increases the player's fall speed to the maximum fall speed.
* startGlide(): Activates the gliding state, reducing gravity's effect.
* stopGlide(): Deactivates the gliding state.
* onGround(): Checks if the player is touching the ground.
* changeShape(): Transforms the player into a circle for a limited time.
* collidesWith(enemy): Checks collision between the player and an enemy.
* collidesWithProjectile(proj): Checks collision between the player and a projectile.
* contains(px, py): Determines if a given point is within the player's bounds (used for mouse interactions).
* reset(): Resets the player's properties to their initial state upon game restart.
### Interactions and Coupling
* With Enemies: Uses collidesWith(enemy) to detect collisions, relying on methods from the Enemy class (collidesWithRect and collidesWithCircle).
* With Projectiles: Uses collidesWithProjectile(proj) to detect collisions with projectiles (Projectile and Projectile360 instances).
* With Game Logic Manager: The player is updated and rendered in the main draw() loop, and responds to user inputs captured in keyPressed(), keyReleased(), and mousePressed() functions.

## Enemy Class
### Purpose
* The Enemy class represents obstacles that the player must avoid. Enemies can be ground-based or flying, and can have different shapes (circle or square). The class handles enemy movement, rendering, and collision detection with the player.
### Properties
* type: A string indicating the enemy type ('ground' or 'flying').
* size: The size of the enemy, randomized within a range.
* pos: A p5.Vector representing the enemy's position on the canvas.
* vel: A p5.Vector representing the enemy's velocity.
* isCircle: A boolean indicating whether the enemy is a circle or a square.
#### Additional properties for flying enemies:
* angle: Used for calculating sinusoidal movement.
* amplitude: The amplitude of the sinusoidal movement.
* angularSpeed: The speed of the sinusoidal oscillation.
### Methods
* constructor(type, score): Initializes the enemy's properties based on its type and the player's score.
* update(dt): Updates the enemy's position, including sinusoidal movement for flying enemies.
* show(): Renders the enemy on the canvas.
* offscreen(): Checks if the enemy has moved off the left edge of the screen.
* collidesWithRect(rx, ry, rw, rh): Checks collision with a rectangle (used when the player is in square form).
* collidesWithCircle(cPos, cR): Checks collision with a circle (used when the player is in circle form).
### Interactions and Coupling
* With Player: Collision detection methods collidesWithRect and collidesWithCircle are used by the player's collidesWith(enemy) method.
* With Game Logic Manager: Enemies are managed in the enemies array, updated and rendered in handleEnemies(deltaTimeSec), and removed when offscreen or upon collision with the player.
## Boss Class
### Purpose
* The Boss class represents a challenging enemy that appears periodically. It has multiple states (entering, spinning, shooting, resting, finalShoot, selfDestruct) and can fire projectiles at the player, including a special 360-degree attack.
### Properties
* size: The size of the boss.
* pos: A p5.Vector representing the boss's position.
* vel: A p5.Vector representing the boss's velocity.
* angle: The current rotation angle of the boss.
* angularVelocity: The rotational speed.
* angularAcceleration: The rate at which the rotational speed changes.
* spinningClockwise: A boolean indicating the spin direction.
* spinTime: Timer for managing spin duration.
* state: A string representing the boss's current state.
* shootTimer: Timer for controlling shooting intervals.
* shootDuration: Tracks the duration of the shooting state.
* restDuration: Tracks the duration of the resting state.
* isDestroyed: A boolean indicating if the boss should be removed.
* shotsFired: Counts the number of shots fired in the current attack cycle.
* timeBetweenShots: The interval between shots during the shooting state.
* maxShots: The maximum number of shots in a shooting cycle, increasing with each boss encounter.
* attackCycles: The number of completed attack cycles.
* maxCycles: The total number of attack cycles before the boss performs the final attack and self-destructs.
### Methods
* constructor(bossCount): Initializes the boss's properties, adjusting difficulty based on the number of bosses encountered.
* update(dt): Manages the boss's behavior based on its current state, including movement, rotation, shooting, and state transitions.
* show(): Renders the boss on the canvas.
* shoot(): Fires a projectile towards the player's position with a slight random offset.
* shoot360(): Fires multiple projectiles in all directions for the final attack.
* selfDestruct(): Sets the boss's state to self-destruct, marking it for removal.
### Interactions and Coupling
* With Projectiles: Uses the shoot() and shoot360() methods to create instances of Projectile and Projectile360, which are added to the projectiles array managed by the game logic.
* With Game Logic Manager: The boss is managed by the boss variable, updated and rendered in handleBoss(deltaTimeSec), and its lifecycle is controlled based on game events.
## Projectile Class
### Purpose
* The Projectile class represents projectiles fired by the boss towards the player. They can be of type 'square' or 'triangle' and move towards the player's position with a slight random offset to create unpredictability.
### Properties
* pos: A p5.Vector representing the projectile's position.
* type: A string indicating the projectile's type ('square' or 'triangle').
* size: The size of the projectile.
* vel: A p5.Vector representing the projectile's velocity, directed towards the player's position with an angle offset.
### Methods
* constructor(pos, type, angleOffset): Initializes the projectile's properties, calculating velocity towards the player with a random angle offset.
* update(dt): Updates the projectile's position based on its velocity.
* show(): Renders the projectile on the canvas, drawing it as a square or triangle based on its type.
* offscreen(): Checks if the projectile has moved off the screen boundaries.
### Interactions and Coupling
* With Boss: Instances are created by the boss's shoot() method.
* With Player: Collision detection is handled in the player's collidesWithProjectile(proj) method.
* With Game Logic Manager: Projectiles are managed in the projectiles array, updated and rendered in handleProjectiles(deltaTimeSec), and removed when offscreen or upon collision with the player.
## Projectile360 Class
### Purpose
* The Projectile360 class represents special projectiles fired by the boss in a 360-degree pattern during its final attack. They move outward in all directions from the boss's position.
### Properties
* pos: A p5.Vector representing the projectile's position.
* angle: The direction of the projectile's movement in radians.
* size: The size of the projectile.
* vel: A p5.Vector representing the projectile's velocity, calculated based on the angle.
### Methods
* constructor(pos, angle): Initializes the projectile's properties, setting velocity based on the given angle.
* update(dt): Updates the projectile's position.
* show(): Renders the projectile on the canvas as a circle.
* offscreen(): Checks if the projectile has moved off the screen boundaries.
### Interactions and Coupling
* With Boss: Instances are created by the boss's shoot360() method during the final attack.
* With Player: Collision detection is handled in the player's collidesWithProjectile(proj) method.
* With Game Logic Manager: Managed alongside other projectiles in the projectiles array.
## Interactions and Coupling with Game Logic Manager
* The Game Logic Manager refers to the main script functions that control the game's flow, including setup(), draw(), and various handler functions.
### Key Interactions
#### Player:
* Updated and rendered each frame in the draw() function.
* Responds to user inputs via keyPressed(), keyReleased(), and mousePressed().
* Collision checks with enemies and projectiles are performed in handleEnemies() and handleProjectiles().
#### Enemies:
* Managed in the enemies array.
* Spawned in handleEnemies() based on timers and game state.
* Each enemy is updated and rendered in handleEnemies().
* Collision with the player triggers a game over.
#### Boss:
* Managed via the boss variable.
* Spawned in handleBoss() after certain conditions are met (score threshold and cooldown).
* Updated and rendered in handleBoss().
* Shoots projectiles that interact with the player.
#### Projectiles:
* Managed in the projectiles array.
* Spawned by the boss's shoot() and shoot360() methods.
* Each projectile is updated and rendered in handleProjectiles().
* Collision with the player triggers a game over.
### Coupling
* Data Coupling: Classes interact by sharing data through parameters and shared variables (e.g., player position, projectiles array).
* Control Coupling: The game logic controls the flow of the game by invoking methods on instances of these classes based on game states and events.
* Temporal Coupling: Certain actions depend on specific timing (e.g., enemy spawning intervals, boss attack cycles).

## UML Diagram
![Diagrama UML ver1 - Fuenteki](https://github.com/user-attachments/assets/479f1b0a-6320-4a16-8db4-e299feca76af)
