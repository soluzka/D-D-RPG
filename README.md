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
        self.story_index = "start"
        self.dark_item_collected = False
        self.paths_taken = []
        
        # Initialize story
        self.stories = {
            "start": {
                "text": "You awake in a small village near the kingdom of Eldoria—a land renowned for its peace and prosperity. "
                        "But rumors swirl of a dark power rising in the East. Will you answer the call for adventure?",
                "options": ["Venture into the Whispering Woods", "Explore the bustling village", "Investigate the mysterious cave"],
                "next": ["woods", "village", "cave"]
            },
            # Additional story branches
            "woods": {
                "text": "The Whispering Woods surround you, an enchanting place characterized by towering trees and flickering lights. "
                        "You hear distant laughter and the rustling of leaves. What do you wish to do?",
                "options": ["Follow the laughter", "Investigate the strange lights", "Seek a hidden path"],
                "next": ["follow_laughter", "investigate_lights", "seek_hidden_path"]
            },
            "village": {
                "text": "The village is alive with people and music. Merchants sell wares from far and wide, and the tavern is filled with adventurers. "
                        "You hear tales of a dragon terrorizing the east and a hidden goblin treasure.",
                "options": ["Talk to a merchant about quests", "Visit the tavern for rumors", "Search the marketplace for magical items"],
                "next": ["talk_merchant", "visit_tavern", "search_market"]
            },
            "cave": {
                "text": "The cave yawns before you, dark and foreboding. You feel the air grow cold as you step inside. "
                        "You hear soft whispers echoing in the distance, drawing you forward.",
                "options": ["Light a torch and explore", "Press forward into the darkness", "Return to the village"],
                "next": ["light_torch", "explore_darkness", "start"]
            },
            "follow_laughter": {
                "text": "You chase the sound, discovering a group of fairies dancing around a shimmering pool. They offer you a chance to make a wish.",
                "options": ["Make a wish for strength", "Make a wish for wisdom", "Thank them and leave"],
                "next": ["wish_strength", "wish_wisdom", "start"]
            },
            "investigate_lights": {
                "text": "As you approach, the lights reveal themselves to be will-o'-the-wisps, ethereal beings that can lead you astray or towards secrets.",
                "options": ["Follow the wisps", "Attempt to communicate", "Turn back"],
                "next": ["follow_wisps", "communicate_wisps", "start"]
            },
            "seek_hidden_path": {
                "text": "You search for a concealed trail and stumble upon an ancient stone altar pulsing with magic. A strange inscription beckons...",
                "options": ["Examine the altar", "Leave the altar alone", "Take a piece of stone"],
                "next": ["examine_alter", "start", "take_stone"]
            },
            "talk_merchant": {
                "text": "The merchant tells you of a quest to retrieve a lost artifact from the haunted ruins nearby. He offers you supplies for the journey.",
                "options": ["Accept the quest", "Decline and explore elsewhere", "Barter for better supplies"],
                "next": ["accept_quest", "start", "barter_supplies"]
            },
            "visit_tavern": {
                "text": "In the tavern, the barkeep shares tales of the goblin king and his hoard of treasures, guarded by fierce beasts.",
                "options": ["Gather a party to hunt the goblin king", "Listen to more rumors", "Dance with the local patrons"],
                "next": ["gather_party", "more_rumors", "dance_locals"]
            },
            "search_market": {
                "text": "You find a peculiar item—a potion rumored to grant temporary invisibility. The vendor is asking for a high price.",
                "options": ["Buy the potion", "Negotiate for a better price", "Leave the market"],
                "next": ["buy_potion", "negotiate_price", "start"]
            },
            "light_torch": {
                "text": "With your torch lit, you illuminate the cave, revealing ancient carvings on the walls that tell of a lost civilization.",
                "options": ["Study the carvings", "Press onward deeper", "Leave the cave"],
                "next": ["study_carvings", "explore_deeper", "start"]
            },
            "explore_darkness": {
                "text": "You tread cautiously, but the darkness envelops you. Suddenly, a bat swoops past, startling you!",
                "options": ["Try to catch the bat", "Draw your weapon", "Retreat"],
                "next": ["catch_bat", "draw_weapon", "start"]
            },
            "wish_strength": {
                "text": "You wish for incredible strength, and the fairies grant your wish! However, your increased might draws attention.",
                "options": ["Leave the area cautiously", "Challenge the fairies", "Seek out a battle with a fierce creature"],
                "next": ["leave_area", "challenge_fairies", "seek_battle"]
            },
            "wish_wisdom": {
                "text": "You seek wisdom, and the fairies fill your mind with knowledge about a powerful artifact hidden in the ruins.\n\nYou will gain +1 Intellect!",
                "options": ["Head to the ruins", "Share this knowledge with the villagers", "Seek guidance from a sage"],
                "next": ["head_to_ruins", "share_knowledge", "seek_sage"]
            },
            "follow_wisps": {
                "text": "The wisps lead you to a hidden treasure chest, but as you reach for it, they scream and vanish!",
                "options": ["Open the chest", "Leave the chest alone", "Look around for the wisps"],
                "next": ["open_chest", "start", "look_for_wisps"]
            },
            "communicate_wisps": {
                "text": "You attempt to communicate, and to your surprise, they share secrets of the forest! You can choose to follow their advice.",
                "options": ["Follow their suggestions", "Thank them and leave", "Demand to learn more"],
                "next": ["follow_suggestions", "start", "demand_knowledge"]
            },
            "examine_alter": {
                "text": "As you inspect the altar, a ghostly figure emerges, seeking a worthy champion to reclaim its lost treasures.",
                "options": ["Accept the challenge", "Refuse and leave", "Ask for guidance"],
                "next": ["accept_challenge", "start", "ask_guidance"]
            },
            "take_stone": {
                "text": "Grabbing a piece of stone activates a hidden trap, summoning a guardian beast!",
                "options": ["Fight the beast", "Set the stone down", "Attempt to reason with the beast"],
                "next": ["fight_beast", "set_down_stone", "reason_with_beast"]
            },
            "accept_quest": {
                "text": "You set out to find the lost artifact, discovering an abandoned village haunted by spirits and dark forces. Prepare for battle!",
                "options": ["Search the ruins", "Forge alliances with spirits", "Leave the area"],
                "next": ["search_ruins", "forge_alliances", "start"]
            },
            "barter_supplies": {
                "text": "You skillfully negotiate for better supplies, gaining a health potion and a map of the surrounding areas.",
                "options": ["Equip yourself and leave", "Ask more about the goblin king", "Head to the tavern"],
                "next": ["equip_and_leave", "ask_goblin_info", "head_to_tavern"]
            },
            "gather_party": {
                "text": "You gather a group of adventurers eager to challenge the goblin king. Are you ready for the quest?",
                "options": ["Set out immediately", "Prepare with training", 'Seek a wise leader'],
                "next": ["set_out_quest", "preparing_training", "seek_leader"]
            },
            "more_rumors": {
                "text": "More whispers fill your ears, tales of dragons and hidden treasures. Your curiosity is piqued; what do you want to do?",
                "options": ["Follow the rumors to the dragon's nest", "Gather supplies for the journey", "Share these rumors with others"],
                "next": ["follow_dragon_rumors", "gather_supplies", "share_with_others"]
            },
            "dance_locals": {
                "text": "You completely lose yourself in the music and dance, gaining favor with the locals who may aid you in your future quests.",
                "options": ["Invite locals for a quest", "Rest and enjoy the evening", "Offer to share your stories"],
                "next": ["invite_locals", "rest_evening", "share_stories"]
            },
            "buy_potion": {
                "text": "You hand over your gold and receive the potion. It may be the key to survival in your future battles.",
                "options": ["Head on your adventures", "Stay and chat with the vendor", "Look for other potions"],
                "next": ["start", "chat_vendor", "search_other_potions"]
            },
            "negotiate_price": {
                "text": "You haggle skillfully and get the potion for a much lower price! You feel like a true adventurer now.",
                "options": ["Buy more items", "Leave the market", "Consider using your savings for quests"],
                "next": ["buy_more_items", "start", "savings_for_quests"]
            },
            "study_carvings": {
                "text": "You study the carvings, unveiling the story of a legendary hero who faced immense odds. Do you wish to follow their path?",
                "options": ["Emulate their strength", "Try to unlock their magic", "Seek guidance from the spirit of the hero"],
                "next": ["emulate_strength", "unlock_magic", "seek_hero_spirit"]
            },
            "explore_deeper": {
                "text": "Continuing deeper, you encounter a haunting melody. It's captivating, but do you feel it's dangerous?",
                "options": ["Investigate the source", "Cover your ears", "Leave quickly"],
                "next": ["investigate_melody", "cover_ears", "start"]
            },
            "catch_bat": {
                "text": "You try to catch the bat but it flies through a narrow passage. Something stirs in the shadows.",
                "options": ["Investigate the shadows", "Prepare to defend yourself", "Flee the cave"],
                "next": ["investigate_shadows", "prepare_defense", "start"]
            },
            "draw_weapon": {
                "text": "You draw your weapon, but find it's just an echo. Relief washes over you, but the cave is still ominous.",
                "options": ["Advance cautiously", "Search for treasure", "Retreat carefully"],
                "next": ["advance_cautiously", "search_treasure", "start"]
            },
            "set_down_stone": {
                "text": "You quickly set the stone down, and the guardian beast pauses, confused. The tension in the air eases.",
                "options": ["Leave quietly", "Talk to the beast", "Attempt to befriend it"],
                "next": ["leave_quietly", "talk_beast", "befriend_beast"]
            },
            "reason_with_beast": {
                "text": "You try to reason with the guardian beast and find that it is a protector of the ancient artifacts.",
                "options": ["Ask for its help", "Challenge it to a duel", "Search for artifacts nearby"],
                "next": ["ask_for_help", "challenge_duel", "search_artifacts"]
            },
            "search_ruins": {
                "text": "In the ruins, you uncover an ancient altar with a glowing gem. It could be the artifact!",
                "options": ["Take the gem", "Investigate the altar", "Leave the ruins"],
                "next": ["take_gem", "investigate_alter", "start"]
            },
            "forge_alliances": {
                "text": "By sharing your plan with the spirits, some agree to assist you, revealing hidden paths and traps.",
                "options": ["Follow the spirits’ guidance", "Ask them more about the goblin king", "Plan an ambush"],
                "next": ["follow_guidance", "ask_about_goblin", "plan_ambush"]
            },
            "equip_and_leave": {
                "text": "With your supplies ready, you head towards the goblin territory, your heart pounding with anticipation.",
                "options": ["Strike first against the goblins", "Try sneaking into their lair", "Wait for nightfall"],
                "next": ["strike_first", "sneak_into_lair", "wait_nightfall"]
            },
            "set_out_quest": {
                "text": "You embark on your heroic quest with your companions, deep into the wild lands. Adventure awaits!",
                "options": ["Listen for goblin chatter", "Explore the woods", "Scout the area for supplies"],
                "next": ["listen_for_goblins", "explore_woods", "scout_area"]
            },
            "preparing_training": {
                "text": "In the training grounds, you hone your skills. After a few days, you feel strong and ready for battle.",
                "options": ["Set out immediately", "Gather more allies", "Learn about spells"],
                "next": ["set_out_quest", "gather_allies", "learn_spells"]
            },
            "seek_leader": {
                "text": "You search for a wise leader among the villagers. They share invaluable insights into defeating the goblin king.",
                "options": ["Follow their advice", "Distrust their words", "Consider recruiting them"],
                "next": ["follow_advice", "disregard_wisdom", "consider_recruitment"]
            },
            "follow_dragon_rumors": {
                "text": "You follow the rumors, leading you to a perilous mountain where the dragon lays in wait.",
                "options": ["Draft a plan to face the dragon", "Try to sneak past", "Seek allies in the village"],
                "next": ["plan_dragon_fight", "sneak_past", "ally_search"]
            },
            "gather_supplies": {
                "text": "You prepare for the journey, assuring you're equipped with potions and weapons.",
                "options": ["Set off on your adventure", "Take a moment to rest", "Plan your route carefully"],
                "next": ["set_off", "rest", "plan_route"]
            },
            "share_with_others": {
                "text": "You share your experiences with other villagers, igniting a spark of potential allies!",
                "options": ["Gather allies together", "Dismiss their help", "Seek more knowledge"],
                "next": ["gather_allies", "dismiss_help", "seek_knowledge"]
            },
            "open_chest": {
                "text": "You open the chest and find a treasure trove of gold and rare artifacts!",
                "options": ["Take your share", "Leave some for future adventurers", "Return to the will-o'-the-wisps"],
                "next": ["take_share", "leave_for_others", "return_wisps"]
            },
            "look_for_wisps": {
                "text": "You look around but find only silence. It's eerily quiet now.",
                "options": ["Seek a way out", "Investigate the chest", "Call for help"],
                "next": ["seek_way_out", "investigate_chest", "call_for_help"]
            },
            "follow_suggestions": {
                "text": "You follow the suggestions and discover a hidden grove, filled with healing herbs.",
                "options": ["Collect herbs", "Explore the grove more", "Sit and rest"],
                "next": ["collect_herbs", "explore_grove", "rest_in_grove"]
            },
            "thank_fairies": {
                "text": "You bow to the fairies as you leave, feeling blessed with their gifts.",
                "options": ["Continue your journey", "Return back to befriend them", "Search for others"],
                "next": ["continue_journey", "befriend_fairies_again", "search_for_others"]
            },
            "challenge_fairies": {
                "text": "You challenge the fairies to a game of wits. They gladly accept!",
                "options": ["Use your knowledge against them", "Attempt to trick them", "Back down"],
                "next": ["wits_game", "try_trickery", "back_down"]
            },
            "seek_battle": {
                "text": "You seek out fierce creatures. The challenge awaits in the forests.",
                "options": ["Ready your weapon", "Hunt for their lair", "Gather a party"],
                "next": ["ready_weapon", "hunt_lair", "gather_party"]
            },
            "head_to_ruins": {
                "text": "You feel drawn to the ruins, knowing powerful magic lies within.",
                "options": ["Search for artifacts", "Seek the help of ancient spirits", "Be cautious"],
                "next": ["search_artifacts", "seek_ancient_spirits", "be_cautious"]
            },
            "share_knowledge": {
                "text": "You share your knowledge with the villagers, empowering them to stand against evil.",
                "options": ["Rally them for a march", "Head into battle alone", "Plan a strategy with them"],
                "next": ["rally_villagers", "battle_alone", "plan_strategy"]
            },
            "seek_sage": {
                "text": "You search for a sage deep in the forest. Their wisdom is said to be unmatched.",
                "options": ["Ask for ancient knowledge", "Seek guidance for your quest", "Request a powerful spell"],
                "next": ["ask_knowledge", "ask_guidance", "request_spell"]
            },
            "investigate_carvings": {
                "text": "Your heart races as you realize these carvings depict prophecies of both glory and doom.",
                "options": ["Attempt to decode them", "Leave them be", "Prepare to face your destiny"],
                "next": ["decode_carvings", "leave", "prepare_facing_destiny"]
            },
            "fight_beast": {
                "text": "You charge at the beast with all your might, ready for the fight of your life!",
                "options": ["Attack quickly", "Use your skills", "Try to reason with it"],
                "next": ["quick_attack", "use_skills", "reason_with_beast"]
            },
            "set_down_stone": {
                "text": "You gently set the stone down. The guardian beast eyes you curiously. What will you do next?",
                "options": ["Speak to the beast", "Leave quietly", "Try to make amends"],
                "next": ["speak_beast", "leave_quietly", "make_amends"]
            },
            "leave_area": {
                "text": "You decide to leave, but the air is filled with tension as you exit.",
                "options": ["Return to your quest", "Search for other magical beings", "Venture to the next destination"],
                "next": ["continue_quest", "search_magical_beings", "next_destination"]
            },
            "talk_beast": {
                "text": "You bravely talk to the guardian beast. It eyes you with interest. Perhaps you can forge an alliance.",
                "options": ["Ask for help", "Challenge it", "Befriend it"],
                "next": ["ask_guardian_help", "challenge_guardian", "befriend_guardian"]
            },
            "search_artifacts": {
                "text": "As you search, treasure hunters appear, eying the same prize!",
                "options": ["Confront them", "Attempt to negotiate", "Sneak past them"],
                "next": ["confront_hunters", "negotiate_with_hunters", "sneak_by"]
            },
            "take_gem": {
                "text": "You take the glowing gem, and the spirits begin to stir. They are not pleased.",
                "options": ["Defend yourself", "Apologize", "Negotiate for peace"],
                "next": ["defend_yourself", "apologize", "negotiate_peace"]
            },
            "ask_for_help": {
                "text": "The guardian beast offers to assist you, stating it knows the secrets of the land.",
                "options": ["Follow it to hidden treasures", "Ask about dangers ahead", "Request its allegiance"],
                "next": ["follow_to_treasures", "ask_about_dangers", "request_alliance"]
            },
            "challenge_guardian": {
                "text": "You challenge the guardian to a duel—it accepts! What will be your strategy?",
                "options": ["Surprise attack", "Defensive posture", "Bide your time"],
                "next": ["surprise_attack", "defensive_posture", "bide_time"]
            },
            "befriend_guardian": {
                "text": "You attempt to befriend the guardian. It is curious, and you might stand a chance of killing. What will you offer?",
                "options": ["Show treasures you've found", "Tell it a tale of adventure", "Promise to protect the forest"],
                "next": ["show_treasures", "tell_tale", "promise_protection"]
            },
            "continue_quest": {
                "text": "You push onward, knowing that danger lurks around every corner. The danger makes you stronger!",
                "options": ["Investigate nearby sounds", "Take a rest before the next adventure", "Plan your next move"],
                "next": ["investigate_sounds", "take_rest", "plan_move"]
            },
            "search_magical_beings": {
                "text": "You wander deeper into the forest, hearing soft whispers guiding you. Suddenly, a magical being appears!",
                "options": ["Approach gently", "Demand answers", "Wait and see what happens"],
                "next": ["approach_gently", "demand_answers", "wait_and_see"]
            },
            "next_destination": {
                "text": "You set your sights on the next destination, resolute in your quest.",
                "options": ["Head to the enchanted river", "Seek the mountains", "Explore the forgotten ruins"],
                "next": ["enchanted_river", "mountains", "forgotten_ruins"]
            },
            "quick_attack": {
                "text": "You move swiftly, but the beast is quick! You will need to strategize.",
                "options": ["Change tactics", "Call for help", "Defend yourself"],
                "next": ["change_tactics", "call_for_help", "defend_self"]
            },
            "use_skills": {
                "text": "You use your quick reflexes and skills. The beast is taken aback! You have the upper hand.",
                "options": ["Press the attack", "Try to reason", "Flee to safety"],
                "next": ["press_attack", "try_reason", "flee"]
            },
            "reason_with_beast": {
                "text": "You find a way to connect with the beast. It seems to understand your intention.",
                "options": ["Ask to join you", "Offer gifts of food", "Leave it be"],
                "next": ["ask_join", "offer_food", "leave"]
            },
            "speak_beast": {
                "text": "You speak softly, explaining that you mean no harm. The beast calms down.",
                "options": ["Ask for guidance", "Leave the area", "Befriend the beast"],
                "next": ["ask_guidance", "leave_area", "befriend"]
            },
            "leave_quietly": {
                "text": "You slip away unnoticed. The shadows feel a bit less foreboding now.",
                "options": ["Explore nearby lands", "Head back to the village", "Prepare for a new journey"],
                "next": ["explore_lands", "head_to_village", "prepare_journey"]
            },
            "search_for_others": {
                "text": "You search for other magical beings in the forest. It's quiet, but you feel watched.",
                "options": ["Continue the search", "Set up a trap", "Rest for the night"],
                "next": ["continue_search", "set_up_trap", "rest"]
            },
            "continue_journey": {
                "text": "As you continue your journey, the road ahead seems clear, but what lurks behind?",
                "options": ["Stay vigilant", "Press on with courage", "Seek counsel from your allies"],
                "next": ["stay_vigilant", "press_on", "seek_counsel"]
            },
            "gather_allies": {
                "text": "You gather allies around you, and together you feel a strong bond forming.",
                "options": ["Embark on a quest together", "Decide on roles", "Plan a strategy"],
                "next": ["embark_quest", "decide_roles", "plan_strategy"]
            },
            "rest": {
                "text": "You find a small clearing and decide to rest. It feels right, but danger lurks still.",
                "options": ["Keep watch", "Sleep soundly", "Prepare a meal"],
                "next": ["keep_watch", "sleep_soundly", "prepare_meal"]
            },
            "investigate_sounds": {
                "text": "The nearby sounds grow louder. You draw closer, ready for anything.",
                "options": ["Prepare to fight", "Sneak closer", "Aim for surprise"],
                "next": ["prepare_fight", "sneak_closer", "aim_surprise"]
            },
            "take_rest": {
                "text": "You take a moment to breathe. What will you do next?",
                "options": ["Set out again", "Reflect on your journey", "Gather your thoughts"],
                "next": ["set_out_again", "reflect_journey", "gather_thoughts"]
            },
            "plan_move": {
                "text": "Determining your next move is critical. What will you do?",
                "options": ["Consult an ally", "Take a risk", "Stick to familiarity"],
                "next": ["consult_ally", "take_risk", "stick_to_familiarity"]
            },
            "play_kings": {
                "text": "You spot the goblin king, looting the village. Prepare yourself, for danger is near.",
                "options": ["Surprise him", "Plan an ambush", "Seek more allies"],
                "next": ["surprise_goblin", "plan_ambush", "seek_more_allies"]
            },
            "defend_yourself": {
                "text": "With swift movements, you defend yourself against the guardian's attacks. What will you do next?",
                "options": ["Strike back", "Parley for peace", "Retreat"],
                "next": ["strike_back", "parley", "retreat"]
            },
            "apologize": {
                "text": "You apologize to the spirits but they remain angry. You may need to take stronger actions.",
                "options": ["Seek a truce", "Defend yourself", "Seek forgiveness"],
                "next": ["seek_truce", "defend_yourself", "seek_forgiveness"]
            },
            "negotiate_peace": {
                "text": "You negotiate for peace with the guardian. It's cautious but seems intrigued.",
                "options": ["Reveal your journey", "Offer a trade", "Leave the area"],
                "next": ["reveal_journey", "offer_trade", "leave_area"]
            },
            "seek_dreams": {
                "text": "You decide to pursue your dreams, letting the wind guide you.",
                "options": ["Follow the road", "Seek treasures", "Share your dreams"],
                "next": ["follow_road", "search_treasures", "share_dreams"]
            },
            "call_for_help": {
                "text": "You call for aid, rallying your companions. They assist you bravely!",
                "options": ["Charge into battle", "Wait for the right moment", "Defend"],
                "next": ["charge_battle", "wait_moment", "defend"]
            },
            "weakness": {
                "text": "You decide that showing weakness will not be an option.",
                "options": ["Strengthen your defenses", "Seek your allies", "Go into hiding"],
                "next": ["strengthen_defenses", "seek_allies", "go_into_hiding"]
            },
            "take_gem": {
                "text": "With the gem in hand, you feel a surge of power coursing through your veins.",
                "options": ["Use its power", "Store it for later", "Show it to your companions"],
                "next": ["use_power", "store_later", "show_companions"]
            },
            "ask_allies": {
                "text": "You canvas the villagers for help. Each agrees to assist in your upcoming quest.",
                "options": ["Lead them into battle", "Seek stronger allies", "Plan a strategy"],
                "next": ["lead_battle", "seek_strong_allies", "plan_strategy"]
            },
            "press_on": {
                "text": "With renewed determination, you press onward, prepared for whatever comes your way.",
                "options": ["Search for a quest", "Explore new lands", "Prepare for battle"],
                "next": ["search_quest", "explore_lands", "prepare_battle"]
            },
            "set_off": {
                "text": "With your supplies secured, you set off towards adventure. What destiny awaits?",
                "options": ["Seek glory", "Challenge the darkness", "Build your legend"],
                "next": ["seek_glory", "challenge_darkness", "build_legend"]
            },
            "search_targets": {
                "text": "With your enemies marked, you resolve to settle the score.",
                "options": ["Strike at dawn", "Seek out key targets", "Gather your team"],
                "next": ["strike_dawn", "seek_key_targets", "gather_team"]
            },
        }

        # Text display and buttons
        self.text_display = tk.Text(root, height=15, width=50)
        self.text_display.pack()

        self.option_frame = tk.Frame(root)
        self.option_frame.pack()

        self.option_buttons = []
        for i in range(3):
            btn = tk.Button(self.option_frame, text=f"Option {i + 1}", command=lambda i=i: self.make_decision(i))
            btn.grid(row=0, column=i)
            self.option_buttons.append(btn)

        tk.Button(root, text="Save Game", command=self.save_game).pack(side=tk.LEFT)
        tk.Button(root, text="Load Game", command=self.load_game).pack(side=tk.RIGHT)
        tk.Button(root, text="Roll Dice", command=self.roll_dice).pack(side=tk.BOTTOM)

        self.start_game()

    def start_game(self):
        self.update_text(self.stories[self.story_index]["text"])
        self.show_options(self.stories[self.story_index]["options"])

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

    def make_decision(self, option_index):
        self.paths_taken.append(self.story_index)  # Record the path taken
        if self.health <= 0:
            self.end_game(success=False)
            return
            
        # Move story index based on player choice
        self.story_index = self.stories[self.story_index]["next"][option_index]

        # Continue the story
        self.update_text(self.stories[self.story_index]["text"])
        self.show_options(self.stories[self.story_index]["options"])

    def end_game(self, success):
        if success:
            messagebox.showinfo("Game Over", "Congratulations! You have completed the adventure! Eldoria is safe.")
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
            'story_index': self.story_index,
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
                self.story_index = save_data['story_index']
                self.dark_item_collected = save_data['dark_item_collected']
                self.paths_taken = save_data['paths_taken']
                self.update_text("Game loaded successfully. Resume your adventure!")
                self.show_options(self.stories[self.story_index]["options"])
        else:
            messagebox.showerror("Load Game", "No saved game found.")

if __name__ == "__main__":
    root = tk.Tk()
    game = AdventureGame(root)
    root.mainloop()
