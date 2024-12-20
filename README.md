**<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Eldoria's Legacy: AI Adventure</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            height: 100vh;
            overflow: hidden; /* Prevent scrolling */
        }

        #game {
            display: flex;
            flex-direction: column;
            justify-content: flex-start;
            align-items: center;
            padding: 20px;
            border: 1px solid #ccc;
            background: white;
            height: 100%;
            width: 100%;
        }

        .chatbox {
            margin: 10px 0;
            max-height: 150px;
            overflow-y: auto;
            width: 100%;
            background-color: #f9f9f9;
            border: 1px solid #ccc;
            padding: 10px;
            border-radius: 5px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }

        input[type="text"] {
            width: calc(100% - 22px);
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
            margin-bottom: 10px;
        }

        button {
            padding: 10px 15px;
            font-size: 16px;
            margin: 5px;
            cursor: pointer;
        }

        .choices {
            display: flex;
            flex-wrap: wrap; /* Allow wrapping */
            justify-content: center; /* Center items */
            margin: 10px 0; /* Add margin around choices */
        }

        .choice-button {
            margin: 5px; /* Space between buttons */
        }
    </style>
</head>

<body>
    <div id="game">
        <div id="chatbox" class="chatbox"></div>
        <input type="text" id="userInput" placeholder="Type your message..." />
        <button onclick="sendMessage()">Send</button>
        <button onclick="saveGame()">Save Game</button>
        <button onclick="loadGame()">Load Game</button>
    </div>

    <script>
        // Game state variables
        let player = {
            name: '',
            classType: '',
            health: 100,
            inventory: [],
            experience: 0,
            chapter: 1,
            characterImage: '',
            abilities: ["Hit", "Dodge", "Cast Spell"],
            level: 1,
            experienceToLevelUp: 100 // Example value for leveling up
        };

        // Modified enemies to reflect lore
        const enemies = [
            { name: "Goblin", health: 100 },
            { name: "Wolf", health: 100 },
            { name: "Bandit", health: 100 }
        ];


        // Expanded chapter lore and choices
        const chapters = Array.from({ length: 30 }, (_, i) => ({
            title: `Chapter ${i + 1}`,
            questDescription: `You are in Chapter ${i + 1}. What adventure lies ahead? The land is full of danger and wonders!`,
            choices: generateChoices() // Using the expanded generateChoices function
        }));

        function generateChoices() {
            const baseChoices = [
                "Engage in Battle",
                "Explore the surroundings",
                "Seek Treasures",
                "Consult the Oracle",
                "Inspect the surroundings",
                "Heal using a potion",
                "Attempt to negotiate"
            ];
            return baseChoices;
        }

        // Function for rolling dice
        function rollDice(sides) {
            return Math.floor(Math.random() * sides) + 1;
        }

        function startGame() {
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h1>Welcome to Eldoria's Legacy</h1>
                                 <p>Enter your character's name:</p>
                                 <input type="text" id="playerName" />
                                 <button onclick="createCharacter()">Create Character</button>`;
        }

        async function createCharacter() {
            const nameInput = document.getElementById('playerName').value;
            if (nameInput) {
                player.name = nameInput;
                await chooseClass();
            } else {
                alert('Please enter a name!');
            }
        }

        async function chooseClass() {
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>Choose Your Class</h2>
                                 <button onclick="setClass('Warrior')">Warrior</button>
                                 <button onclick="setClass('Mage')">Mage</button>
                                 <button onclick="setClass('Rogue')">Rogue</button>`;
        }

        async function setClass(classType) {
            player.classType = classType;

            // Class benefits
            if (classType === 'Warrior') {
                player.health += 20;
            } else if (classType === 'Mage') {
                player.health += 10;
                player.abilities.push("Fireball");
            } else {
                player.health += 15;
                player.abilities.push("Stealth");
            }

            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML += `<p>You have chosen the ${classType} class!</p>`;
            gameDiv.innerHTML += `<p>Your abilities include: ${player.abilities.join(', ')}</p>
                                  <button onclick="startAIAdventure()">Play</button>`;
        }

        async function startAIAdventure() {
            startChapter();
        }

        function startChapter() {
            const chapter = chapters[player.chapter - 1];
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>${chapter.title}</h2>
                                 <p>${chapter.questDescription}</p>
                                 <h3>Choices:</h3>
                                 <div class="choices">
                                     ${chapter.choices.map((choice, i) => `<button class="choice-button" onclick="handleChoice(${i})">${choice}</button>`).join('')}
                                 </div>`;
            displayPlayerInfo();
        }

        function displayPlayerInfo() {
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML += `<h3>Your Info:</h3>
                                  <p>Name: ${player.name}</p>
                                  <p>Class: ${player.classType}</p>
                                  <p>Health: ${player.health}</p>
                                  <p>Experience: ${player.experience} / ${player.experienceToLevelUp}</p>
                                  <p>Level: ${player.level}</p>
                                  <p>Abilities: ${player.abilities.join(', ')}</p>`;
        }

        async function handleChoice(choiceIndex) {
            const chapter = chapters[player.chapter - 1];
            const choice = chapter.choices[choiceIndex];
            const aiResponse = "The adventure continues...";
            const diceRoll = rollDice(6); // Roll a 6-sided die for decision-making

            switch (choiceIndex) {
                case 0: // Engage in Battle
                    await battle(diceRoll);
                    break;
                case 1: // Explore
                    await explore(diceRoll);
                    break;
                case 2: // Seek Treasures
                    await findTreasure(diceRoll);
                    break;
                case 3: // Consult the Oracle
                    alert(`Oracle says: ${aiResponse}`);
                    break;
                case 6:
                    alert("You attempted to negotiate. The outcome is uncertain...");
                    break;
                default:
                    alert(`You chose to: ${choice}. ${aiResponse}`);
                    break;
            }
        }

        async function battle(diceRoll) {
            const enemyIndex = Math.floor(Math.random() * enemies.length);
            const enemy = enemies[enemyIndex];
            const gameDiv = document.getElementById('game');

            if (diceRoll <= 3) {
                // Unlucky roll, enemy strikes first
                const damageToPlayer = rollDice(10);
                player.health -= damageToPlayer;
                gameDiv.innerHTML = `<h2>Oh no! The ${enemy.name} attacked first!</h2>
                                     <p>You took ${damageToPlayer} damage!</p>
                                     <p>Your Health: ${player.health}</p>
                                     <button onclick="attack(${enemy.health}, '${enemy.name}')">Counterattack!</button>`;
            } else {
                // Player strikes first
                gameDiv.innerHTML = `<h2>You encounter a ${enemy.name}!</h2>
                                     <p>Prepare to fight!</p>
                                     <button onclick="attack(${enemy.health}, '${enemy.name}')">Attack!</button>`;
            }
        }

        function attack(enemyHealth, enemyName) {
            const damageToEnemy = rollDice(12); // Player damage
            enemyHealth -= damageToEnemy;

            const playerDamage = rollDice(10); // Damage to player from enemy's retaliation
            player.health -= playerDamage;

            const gameDiv = document.getElementById('game');

            if (enemyHealth <= 0) {
                player.experience += 10; // Earn experience for winning a battle
                checkLevelUp(); // Check level up after gaining experience
                gameDiv.innerHTML = `<h2>You have defeated the ${enemyName}!</h2>
                                     <p>You gained 10 experience points!</p>
                                     <button onclick="nextChapter()">Continue your adventure</button>`;
            } else {
                if (player.health <= 0) {
                    gameDiv.innerHTML = `<h2>You have been defeated!</h2>
                                         <p>Your adventure has come to an end!</p>`;
                } else {
                    gameDiv.innerHTML = `<h2>${enemyName} has ${enemyHealth} health left!</h2>
                                         <p>You took ${playerDamage} damage; your health is now ${player.health}.</p>
                                         <button onclick="attack(${enemyHealth}, '${enemyName}')">Attack Again</button>`;
                }
            }
        }

        async function explore(diceRoll) {
            const gameDiv = document.getElementById('game');

            if (diceRoll <= 2) {
                // A poor exploration
                const damageTaken = rollDice(15);
                player.health -= damageTaken;
                alert(`You stumbled upon a hidden trap and took ${damageTaken} damage!`);

                if (player.health <= 0) {
                    gameDiv.innerHTML = `<h2>You have fallen!</h2>
                                         <p>Your adventures come to an end! Rest in peace!</p>`;
                } else {
                    gameDiv.innerHTML += `<h2>Unfortunate News!</h2>
                                         <p>Your health is now ${player.health}.</p>`;
                }
            } else if (diceRoll <= 4) {
                // Lucky exploration
                player.experience += 5;
                checkLevelUp(); // Check if leveled up
                alert("You successfully explored the area and gained valuable insights! You earned 5 experience points!");
            } else {
                // Very lucky outcome
                player.experience += 15;
                checkLevelUp(); // Check if leveled up
                alert("You found a hidden grove filled with precious herbs! You gained 15 experience points!");
            }
        }

        async function findTreasure(diceRoll) {
            const treasureAmount = rollDice(50) + 10; // Random amount of treasure
            player.health += treasureAmount;

            if (diceRoll <= 2) {
                alert("You found a treasure chest, but it was empty!");
            } else if (diceRoll <= 5) {
                player.experience += 15; // Experience for finding treasure
                checkLevelUp(); // Check level up
                alert(`You found a treasure chest! You restored ${treasureAmount} health and gained 15 experience points!`);
            } else {
                player.experience += 30; // Large treasure reward
                checkLevelUp(); // Check level up
                alert(`You discovered a legendary treasure hoard! You regained ${treasureAmount} health and received 30 experience points!`);
            }
        }

        // New function to check for level up
        function checkLevelUp() {
            if (player.experience >= player.experienceToLevelUp) {
                player.level++;
                player.experience -= player.experienceToLevelUp; // Carry over the excess experience
                player.experienceToLevelUp = Math.round(player.experienceToLevelUp * 1.5); // Increase next level's requirement
                player.health += 10; // Give bonus health on level up
                displayPlayerInfo(); // Refresh player info to show new stats
                alert(`Congratulations! You've leveled up to Level ${player.level}! Your health has increased!`);
            }
        }

        async function sendMessage() {
            const userInput = document.getElementById('userInput').value;
            const chatbox = document.getElementById('chatbox');

            if (!userInput) return; // Avoid sending empty messages

            chatbox.innerHTML += `<div><strong>User:</strong> ${userInput}</div>`;

            // AI interaction can be implemented here
            // For the example, I am just echoing back the user message
            chatbox.innerHTML += `<div><strong>AI:</strong> You said: ${userInput}</div>`;

            document.getElementById('userInput').value = ''; // Clear input
        }

        function nextChapter() {
            player.chapter = (player.chapter % chapters.length) + 1;
            startChapter(); // Proceed to next chapter
        }

        function saveGame() {
            const playerData = JSON.stringify(player);
            localStorage.setItem('eldoriaGame', playerData);
            alert('Game saved!');
        }

        function loadGame() {
            const savedData = localStorage.getItem('eldoriaGame');
            if (savedData) {
                Object.assign(player, JSON.parse(savedData));
                startChapter();
            } else {
                alert('No saved game found!');
            }
        }

        // Start the game on page load
        window.onload = startGame;
    </script>
</body>

</html>
**
