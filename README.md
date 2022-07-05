# What I Learned Making a Heroes of the Storm Replay Analyzer
Listen to my story. It's a story of intrigue, of disappointment, but mostly it's a story about how incompetent Blizzard is.

One day, frustrated at how worthless the ranking system in Heroes of the Storm is, I decided to make my own ranking system. There are several issues with the official ranking system the game uses:

### 1) It's not a ranking system
A ranking system is defined as any system that orders players by some metric. A single player has a **unique** rank. Gold, Silver, and Bronze are ranks at the Olympics - only one competitor can have each medal. First place, second place, etc are ranks in racing games. Gold 3 is not a rank. Bronze 1 is not a rank. These are achievements. The game lumps players into these achievements, which are nothing more than a shiny icon with a feel-good word next to it.

### 2) It scores solely by wins and losses
Rating systems that score players often use wins and losses as the main method for determining how players move in the ladder. This works for skill-based 1v1 games like chess, because better players are more likely to win a game, and they have total control over the outcome of the match. Heroes of the Storm is not a skill-based game (as we'll see later), and it is not a 1v1 game where you control 100% of your team's actions. You can be the best player in a match and still lose. You can be the worst player in a match and still win. This is because you have 4 teammates, any of which could be an AFK feeding dumbass. 

### 3) It's designed to create unfair matches
The matchmaker in Heroes of the Storm assigns a matchmaking rating (MMR) to each player. The matchmaker tries to create "fair" matchups by getting 5 players together whose average MMR is near the average MMR of 5 other players. This means it is intentionally putting good players on the same team as bad players, to "even them out". It also gives you more or less points for wins based on whether you've been winning a lot recently. The problem with this approach is that you are biasing the outcomes of matches and forcing all players towards a 50% win rate. You are then using this forced 50% win rate to determine how good players are. The Heroes of the Storm matchmaker is actually much worse than if you were to grab 10 random players logged in and shuffle them around randomly between the teams. Yes, this would sometimes create unfair matchups (which is no worse than what they do now), but you would get a far more accurate MMR for each player over a much smaller number of games.

### 4) It's designed to sell skins
The least impactful but most egregious flaw of the rating system is that Blizzard designed it to maximize microtransactions. No, really. they've even [patented it](https://patft.uspto.gov/netacgi/nph-Parser?Sect2=PTO1&Sect2=HITOFF&p=1&u=/netahtml/PTO/search-bool.html&r=1&f=G&l=50&d=PALL&RefSrch=yes&Query=PN/9789406). Players who spend money on the game are more likely to be paired up with players who haven't, to make the latter player jealous and encourage them to buy skins.

## A better way to rank group games
I've written game engines before that measure player skill directly by looking at the actions they take in-game. Blizzard doesn't have this capability, so they can't provide it, but I thought I could use the replays to tease some useful information out of them.

Firstly, what is a replay? While players are playing a match, the game is recording all the actions players are taking. Player 3 clicked on this location. Player 5 used ability B on target unit X. A file gets saved to your computer whenever a match finishes, and you can open that replay with the game, which essentially plays back all the players' actions. Replays also contain something called Stat Events which contain some useful metadata about things going on in the match.

I thought grabbing data out of the replay was going to be easy. I'll just look at the stat events to get everything I need. Man, was I wrong. Heroes of the Storm is actually a modification of the StarCraft 2 engine. While making StarCraft 2, Blizzard decided they would do as little as possible to support replay analysis. If you've ever been to chess.com, you know they have some really awesome tools to analyze any game or position, and a top chess engine to explain what all the best moves are and why. Blizzard went out of their way to make sure this would be impossible for the fans to create.

Here's the full list of all the useful information the replay files contain:
- The player names, hero choices, and talent selections (upgrades) of each player.
- Player positions, which I get every 240 game ticks (15 seconds)
- **SOME** map-specific events (more on this in a bit)
- Notification events like deaths and mercenary captures.
- Overall endgame stats. This mainly includes how much damage and healing they did.

The more astute among you may notice that this list is rather short. Here are some things that the replays do **NOT** contain:
- The locations of players for any of the other 239 game ticks within that 15 second interval.
- Most important information about map mechanics
- Individual damage events (who's dealing damage to who and when)
- Which abilities players are using (I get an ability ID, but these change with every patch and there is no known way to map them on a by-replay basis)

One of the things I wanted to track was how efficient players were being with their damage. Are you attacking invincible tanks who are actively receiving healing? Or are you attacking an enemy that can actually be killed? This cannot be determined without making my own StarCraft 2 engine.

## The replay format was made by 10 dudes who did not share ideas
Most of the time it took me to make the replay analyzer was in figuring out how everything in the replay worked. It is wildly inconsistent, with references to the same entities being represented in completely different ways. Sometimes team 0 is team 0, and sometimes it's team 11, and sometimes it's team 14. Sometimes the player IDs are 0-indexed, and sometimes they're 1-indexed. X/Y coordinates are sometimes the correct value, but sometimes they are the correct value times 4096. On Hanamura Temple, the obejctive spawns at a +3,-3 offset of where the replay says it spawns. All of this is strong evidence that different pieces of the replay were made by different people, and none of them shared any information or documentation about how things should be done.
