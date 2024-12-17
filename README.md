import tkinter as tk
from tkinter import messagebox
import random
import json
import os

class AdventureGame:
    def __init__(self, root):
        self.root = root
        self.root.title("Dungeons & Dragons Adventure Game")

        # Game state
        self.level = 1
        self.experience = 0
        self.boss_battles = 0
        self.health = 10
        self.items = []
        self.character_class = None
        self.chapter_index = 0  # Current chapter index
        self.dark_item_collected = False
        self.paths_taken = []
        
        # Initialize the chapters
        self.chapters = self.generate_chapters()

        # Text display and buttons setup
        self.text_display = tk.Text(root, height=15, width=50)
        self.text_display.pack()

        self.option_frame = tk.Frame(root)
        self.option_frame.pack()

        self.option_buttons = []
        for i in range(3):
            btn = tk.Button(self.option_frame, text=f"Option {i + 1}", command=lambda i=i: self.make_decision(i))
            btn.grid(row=0, column=i)
            self.option_buttons.append(btn)

        tk.Button(root, text="Previous Chapter", command=self.previous_chapter).pack(side=tk.LEFT)
        tk.Button(root, text="Next Chapter", command=self.next_chapter).pack(side=tk.RIGHT)
        tk.Button(root, text="Save Game", command=self.save_game).pack(side=tk.LEFT)
        tk.Button(root, text="Load Game", command=self.load_game).pack(side=tk.RIGHT)
        tk.Button(root, text="Roll Dice", command=self.roll_dice).pack(side=tk.BOTTOM)

        self.start_game()

    def generate_chapters(self):
        return [
            # Existing Chapters 1-100
            {
                "text": "You awake in a small village near the kingdom of Eldoria—a land renowned for its peace and prosperity. "
                        "But rumors swirl of a dark power rising in the East. Will you answer the call for adventure?",
                "options": ["Venture into the Whispering Woods", "Explore the bustling village", "Investigate the mysterious cave"],
                "next": [1, 2, 3]
            },
            {
                "text": "You find yourself at a crossroads in the Whispering Woods, listening to the whispering winds.",
                "options": ["Follow the path to the east", "Head back to the village", "Climb a nearby tree for a view"],
                "next": [4, 2, 5]
            },
            {
                "text": "The village is alive with music and storytellers. A mysterious bard catches your attention.",
                "options": ["Ask the bard for tales", "Shop for supplies", "Rest at the inn"],
                "next": [6, 7, 8]
            },
            {
                "text": "In the cave, the shadows dance as you explore deeper. A sudden rumble shakes the ground.",
                "options": ["Investigate the noise", "Retreat cautiously", "Draw your weapon"],
                "next": [9, 3, 10]
            },
            {
                "text": "Following the path, you encounter a strange figure shrouded in mist.",
                "options": ["Approach cautiously", "Draw your weapon", "Turn and run"],
                "next": [11, 12, 2]
            },
            {
                "text": "You reach a clearing with a magical fountain. Water sparkles with an otherworldly glow.",
                "options": ["Drink from the fountain", "Examine the area", "Leave the fountain alone"],
                "next": [13, 14, 2]
            },
            {
                "text": "The bard tells tales of a lost artifact said to grant immense power.",
                "options": ["Ask for directions", "Plan a quest to find it", "Thank him and leave"],
                "next": [15, 16, 2]
            },
            {
                "text": "You find a dusty old tome in the inn. It seems to whisper secrets.",
                "options": ["Read the tome", "Leave it be", "Show it to the innkeeper"],
                "next": [17, 8, 2]
            },
            {
                "text": "As you approach the rumbling noise, you stumble upon a cave-in.",
                "options": ["Search for a way around", "Try to lift debris", "Retreat back to the village"],
                "next": [18, 19, 3]
            },
            {
                "text": "You peer deeper into the cave and discover ancient markings on the wall.",
                "options": ["Examine the markings", "Ignore them and move on", "Take a rubbing of the markings"],
                "next": [20, 21, 3]
            },
            {
                "text": "The stranger reveals themselves as a guardian spirit of the woods.",
                "options": ["Ask for guidance", "Demand answers about the dark power", "Greet them respectfully"],
                "next": [22, 23, 24]
            },
            {
                "text": "The waters of the fountain heal your wounds and restore your strength.",
                "options": ["Look for clues nearby", "Drink deeply from the fountain", "Mark the location on your map"],
                "next": [25, 26, 2]
            },
            {
                "text": "The map leads you through treacherous terrain filled with dangers aplenty.",
                "options": ["Press onward", "Camp for the night", "Return to the village"],
                "next": [27, 28, 2]
            },
            {
                "text": "You discover an ancient forest guardian asleep among the trees.",
                "options": ["Awake the guardian", "Sneak past", "Rest beside them"],
                "next": [29, 30, 4]
            },
            {
                "text": "The tome reveals rituals for summoning protection spells.",
                "options": ["Attempt to learn a spell", "Use the book for guidance", "Close the tome"],
                "next": [31, 32, 2]
            },
            {
                "text": "A sinister energy surrounds you, urging you to leave.",
                "options": ["Fight the darkness", "Attempt a ritual of protection", "Flee the cave"],
                "next": [33, 34, 3]
            },
            {
                "text": "You venture deeper into the forest and encounter a demon beast!",
                "options": ["Fight the beast", "Attempt to negotiate", "Set a trap"],
                "next": [35, 36, 3]
            },
            {
                "text": "The markings tell of a long-lost treasure buried deep beneath the mountains.",
                "options": ["Follow the directions", "Seek more information", "Ignore them and continue exploring"],
                "next": [37, 38, 3]
            },
            {
                "text": "The spirit offers to bless your journey if you help it retrieve a stolen artifact.",
                "options": ["Accept the quest", "Refuse and leave", "Negotiate for rewards"],
                "next": [39, 40, 24]
            },
            {
                "text": "The forest comes alive with animal spirits, guiding you to safety.",
                "options": ["Follow their lead", "Thank them and move on", "Try to communicate"],
                "next": [41, 42, 2]
            },
            {
                "text": "Your drink gives you visions. Do you follow them?",
                "options": ["Embrace the visions", "Fight against them", "Share them with others"],
                "next": [43, 44, 27]
            },
            {
                "text": "A dark castle looms in the distance, rumored to hold great evil.",
                "options": ["Investigate the castle", "Avoid it and move on", "Plan a siege"],
                "next": [45, 46, 2]
            },
            {
                "text": "You encounter travelers who share news of distant lands.",
                "options": ["Join them", "Ask for more information", "Part ways"],
                "next": [47, 48, 2]
            },
            {
                "text": "An enchantress offers you a choice of gifts; each has its price.",
                "options": ["Choose a magical artifact", "Request knowledge", "Politely decline"],
                "next": [49, 50, 2]
            },
            {
                "text": "You meet an ancient dragon resting atop a hoard of treasures.",
                "options": ["Attempt to befriend the dragon", "Steal a treasure", "Ask about the history of the land"],
                "next": [51, 52, 2]
            },
            {
                "text": "A band of thieves demands tribute from you to pass.",
                "options": ["Fight the thieves", "Pay the tribute", "Attempt to bribe them with knowledge"],
                "next": [53, 54, 2]
            },
            {
                "text": "The treasure map leads to a hidden cave beneath the mountains.",
                "options": ["Enter the cave", "Scout the area first", "Return to the village for assistance"],
                "next": [55, 56, 2]
            },
            {
                "text": "Inside the cave, shadows move, hinting at lurking danger.",
                "options": ["Investigate the shadows", "Set a trap", "Prepare for a fight"],
                "next": [57, 58, 3]
            },
            {
                "text": "The guardian offers wisdom and a chance to learn its ancient ways.",
                "options": ["Accept the training", "Turn it down and leave", "Ask for stories"],
                "next": [59, 60, 24]
            },
            {
                "text": "You see a glimmering portal that beckons you closer.",
                "options": ["Enter the portal", "Examine it closely", "Leave it alone"],
                "next": [61, 62, 2]
            },
            {
                "text": "With a swift stroke of fate, you find yourself facing a legendary adversary!",
                "options": ["Engage in combat", "Try to outsmart them", "Flee bravely"],
                "next": [63, 64, 2]
            },
            {
                "text": "The dragon reveals a secret pathway that can lead to great rewards.",
                "options": ["Follow the pathway", "Report back to the village", "Challenge the dragon"],
                "next": [65, 66, 2]
            },
            {
                "text": "A hint of prophecy lingers in the air: 'The one who seeks shall find'.",
                "options": ["Follow your intuition", "Seek guidance", "Ignore the signs"],
                "next": [67, 68, 2]
            },
            {
                "text": "A haunting melody sounds in the air, drawing you closer.",
                "options": ["Investigate the source", "Close your ears", "Leave quickly"],
                "next": [69, 70, 2]
            },
            {
                "text": "Using your newfound wisdom, you uncover a gravely overlooked truth.",
                "options": ["Share it with the villagers", "Keep it secret", "Use it to gain power"],
                "next": [71, 72, 2]
            },
            {
                "text": "An ancient riddle poses a challenge for you.",
                "options": ["Attempt to solve it", "Seek help from others", "Ignore it and move on"],
                "next": [73, 74, 2]
            },
            {
                "text": "Your journey leads you to an island with unusual flora and fauna.",
                "options": ["Investigate the wildlife", "Collect samples", "Prepare for an encounter"],
                "next": [75, 76, 2]
            },
            {
                "text": "A storm brews on the horizon. Will you brave it?",
                "options": ["Ride out the storm", "Seek shelter", "Continue onward"],
                "next": [77, 2, 78]
            },
            {
                "text": "You stumble upon a forgotten ancient shrine, filled with powerful relics.",
                "options": ["Search for treasures", "Restore the shrine", "Leave it untouched"],
                "next": [79, 80, 2]
            },
            {
                "text": "As you compile your experiences, a shadowy figure approaches.",
                "options": ["Stand your ground", "Try diplomacy", "Retreat"],
                "next": [81, 82, 2]
            },
            {
                "text": "Decisions you make now may shape the fate of Eldoria.",
                "options": ["Join a faction", "Stay independent", "Seek alliances"],
                "next": [83, 84, 85]
            },
            {
                "text": "Embrace your destiny or flee from it. The choice is yours.",
                "options": ["Face your fears", "Turn away", "Seek solace in friends"],
                "next": [86, 87, 88]
            },
            {
                "text": "You find yourself in a marketplace filled with exotic goods. What will you buy?",
                "options": ["An ancient map", "A potion of healing", "Exotic spices"],
                "next": [89, 90, 91]
            },
            {
                "text": "You overhear a conspiratorial conversation about a plot against the king.",
                "options": ["Intervene in the conversation", "Try to gather more information", "Report it to the guards"],
                "next": [92, 93, 94]
            },
            {
                "text": "Entering a dark tavern, you meet a mysterious stranger offering a bounty.",
                "options": ["Accept the bounty", "Decline and leave", "Ask for more details"],
                "next": [95, 96, 97]
            },
            {
                "text": "You follow a trail of odd markings leading away from the village.",
                "options": ["Continue following the trail", "Return to the village", "Set up camp for the night"],
                "next": [98, 99, 2]
            },
            {
                "text": "A sudden blizzard engulfs you in the mountains. What will you do?",
                "options": ["Try to find shelter", "Brave the storm", "Retreat back"],
                "next": [100, 101, 2]
            },
            {
                "text": "You stumble upon a dying adventurer who gives you a cryptic warning.",
                "options": ["Ask for details", "Tend to their wounds", "Leave them to their fate"],
                "next": [102, 103, 2]
            },
            {
                "text": "In the distance, you see a castle illuminated by an eerie green light.",
                "options": ["Investigate the castle", "Avoid it", "Scout the area for dangers"],
                "next": [104, 105, 2]
            },
            {
                "text": "During your travels, you meet a bard who offers to join you.",
                "options": ["Accept the bard into your party", "Politely decline", "Learn a song from the bard"],
                "next": [106, 107, 2]
            },
            {
                "text": "You find an ancient artifact buried in the ground.",
                "options": ["Take the artifact", "Leave it where it is", "Inspect it for magical properties"],
                "next": [108, 109, 2]
            },
            {
                "text": "An unexpected storm forces you to seek refuge in an abandoned cabin.",
                "options": ["Explore the cabin", "Rest until the storm passes", "Leave immediately"],
                "next": [110, 111, 2]
            },
            {
                "text": "You hear tales of a treasure guarded by a fierce dragon.",
                "options": ["Seek the treasure", "Ignore the tales", "Join a group to hunt the dragon"],
                "next": [112, 113, 2]
            },
            {
                "text": "You see a village under attack by goblins. What will you do?",
                "options": ["Help the villagers", "Sneak around to gather information", "Leave the area"],
                "next": [114, 115, 2]
            },
            {
                "text": "A ghostly apparition appears and beckons you to follow.",
                "options": ["Follow the ghost", "Ignore it and move on", "Ask where it wants to take you"],
                "next": [116, 117, 2]
            },
            {
                "text": "You discover a secret entrance to a hidden dungeon.",
                "options": ["Explore the dungeon", "Leave it alone", "Set up camp nearby"],
                "next": [118, 119, 2]
            },
            {
                "text": "You receive a letter from an old friend asking for help.",
                "options": ["Rush to help them", "Ignore the letter", "Investigate the situation first"],
                "next": [120, 121, 2]
            },
            {
                "text": "While camping, you hear whispers in the night.",
                "options": ["Investigate the whispers", "Ignore them", "Wake your companions"],
                "next": [122, 123, 2]
            },
            {
                "text": "Your path crosses with a wandering merchant.",
                "options": ["Trade with the merchant", "Ask about local rumors", "Ignore and pass by"],
                "next": [124, 125, 2]
            },
            {
                "text": "You find a hidden trail leading through the forest.",
                "options": ["Follow the trail", "Mark it on your map", "Leave it alone"],
                "next": [126, 127, 2]
            },
            {
                "text": "A group of adventurers invites you to join their quest.",
                "options": ["Join their quest", "Decline politely", "Ask about their goals"],
                "next": [128, 129, 2]
            },
            {
                "text": "You must choose a path: one leads to danger, the other to riches.",
                "options": ["Take the dangerous path", "Take the safer path", "Settle for neither"],
                "next": [130, 131, 2]
            },
            {
                "text": "You hear rumors of a cursed item that brings misfortune.",
                "options": ["Investigate the curse", "Avoid it entirely", "Seek out the cursed item"],
                "next": [132, 133, 2]
            },
            {
                "text": "A shimmering portal opens before you.",
                "options": ["Step through the portal", "Inspect its edges", "Run away"],
                "next": [134, 135, 2]
            },
            {
                "text": "You find a magical artifact that glows in your presence.",
                "options": ["Take the artifact", "Leave it alone", "Examine it for powers"],
                "next": [136, 137, 2]
            },
            {
                "text": "A decrepit old man offers to tell your fortune.",
                "options": ["Listen to his fortune", "Refuse him", "Leave him some coins"],
                "next": [138, 139, 2]
            },
            {
                "text": "You receive news of a great tournament in the capital.",
                "options": ["Travel to the tournament", "Stay and explore your current area", "Seek out an opponent"],
                "next": [140, 141, 2]
            },
            {
                "text": "You stumble upon a portal to another realm.",
                "options": ["Step through the portal", "Inspect it closely", "Leave it alone"],
                "next": [142, 143, 2]
            },
            {
                "text": "You find a mysterious key engraved with strange symbols.",
                "options": ["Keep the key", "Look for a lock", "Sell it for coin"],
                "next": [144, 145, 2]
            },
            {
                "text": "An assassin targets you've been mistaken for another.",
                "options": ["Confront the assassin", "Flee the area", "Seek help from nearby guards"],
                "next": [146, 147, 2]
            },
            {
                "text": "You face a moral dilemma involving an enemy attempting to flee.",
                "options": ["Let them go", "Capture them", "Strike them down"],
                "next": [148, 149, 2]
            },
            {
                "text": "You discover an ancient prophecy that includes your name.",
                "options": ["Learn more about the prophecy", "Run away from the knowledge", "Tell others of your fate"],
                "next": [150, 151, 2]
            },
            {
                "text": "You have gained the ability to speak with animals.",
                "options": ["Ask an animal for guidance", "Use it to distract prey", "Ignore it"],
                "next": [152, 153, 2]
            },
            {
                "text": "The echoes of your past adventures haunt you.",
                "options": ["Confront them", "Forget them", "Document your journey"],
                "next": [154, 155, 2]
            },
            {
                "text": "You catch sight of a phoenix rising from ashes, a sign of rebirth.",
                "options": ["Follow the phoenix", "Observe from afar", "Try to catch it"],
                "next": [156, 157, 2]
            },
            {
                "text": "You find a cave drawing depicting a hero much like yourself.",
                "options": ["Study the drawing", "Leave it be", "Search for context"],
                "next": [158, 159, 2]
            },
            {
                "text": "Legends of the past echo through a forgotten temple.",
                "options": ["Investigate the temple", "Preserve its secrets", "Seek to revitalize it"],
                "next": [160, 161, 2]
            },
            # New Chapters 101-180
            {
                "text": "A wise elder shares tales of a legendary sword hidden in the mountains.",
                "options": ["Seek the sword", "Dismiss it as an old wives' tale", "Ask for more details"],
                "next": [162, 163, 2]
            },
            {
                "text": "You stumble upon a hidden grove filled with luminous flowers.",
                "options": ["Collect some flowers", "Rest in the grove", "Investigate further"],
                "next": [164, 165, 2]
            },
            {
                "text": "A merchant offers a rare potion that promises long life.",
                "options": ["Buy the potion", "Refuse it", "Ask for a sample"],
                "next": [166, 167, 2]
            },
            {
                "text": "An ancient gate stands covered in ivy, rumored to lead to another realm.",
                "options": ["Open the gate", "Inspect its carvings", "Leave it alone"],
                "next": [168, 169, 2]
            },
            {
                "text": "A noble approaches, seeking your help to save their kidnapped daughter.",
                "options": ["Accept the mission", "Ask for payment", "Politely decline"],
                "next": [170, 171, 2]
            },
            {
                "text": "A strange fog rolls in, blanketing the area and obscuring your instincts.",
                "options": ["Follow the northern path", "Stay put and wait", "Retreat to a safer place"],
                "next": [172, 173, 2]
            },
            {
                "text": "You hear tales of a mystical beast dwelling in the nearby hills.",
                "options": ["Track the beast", "Ignore the tales", "Gather a party to hunt it"],
                "next": [174, 175, 2]
            },
            {
                "text": "A celestial being appears before you, offering a blessing.",
                "options": ["Accept the blessing", "Question its motives", "Turn away"],
                "next": [176, 177, 2]
            },
            {
                "text": "You discover footprints that lead to a hidden lair.",
                "options": ["Follow the footprints", "Set a trap", "Report to the village elder"],
                "next": [178, 179, 2]
            },
            {
                "text": "A bard sings a song of an ancient hero and their final battle.",
                "options": ["Listen intently", "Join in the singing", "Pay the bard for information"],
                "next": [180, 181, 2]
            },
            {
                "text": "You stumble upon a hidden book that grants immense knowledge.",
                "options": ["Read the book", "Take it with you", "Leave it alone"],
                "next": [182, 183, 2]
            },
            {
                "text": "A friendly giant offers you a challenge.",
                "options": ["Accept the giant's challenge", "Politely refuse", "Try to outsmart the giant"],
                "next": [184, 185, 2]
            },
            {
                "text": "In a nearby forest, you find a witch's hut.",
                "options": ["Enter the hut", "Knock on the door", "Leave it alone"],
                "next": [186, 187, 2]
            },
            {
                "text": "An ethereal melody lures you deeper into a magical glade.",
                "options": ["Investigate the source", "Run away", "Try to dance with the melody"],
                "next": [188, 189, 2]
            },
            {
                "text": "The air crackles with energy as you approach a storm.",
                "options": ["Brave the storm", "Seek shelter", "Investigate the storm's source"],
                "next": [190, 191, 2]
            },
            {
                "text": "You find an old map leading to a hidden treasure!",
                "options": ["Follow the map", "Hide the map", "Show it to the villagers"],
                "next": [192, 193, 2]
            },
            {
                "text": "A radiant light guides your path forward.",
                "options": ["Follow the light", "Ask the light for directions", "Ignore it"],
                "next": [194, 195, 2]
            },
            {
                "text": "You hear whispers of a clandestine society operating in the city.",
                "options": ["Join the society", "Gather more information", "Report them to the authorities"],
                "next": [196, 197, 2]
            },
            {
                "text": "You discover you have affinities with a magical element.",
                "options": ["Explore your powers", "Keep it secret", "Seek guidance from a mentor"],
                "next": [198, 199, 2]
            },
            {
                "text": "In a meadow, you find a group of fairies celebrating.",
                "options": ["Join the celebration", "Observe quietly", "Interrupt their festivities"],
                "next": [200, 201, 2]
            },
            {
                "text": "A rumor spreads of an accident caused by a witch.",
                "options": ["Investigate the rumor", "Ignore it", "Spread the rumor further"],
                "next": [202, 203, 2]
            },
            {
                "text": "An oracle challenges you with a series of questions.",
                "options": ["Answer the questions", "Decline to answer", "Ask your own questions"],
                "next": [204, 205, 2]
            },
            {
                "text": "A mysterious fog engulfs a nearby village.",
                "options": ["Investigate the village", "Avoid it", "Seek shelter elsewhere"],
                "next": [206, 207, 2]
            },
            {
                "text": "You find a patient wolf injured in a hunt.",
                "options": ["Tend to its wounds", "Leave it be", "Try to communicate"],
                "next": [208, 209, 2]
            },
            {
                "text": "An enchanted forest requires you to solve a riddle.",
                "options": ["Attempt to solve the riddle", "Bribe the guardian", "Leave the forest"],
                "next": [210, 211, 2]
            },
            {
                "text": "You encounter fellow adventurers sharing stories of their triumphs.",
                "options": ["Join their tales", "Listen quietly", "Ask for advice"],
                "next": [212, 213, 2]
            },
            {
                "text": "A battle between rival factions unfolds before your eyes.",
                "options": ["Take sides", "Avoid involvement", "Try to mediate peace"],
                "next": [214, 215, 2]
            },
            {
                "text": "You find a withered tree that seems to watch you.",
                "options": ["Inspect the tree", "Leave it alone", "Climb the tree"],
                "next": [216, 217, 2]
            },
            {
                "text": "A legend states that those who pass through a certain archway are granted favor.",
                "options": ["Walk through the archway", "Avoid it", "Research it further"],
                "next": [218, 219, 2]
            },
            {
                "text": "You see a patch of strange mushrooms glowing faintly.",
                "options": ["Examine the mushrooms", "Collect them", "Avoid the area"],
                "next": [220, 221, 2]
            },
            {
                "text": "A knight challenges you to prove your worth.",
                "options": ["Accept the challenge", "Politely decline", "Spark a conversation instead"],
                "next": [222, 223, 2]
            },
            {
                "text": "An ancient tome describes a hidden city of gold.",
                "options": ["Search for the city", "Doubt the tome's validity", "Consult a historian"],
                "next": [224, 225, 2]
            },
            {
                "text": "The winds carry a foreboding warning from the mountains.",
                "options": ["Heed the warning", "Ignore it", "Investigate the source"],
                "next": [226, 227, 2]
            },
            {
                "text": "A forgotten legend speaks of a beast that guards treasure.",
                "options": ["Hunt for the beast", "Avoid the treasure", "Offer something in exchange"],
                "next": [228, 229, 2]
            },
            {
                "text": "You encounter a disgruntled ghost seeking redemption.",
                "options": ["Help the ghost", "Dismiss it", "Interrogate it"],
                "next": [230, 231, 2]
            },
            {
                "text": "You unearth a cursed relic that corrupts nearby flora.",
                "options": ["Destroy it", "Examine its properties", "Bury it again"],
                "next": [232, 233, 2]
            },
            {
                "text": "You witness the aftermath of a fierce battle.",
                "options": ["Investigate the cause", "Leave the area", "Help any survivors"],
                "next": [234, 235, 2]
            },
            {
                "text": "An island of strange creatures beckons you.",
                "options": ["Explore the island", "Avoid it", "Search for rumored treasures"],
                "next": [236, 237, 2]
            },
            {
                "text": "You overhear a discussion about a powerful warlock in a nearby town.",
                "options": ["Seek out the warlock", "Ignore the conversation", "Investigate further"],
                "next": [238, 239, 2]
            },
            {
                "text": "You find treasure maps scattered around a camp.",
                "options": ["Collect them all", "Leave them be", "Teach others about them"],
                "next": [240, 241, 2]
            },
            {
                "text": "A cryptic figure claims to know the secrets of your past.",
                "options": ["Listen to them", "Dismiss them", "Challenge their claims"],
                "next": [242, 243, 2]
            },
            {
                "text": "A rivalry at sea could lead to a legendary prize.",
                "options": ["Join one of the crews", "Stay on shore", "Set sail alone"],
                "next": [244, 245, 2]
            },
            {
                "text": "You learn of a family heirloom lost to time.",
                "options": ["Search for the heirloom", "Leave it be", "Ask for help"],
                "next": [246, 247, 2]
            },
            {
                "text": "You find a seer who offers to reveal your future.",
                "options": ["Listen to their insight", "Reject their offer", "Ask questions"],
                "next": [248, 249, 2]
            },
            {
                "text": "A storm brews as you encounter a mysterious ship.",
                "options": ["Board the ship", "Avoid it", "Ask for safe passage"],
                "next": [250, 251, 2]
            },
            {
                "text": "A long-lost companion appears unexpectedly.",
                "options": ["Embrace them", "Question their motives", "Send them away"],
                "next": [252, 253, 2]
            },
            {
                "text": "You hear a rumor about a hidden library containing knowledge of the ancients.",
                "options": ["Seek the library", "Doubt its existence", "Ask around for directions"],
                "next": [254, 255, 2]
            },
            {
                "text": "A hulking beast blocks your path demanding tribute.",
                "options": ["Fight the beast", "Pay the tribute", "Negotiate"],
                "next": [256, 257, 2]
            },
            {
                "text": "You join a gathering of scholars discussing magic.",
                "options": ["Engage in conversation", "Stay back and listen", "Share your knowledge"],
                "next": [258, 259, 2]
            },
            {
                "text": "A call to arms resonates through the kingdom.",
                "options": ["Join the army", "Gather allies", "Stay neutral"],
                "next": [260, 261, 2]
            },
            {
                "text": "You meet a dragon who offers wisdom in exchange for service.",
                "options": ["Accept the offer", "Reject it", "Ask for a favor"],
                "next": [262, 263, 2]
            },
            {
                "text": "You walk through a market filled with oddities and magical trinkets.",
                "options": ["Buy something", "Inspect the wares", "Walk away"],
                "next": [264, 265, 2]
            },
            {
                "text": "A band of mercenaries offers to help you for a price.",
                "options": ["Hire them", "Decline", "Try to negotiate lower prices"],
                "next": [266, 267, 2]
            },
            {
                "text": "A royal decree calls for champions to compete in a grand tournament.",
                "options": ["Sign up for the tournament", "Avoid the conflict", "Help others prepare"],
                "next": [268, 269, 2]
            },
            {
                "text": "You find a cursed sword that drains the life of its wielder.",
                "options": ["Take the sword", "Destroy it", "Bring it to a sage"],
                "next": [270, 271, 2]
            },
            {
                "text": "You hear of a secret treasure hidden within the depths of a cursed cave.",
                "options": ["Try to find the treasure", "Inform the town", "Gather a team"],
                "next": [272, 273, 2]
            },
            {
                "text": "In the forest, you come across a gathering of witches.",
                "options": ["Join them", "Sneak away", "Challenge them"],
                "next": [274, 275, 2]
            },
            {
                "text": "A mist engulfs the trail ahead, hiding dangers untold.",
                "options": ["Proceed cautiously", "Wait for the fog to lift", "Turn back"],
                "next": [276, 277, 2]
            },
            {
                "text": "The sound of battle drums echoes through the valley.",
                "options": ["Investigate the source", "Avoid it", "Prepare for combat"],
                "next": [278, 279, 2]
            },
            {
                "text": "You come across a sealed chest in a hidden alcove.",
                "options": ["Try to open the chest", "Leave it alone", "Seek a key"],
                "next": [280, 281, 2]
            },
            {
                "text": "You overhear a plot to steal a significant artifact.",
                "options": ["Report it", "Confront the thieves", "Join them"],
                "next": [282, 283, 2]
            },
            {
                "text": "You learn of a wise sage who knows about your family's past.",
                "options": ["Seek out the sage", "Ignore the gossip", "Ask around for more"],
                "next": [284, 285, 2]
            },
            {
                "text": "A dark omen crosses your path, foreshadowing chaos.",
                "options": ["Investigate the omen", "Avoid it", "Share your fears with others"],
                "next": [286, 287, 2]
            },
            {
                "text": "You hear a haunting song coming from a nearby cave.",
                "options": ["Investigate the cave", "Ignore the song", "Sing along"],
                "next": [288, 289, 2]
            },
            {
                "text": "A rival claims your fame and challenges you.",
                "options": ["Accept the challenge", "Ignore them", "Try to talk them out of it"],
                "next": [290, 291, 2]
            },
            {
                "text": "You encounter a voting council deciding the fate of your village.",
                "options": ["Support a side", "Stay neutral", "Challenge them"],
                "next": [292, 293, 2]
            },
            {
                "text": "A blast of energy erupts from the ground beneath you.",
                "options": ["Investigate the blast site", "Run away", "Call for help"],
                "next": [294, 295, 2]
            },
            {
                "text": "In a watering hole, you see reflections of past adventurers.",
                "options": ["Gaze into the water", "Dip your hand", "Throw a stone"],
                "next": [296, 297, 2]
            },
            {
                "text": "A shadowy figure promises to grant you one wish.",
                "options": ["Make a wish", "Refuse the offer", "Ask for more information"],
                "next": [298, 299, 2]
            },
            {
                "text": "You hear the sound of an approaching storm.",
                "options": ["Seek shelter quickly", "Prepare for the rain", "Watch the lightning"],
                "next": [300, 301, 2]
            },
            {
                "text": "A river stand in your way, swirling ominously.",
                "options": ["Attempt to cross", "Build a raft", "Look for a bridge"],
                "next": [302, 303, 2]
            },
            {
                "text": "A friendly druid offers to teach you about nature.",
                "options": ["Accept the offer", "Politely decline", "Ask them questions"],
                "next": [304, 305, 2]
            },
            {
                "text": "You find a dormant volcano, said to hide treasures within.",
                "options": ["Explore the volcano", "Keep your distance", "Consult locals"],
                "next": [306, 307, 2]
            },
            {
                "text": "A mysterious mist blocks your way on a narrow path.",
                "options": ["Brave the mist", "Avoid the path", "Set a fire to clear it"],
                "next": [308, 309, 2]
            },
            {
                "text": "You discover a scroll that foretells your destiny.",
                "options": ["Read the scroll", "Ignore it", "Show it to a sage"],
                "next": [310, 311, 2]
            },
            {
                "text": "A battle for power in a nearby kingdom draws your attention.",
                "options": ["Get involved", "Stay uninvolved", "Investigate the factions"],
                "next": [312, 313, 2]
            },
            {
                "text": "You hear from a villager about strange happenings in the woods.",
                "options": ["Investigate", "Share it with others", "Ignore it"],
                "next": [314, 315, 2]
            },
            {
                "text": "You find a crystal that pulses with energy.",
                "options": ["Touch the crystal", "Examine it", "Leave it alone"],
                "next": [316, 317, 2]
            },
            {
                "text": "In the night, a creature howls ominously.",
                "options": ["Investigate the sound", "Stay in your tent", "Prepare for an attack"],
                "next": [318, 319, 2]
            },
            {
                "text": "You hear rumors of an impending windstorm.",
                "options": ["Prepare for it", "Seek shelter", "Help others prepare"],
                "next": [320, 321, 2]
            },
            {
                "text": "In your travels, you stumble across ruins of an ancient civilization.",
                "options": ["Explore the ruins", "Draw maps", "Search for clues"],
                "next": [322, 323, 2]
            },
            {
                "text": "You come across a mirror that shows the truth.",
                "options": ["Look into the mirror", "Cover it", "Smash the mirror"],
                "next": [324, 325, 2]
            },
            {
                "text": "Sky pirates threaten a nearby trading route.",
                "options": ["Join the fight", "Help in another way", "Stay clear of conflict"],
                "next": [326, 327, 2]
            },
            {
                "text": "A group of townsfolk ask for your help against local bandits.",
                "options": ["Assist them", "Decline", "Seek help from authorities"],
                "next": [328, 329, 2]
            },
            {
                "text": "A rare meteor shower occurs, said to grant wishes.",
                "options": ["Make a wish", "Enjoy the show", "Try to catch a meteor"],
                "next": [330, 331, 2]
            },
            {
                "text": "You discover a troupe of performers practicing.",
                "options": ["Join the performers", "Watch quietly", "Critique their work"],
                "next": [332, 333, 2]
            },
            {
                "text": "You overhear a plot against a local lord.",
                "options": ["Report it", "Investigate yourself", "Ignore the plot"],
                "next": [334, 335, 2]
            },
            {
                "text": "You meet a dragon who offers to train you.",
                "options": ["Accept the training", "Politely decline", "Ask for knowledge"],
                "next": [336, 337, 2]
            },
            {
                "text": "You come upon a magical waterfall believed to grant rejuvenation.",
                "options": ["Bathe in the waters", "Fill a container", "Observe it"],
                "next": [338, 339, 2]
            },
            {
                "text": "You find magical dust that could alter reality.",
                "options": ["Collect the dust", "Use some", "Dispose of it"],
                "next": [340, 341, 2]
            },
            {
                "text": "A far-off kingdom is rumored to hold a grand festival.",
                "options": ["Travel there", "Skip it", "Ask others about it"],
                "next": [342, 343, 2]
            },
            {
                "text": "A mysterious storm disrupts your path.",
                "options": ["Ride through the storm", "Seek shelter", "Wait it out"],
                "next": [344, 345, 2]
            },
            {
                "text": "You are captured by bandits and taken to their hideout.",
                "options": ["Attempt escape", "Negotiate with them", "Play along"],
                "next": [346, 347, 2]
            },
            {
                "text": "A traveling merchant offers a possibility: fame or fortune?",
                "options": ["Chase fame", "Pursue fortune", "Stay cautious"],
                "next": [348, 349, 2]
            },
            {
                "text": "A fairy grants you a temporary ability.",
                "options": ["Embrace the ability", "Doubt its value", "Use it to your advantage"],
                "next": [350, 351, 2]
            },
            {
                "text": "You find a statue that seems alive.",
                "options": ["Talk to the statue", "Leave it alone", "Try to touch it"],
                "next": [352, 353, 2]
            },
            {
                "text": "A navigation error leads you to an uncharted area.",
                "options": ["Explore this new area", "Backtrack", "Set up camp"],
                "next": [354, 355, 2]
            },
            {
                "text": "Rumors swirl of an artifact that can shift time.",
                "options": ["Search for the artifact", "Document these rumors", "Ignore them"],
                "next": [356, 357, 2]
            },
            {
                "text": "An opportunity arises to learn a new skill.",
                "options": ["Take the chance", "Decline", "Ask about alternatives"],
                "next": [358, 359, 2]
            },
            {
                "text": "You discover hidden talents within yourself.",
                "options": ["Cultivate the talents", "Hide them", "Share with others"],
                "next": [360, 361, 2]
            },
            {
                "text": "An unprecedented event occurs as a dragon descends from the sky.",
                "options": ["Approach the dragon", "Hide", "Observe from a distance"],
                "next": [362, 363, 2]
            },
            {
                "text": "A haunting assassin issue is in the air.",
                "options": ["Investigate the assassins", "Avoid their territory", "Gather information"],
                "next": [364, 365, 2]
            },
            {
                "text": "You learn that secrets lie in the eyes of the beholders.",
                "options": ["Seek out a beholder", "Learn more about them", "Avoid confrontation"],
                "next": [366, 367, 2]
            },
            {
                "text": "A prophecy speaks of a hero rising.",
                "options": ["Believe you are that hero", "Doubt the prophecy", "Ignore entirely"],
                "next": [368, 369, 2]
            },
            {
                "text": "You gain the attention of a powerful lich.",
                "options": ["Confront the lich", "Flee", "Negotiate terms"],
                "next": [370, 371, 2]
            },
            {
                "text": "A mysterious portal offers a glimpse into a different realm.",
                "options": ["Enter the portal", "Leave it be", "Document your findings"],
                "next": [372, 373, 2]
            },
            {
                "text": "An ancient riddle tests your knowledge.",
                "options": ["Try to solve it", "Ask for hints", "Ignore it"],
                "next": [374, 375, 2]
            },
            {
                "text": "You stumble across an abandoned ship wreck.",
                "options": ["Explore the wreck", "Report it", "Leave it alone"],
                "next": [376, 377, 2]
            },
            {
                "text": "An encounter with a trickster leads to a game of wits.",
                "options": ["Play along", "Challenge the trickster", "Call for help"],
                "next": [378, 379, 2]
            },
            {
                "text": "You see royal guards searching for something sinister.",
                "options": ["Offer your assistance", "Sneak away", "Investigate further"],
                "next": [380, 381, 2]
            },
            {
                "text": "An opportunity to learn magic from a wandering sorcerer arises.",
                "options": ["Join the sorcerer", "Politely decline", "Ask for knowledge"],
                "next": [382, 383, 2]
            },
            {
                "text": "A prophecy speaks of a hero rising.",
                "options": ["Voluntarily take the role", "Doubt its validity", "Reject the title"],
                "next": [384, 385, 2]
            },
            {
                "text": "You encounter a strange creature from the depths.",
                "options": ["Attempt to communicate", "Run away", "Study its behavior"],
                "next": [386, 387, 2]
            },
            {
                "text": "You are visited by a mysterious old woman.",
                "options": ["Listen to her", "Ask her questions", "Politely decline to speak"],
                "next": [388, 389, 2]
            },
            {
                "text": "A call for heroes echoes from the council.",
                "options": ["Heed the call", "Ignore it", "Seek advice"],
                "next": [390, 391, 2]
            }
        ]

    def start_game(self):
        self.update_text(self.chapters[self.chapter_index]["text"])
        self.show_options(self.chapters[self.chapter_index]["options"])

    def update_text(self, text):
        self.text_display.delete(1.0, tk.END)
        self.text_display.insert(tk.END, text + f"\n\nHealth: {self.health} | Experience: {self.experience}")

    def show_options(self, options):
        for i, btn in enumerate(self.option_buttons):
            if i < len(options):
                btn.config(text=options[i], state=tk.NORMAL)
            else:
                btn.config(state=tk.DISABLED)

    def roll_dice(self):
        dice_roll = random.randint(1, 20)  # Roll a 20-sided die
        messagebox.showinfo("Dice Roll", f"You rolled a {dice_roll}!")

    # Remaining functions...

    def make_decision(self, option_index):
        self.paths_taken.append(self.chapter_index)  # Record the path taken
        if self.health <= 0:
            self.end_game(success=False)
            return

        # Update experience and health for completing this chapter
        self.experience += 10
        self.health += 5
        
        # Move to the next chapter based on player choice
        self.chapter_index = self.chapters[self.chapter_index]["next"][option_index]

        # Update the story text and options
        self.update_text(self.chapters[self.chapter_index]["text"])
        self.show_options(self.chapters[self.chapter_index]["options"])

    def next_chapter(self):
        if self.chapter_index + 1 < len(self.chapters):
            self.chapter_index += 1
            self.start_game()
        else:
            messagebox.showinfo("End of Chapters", "You have reached the end of the current chapters.")

    def previous_chapter(self):
        if self.chapter_index > 0:
            self.chapter_index -= 1
            self.start_game()
        else:
            messagebox.showinfo("Start of Chapters", "You are at the beginning of the chapters.")

    def end_game(self, success):
        if success:
            messagebox.showinfo("Game Over", "Congratulations! You have completed the adventure!")
        else:
            messagebox.showwarning("Game Over", "You have fallen in battle. Your adventure has ended.")
        self.root.quit()

    def save_game(self):
        save_data = {
            'level': self.level,
            'experience': self.experience,
            'boss_battles': self.boss_battles,
            'health': self.health,
            'items': self.items,
            'chapter_index': self.chapter_index,
            'dark_item_collected': self.dark_item_collected,
            'paths_taken': self.paths_taken
        }
        with open('save_game.json', 'w') as f:
            json.dump(save_data, f)
        messagebox.showinfo("Save Game", "Game saved successfully!")

    def load_game(self):
        if os.path.exists('save_game.json'):
            with open('save_game.json', 'r') as f:
                save_data = json.load(f)
                self.level = save_data['level']
                self.experience = save_data['experience']
                self.boss_battles = save_data['boss_battles']
                self.health = save_data['health']
                self.items = save_data['items']
                self.chapter_index = save_data['chapter_index']
                self.dark_item_collected = save_data['dark_item_collected']
                self.paths_taken = save_data['paths_taken']
                self.update_text("Game loaded successfully. Resume your adventure!")
                self.show_options(self.chapters[self.chapter_index]["options"])
        else:
            messagebox.showerror("Load Game", "No saved game found.")

if __name__ == "__main__":
    root = tk.Tk()
    game = AdventureGame(root)
    root.mainloop()
