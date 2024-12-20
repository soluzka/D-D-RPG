<!DOCTYPE html>
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

        const enemies = [
            { name: "Goblin", health: 100 },
            { name: "Wolf", health: 100 },
            { name: "Bandit", health: 100 }
        ];

        const aiResponses = {
            introduction: "An introduction to your journey awaits...",
            combat: "What will be your next move in this fierce battle?",
            exploration: "As you explore, you might find treasures or traps!",
            treasures: "You discover a treasure chest hidden away in the woods."
        };

        // Expanded function to generate unique dynamic choices for quests
        function generateChoices() {
            const baseChoices = [
                "Engage in Battle",
                "Explore the surroundings",
                "Seek Treasures",
                "Consult the Oracle",
                "Inspect the surroundings",
                "Heal using a potion",
                "Attempt to negotiate",
                "Set a trap",
                "Scout the area",
                "Search for hidden doors",
                "Join a Guild",
                "Train in Skill",
                "Craft a Potion",
                "Visit a Tavern",
                "Engage in Diplomacy",
                "Discover Lore",
                "Summon a Familiar",
                "Challenge a Rival",
                "Participate in a Festival",
                "Manage Resources",
                "Scout Enemy Camps",
                "Map the Area",
                "Encourage Allies",
                "Steal an Item",
                "Perform a Ritual",
                "Bargain with Merchants",
                "Travel to Another Location",
                "Meditate for Clarity",
                "Use a Special Item",
                "Participate in a Contest",
                "Write in a Journal",
                "Train a Companion"
            ];
            return baseChoices;
        }

        // Chapters with stories and choices
        const chapters = Array.from({ length: 30 }, (_, i) => ({
            title: `Chapter ${i + 1}`,
            questDescription: `You are in Chapter ${i + 1}. What adventure lies ahead?`,
            choices: generateChoices() // Using the expanded generateChoices function
        }));

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
            const aiResponse = aiResponses[choice.toLowerCase().replace(/\s+/g, '')] || "The adventure continues...";

            switch (choiceIndex) {
                case 0: // Engage in Battle
                    battle();
                    break;
                case 1: // Explore
                    await explore();
                    break;
                case 2: // Seek Treasures
                    await findTreasure();
                    break;
                case 3: // Consult the Oracle
                    alert(`Oracle says: ${aiResponse}`);
                    break;
                case 10: // Join a Guild
                    alert("You've joined a guild! You gain new allies and resources.");
                    break;
                case 11: // Train in Skill
                    alert("You train and improve your skills! Gain 5 experience.");
                    player.experience += 5;
                    checkLevelUp();
                    break;
                case 12: // Craft a Potion
                    alert("You crafted a potion! It will restore 20 health.");
                    player.inventory.push("Health Potion");
                    break;
                case 13: // Visit a Tavern
                    alert("You visited a tavern and gathered local rumors.");
                    break;
                case 14: // Engage in Diplomacy
                    alert("You negotiated successfully! You avoided conflict.");
                    break;
                case 15: // Discover Lore
                    alert("You learned ancient lore about the land!");
                    break;
                case 16: // Summon a Familiar
                    alert("You summoned a familiar to assist you!");
                    break;
                case 17: // Challenge a Rival
                    alert("You challenged your rival to a duel!");
                    break;
                case 18: // Participate in a Festival
                    alert("You enjoyed a festival, receiving a minor boon!");
                    break;
                case 19: // Manage Resources
                    alert("You managed your resources effectively!");
                    break;
                case 20: // Scout Enemy Camps
                    alert("You scouted enemy camps, gaining valuable intel.");
                    break;
                case 21: // Map the Area
                    alert("You mapped the area, revealing new locations to explore.");
                    break;
                case 22: // Encourage Allies
                    alert("You encouraged your allies, boosting their morale!");
                    break;
                case 23: // Steal an Item
                    alert("You attempted to steal an item, but were caught!");
                    break;
                case 24: // Perform a Ritual
                    alert("You performed a ritual and felt a surge of power!");
                    break;
                case 25: // Bargain with Merchants
                    alert("You bargained with merchants and got great deals!");
                    break;
                case 26: // Travel to Another Location
                    alert("You traveled to another location, discovering new quests.");
                    break;
                case 27: // Meditate for Clarity
                    alert("You meditated and gained insight into your path.");
                    break;
                case 28: // Use a Special Item
                    alert("You used a special item; its effects are powerful!");
                    break;
                case 29: // Participate in a Contest
                    alert("You entered a contest and showcased your skills!");
                    break;
                case 30: // Write in a Journal
                    alert("You wrote in your journal, noting your adventures.");
                    break;
                case 31: // Train a Companion
                    alert("You trained a companion to fight alongside you.");
                    break;
                default:
                    alert(`You chose to: ${choice}. ${aiResponse}`);
                    break;
            }
        }

        // New function to check for level up
        function checkLevelUp() {
            if (player.experience >= player.experienceToLevelUp) {
                player.level++;
                player.experience -= player.experienceToLevelUp; // Carry over the excess experience
                player.experienceToLevelUp = Math.round(player.experienceToLevelUp * 1.5); // Increase next level's requirement (e.g., by 50%)
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

        function battle() {
            const enemyIndex = Math.floor(Math.random() * enemies.length);
            const enemy = enemies[enemyIndex];

            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>You are battling with a ${enemy.name}!</h2>
                                 <p>Enemy Health: ${enemy.health}</p>
                                 <p>Your Health: ${player.health}</p>
                                 <button onclick="attack(${enemy.health}, '${enemy.name}')">Attack!</button>`;
        }

        function attack(enemyHealth, enemyName) {
            const damageToEnemy = Math.floor(Math.random() * 20) + 5;
            enemyHealth -= damageToEnemy;

            const gameDiv = document.getElementById('game');

            if (enemyHealth <= 0) {
                player.experience += 10; // Give experience for defeating the enemy
                checkLevelUp(); // Check if the player levels up after gaining experience
                gameDiv.innerHTML = `<h2>You have defeated the ${enemyName}!</h2>
                                     <p>You gained 10 experience points!</p>
                                     <button onclick="nextChapter()">Continue your adventure</button>`;
            } else {
                const playerDamage = Math.floor(Math.random() * 20) + 5;
                player.health -= playerDamage;

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

        async function explore() {
            const treasureFound = Math.random() < 0.5; // 50% chance of finding treasure
            const gameDiv = document.getElementById('game');
            if (treasureFound) {
                gameDiv.innerHTML = `<h2>Exploration Success!</h2>
                                     <p>You discovered valuable treasures!</p>
                                     <p>Your adventure continues...</p>
                                     <button onclick="nextChapter()">Continue</button>`;
            } else {
                gameDiv.innerHTML = `<h2>Exploration Failed!</h2>
                                     <p>You ran into a trap!</p>
                                     <p>Your health is reduced by 10!</p>`;
                player.health -= 10;
            }
        }

        async function findTreasure() {
            const treasure = Math.floor(Math.random() * 50) + 10; 
            player.health += treasure;
            const gameDiv = document.getElementById('game');
            gameDiv.innerHTML = `<h2>Treasure Found!</h2>
                                 <p>You found treasure that restored ${treasure} health!</p>
                                 <p>Your current health is ${player.health}.</p>
                                 <button onclick="nextChapter()">Continue your adventure</button>`;
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
