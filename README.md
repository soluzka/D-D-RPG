<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Eldoria's Legacy</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f4f4f4; }
        #game { margin: 20px; padding: 20px; border: 1px solid #ccc; background: white; }
        button { margin: 5px; }
    </style>
</head>
<body>
    <div id="game"></div>
    <script>
        // Game state variables
        let player = {
            name: '',
            classType: '',
            health: 100,
            inventory: [],
            experience: 0
        };

        let enemies = [
            { name: "Goblin", health: 30 },
            { name: "Wolf", health: 50 },
            { name: "Bandit", health: 60 }
        ];
        
        let battleInterval; // Timer for battles

        // Game start function
        function startGame() {
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h1>Welcome to Eldoria's Legacy</h1>
                                 <p>Enter your character's name:</p>
                                 <input type="text" id="playerName" />
                                 <button onclick="createCharacter()">Create Character</button>`;
        }

        // Function to create character
        function createCharacter() {
            const nameInput = document.getElementById('playerName').value;
            if (nameInput) {
                player.name = nameInput;
                chooseClass();
            } else {
                alert('Please enter a name!');
            }
        }

        // Function to choose character class
        function chooseClass() {
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>Choose Your Class</h2>
                                 <button onclick="setClass('Warrior')">Warrior</button>
                                 <button onclick="setClass('Mage')">Mage</button>
                                 <button onclick="setClass('Rogue')">Rogue</button>`;
        }

        // Set class and start the first quest
        function setClass(classType) {
            player.classType = classType;
            player.health += classType === 'Warrior' ? 20 : classType === 'Mage' ? 10 : 15;
            startQuest();
        }

        // Function for the first quest
        function startQuest() {
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>Your Adventure Begins!</h2>
                                 <p>As a ${player.classType} named ${player.name}, you enter the Whispering Grove. What will you do?</p>
                                 <button onclick="battle()">Engage in Battle</button>
                                 <button onclick="explore()">Explore</button>
                                 <button onclick="leaveGrove()">Leave the Grove</button>`;
        }

        // Engage in battle
        function battle() {
            const enemy = enemies[Math.floor(Math.random() * enemies.length)];
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>Battle with ${enemy.name}!</h2>
                                 <p>Enemy Health: ${enemy.health}</p>
                                 <p>Your Health: ${player.health}</p>
                                 <p>Time remaining: <span id="timer">15</span> seconds</p>
                                 <button onclick="attack(${enemy.health}, '${enemy.name}')">Attack</button>`;
            startBattleTimer();
        }

        // Start the battle countdown timer
        function startBattleTimer() {
            let timeLeft = 15;
            const timerDisplay = document.getElementById('timer');

            battleInterval = setInterval(() => {
                timeLeft--;
                timerDisplay.innerText = timeLeft;

                if (timeLeft <= 0) {
                    clearInterval(battleInterval);
                    loseBattle();
                }
            }, 1000);
        }

        // Handle attack logic
        function attack(enemyHealth, enemyName) {
            clearInterval(battleInterval);
            const damageToEnemy = Math.floor(Math.random() * 20) + 5; // Random damage between 5 and 25
            enemyHealth -= damageToEnemy;

            const gameDiv = document.getElementById('game');
            if (enemyHealth <= 0) {
                player.experience += 10;
                player.health += 15; // Health upgrade upon victory
                gameDiv.innerHTML = `<h2>You defeated the ${enemyName}!</h2>
                                     <p>You gained 10 experience points!</p>
                                     <p>Your health has been upgraded by 15!</p>
                                     <button onclick="startQuest()">Continue your adventure</button>`;
                return;
            } else {
                const playerDamage = Math.floor(Math.random() * 10) + 5; // Random damage between 5 and 15
                player.health -= playerDamage;

                if (player.health <= 0) {
                    // Game Over case if you lose health but allow to retry
                    gameDiv.innerHTML = `<h2>You have been defeated!</h2>
                                         <p>Your adventure continues, but you suffered injuries.</p>
                                         <p>You lost 20 health. Current health: ${player.health - 20}</p>`;
                    player.health -= 20;
                    setTimeout(startQuest, 3000); // Restart adventure after 3 seconds
                } else {
                    gameDiv.innerHTML = `<h2>Battle with ${enemyName}!</h2>
                                         <p>${enemyName} has ${enemyHealth} health left.</p>
                                         <p>You took ${playerDamage} damage. Your health is ${player.health}.</p>
                                         <button onclick="attack(${enemyHealth}, '${enemyName}')">Attack Again</button>`;
                    startBattleTimer(); // Restart timer for next attack
                }
            }
        }

        // Handle losing a battle
        function loseBattle() {
            clearInterval(battleInterval);
            player.health -= 20; // Lose health
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>The enemy fled!</h2>
                                 <p>You lost 20 health!</p>
                                 <p>Your current health is ${player.health}.</p>
                                 <button onclick="startQuest()">Continue your adventure</button>`;
        }

        // Explore option
        function explore() {
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>Exploration</h2>
                                 <p>You discover a hidden path that leads to a treasure chest!</p>
                                 <button onclick="findTreasure()">Open Treasure Chest</button>`;
        }

        function findTreasure() {
            const treasure = Math.floor(Math.random() * 50) + 10; // Random health restoration
            player.health += treasure;
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>Treasure Found!</h2>
                                 <p>You found a treasure that restored ${treasure} health!</p>
                                 <p>Your current health is ${player.health}.</p>
                                 <button onclick="startQuest()">Continue your adventure</button>`;
        }

        // Leave the grove
        function leaveGrove() {
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>You Leave the Grove</h2>
                                 <p>You decide it’s best to regroup with your allies. Your adventure continues!</p>
                                 <button onclick="startQuest()">Continue your adventure</button>`;
        }

        // Start the game on page load
        window.onload = startGame;
    </script>
</body>
</html>
