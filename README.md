# What I Learned Making a Heroes of the Storm Replay Analyzer
Listen to my story. It's a story of intrigue, of disappointment, but mostly it's a story about how incompetent Blizzard is.

One day, frustrated at how bad and inaccurate the ranking system in Heroes of the Storm is, I decided to make my own ranking system. Here it is: [StormStats](https://www.stormstats.net/hots/home)

There are several issues with the official ranking system the game uses:

### 1) It's not a ranking system
To qualify as a ranking system, one must order players by some metric. A single player has a **unique** rank. Gold, Silver, and Bronze are ranks at the Olympics - only one competitor can have each medal. First place, second place, third place, etc are ranks in racing games. Gold 3 is not a rank. Bronze 1 is not a rank. These are achievements. The game lumps players into these achievements, which are nothing more than a shiny icon with a feel-good word next to it.

In an actual ranking system, any player who knows their rank also knows how many players are ranked higher than them. This is not the case in most online games.

### 2) It scores solely by wins and losses
Rating systems that score players often use wins and losses as the main method for determining how players move in the ladder. This works for symmetrical, skill-based 1v1 games like chess, because better players are more likely to win a game, and they have maximum control over the outcome of the match. Heroes of the Storm is not a skill-based game (as we'll see later), and it is not a 1v1 game. The outcome of the match is 90% out of your hands. It's also not symmetrical - each team has a different set of heroes. You can be the best player in a match and still lose. You can be the worst player in a match and still win. This is because you have 4 teammates, any of which could be an AFK feeding dumbass.

### 3) It's designed to create unfair matches
The matchmaker in Heroes of the Storm assigns a matchmaking rating (MMR) to each player. The matchmaker tries to create "fair" matchups by getting 5 players together whose average MMR is near the average MMR of 5 other players. This means it is intentionally putting what it considers good players on the same team as bad players, to "even them out". It also gives you more or less points for wins based on whether you've been winning a lot recently.

The problem with this approach is that you are biasing the outcomes of matches and forcing all players towards a 50% win rate, regardless of how good they actually are. You are then using this forced 50% win rate to determine how good players are. The Heroes of the Storm matchmaker is actually much worse than a matchmaker picking players at random. Yes, this would sometimes create unfair matchups (which is no worse than what the matchmaker does now), but you would get a far more accurate MMR for each player.

### 4) It's designed to sell skins
The least impactful but most egregious flaw of the rating system is that Blizzard designed it to maximize microtransactions. No, really. they've even [patented it](https://patft.uspto.gov/netacgi/nph-Parser?Sect2=PTO1&Sect2=HITOFF&p=1&u=/netahtml/PTO/search-bool.html&r=1&f=G&l=50&d=PALL&RefSrch=yes&Query=PN/9789406). Players who spend money on the game are more likely to be paired up with players who haven't, to make the latter player jealous and encourage them to buy skins.

## A better way to rank group games
I've written game engines before that measure player skill directly by looking at the actions they take in-game. Blizzard doesn't have this capability, so they can't provide it, but I thought I could use the replays to tease some useful information out of them.

Firstly, what is a replay? While players are playing a match, the game is recording all the actions players are taking. Player 3 clicked on this location. Player 5 used ability B on target unit X, and so on. A file gets saved to your computer whenever a match finishes, and you can open that replay with the game, which essentially plays back all the players' actions. Replays also contain something called Stat Events which contain some useful metadata about what's happening in the match.

I thought grabbing data out of the replay was going to be easy. I'll just look at the stat events to get everything I need. Man, was I wrong. Heroes of the Storm is actually a modification of the StarCraft 2 engine. While making StarCraft 2, Blizzard decided they would do as little as possible to support replay analysis. If you've ever been to chess.com, you know they have some really awesome tools to analyze any game or position, and a top chess engine to explain what all the best moves are and why. Blizzard went out of their way to make sure this would be impossible for the fans to create.

Here's the full list of all the useful information the replay files contain:
- The player names, hero choices, and talent selections (sort of like items from other mobas) of each player.
- Player positions, which I get once every 240 game ticks (15 seconds)
- **SOME** map-specific events (more on this in a bit)
- Notification events like kills/deaths and mercenary (jungle) captures.
- Overall endgame stats. This mainly includes how much damage and healing each player caused.

The more astute among you may notice that this list is rather short. Here are some things that the replays do **NOT** contain:
- The locations of players for any of the other 239 game ticks within that 15 second interval.
- Most important information about map mechanics
- Individual damage events (who's dealing damage to who and when and with which abilities)
- Which abilities players are using (I get an ability ID, but these change with every patch and there is no known way to map them on a by-replay basis)

One of the things I wanted to track was how efficient players were being with their damage. Are you attacking invincible tanks who are actively receiving healing? Or are you attacking an enemy that can actually be killed? This cannot be determined without making my own StarCraft 2 engine.

### The replay format was made by 10 dudes who did not share ideas
Most of the time it took me to make the replay analyzer was in figuring out how everything in the replay worked. It is wildly inconsistent, with references to the same entities being represented in completely different ways. Sometimes team 0 is team 0, and sometimes it's team 11, and sometimes it's team 14. Sometimes the player IDs are 0-indexed, and sometimes they're 1-indexed. X/Y coordinates are sometimes the correct value, but sometimes they are the correct value times 4096. On Hanamura Temple, the objective spawns at a +3,-3 offset of where the replay says it spawns. When you channel the objective on Alterac Pass, it creates a unit to act as the channel beam VFX. On every other map where you channel the objective, it does not. All of this is strong evidence that different pieces of the replay were made by different people, and none of them shared any information or documentation about how things should be done, so they made it up for their individual pieces of it and nobody ever cared to standardize anything.

### The later maps were made by interns
Heroes of the Storm is a game with several different maps. Each map has its own unique map objective, which occasionally appears to force the teams to fight each other. Completing the objective gives your team a huge advantage in the game.

A lot of the earlier maps put into the game are supported by replay events. On Cursed Hollow, the goal is to collect three tributes that periodically spawn around the map, one at a time. I get the location and spawn position of each tribute, as well as the time it was claimed. I don't know which player claimed it, but I can accurately guess by looking at each player's click events. For replay analysis, I don't really care, because the players who are guarding that player are equally important in claiming the tributes. In the analyzer for that map, I'm giving points to each allied player who is near the objective when it is captured.

Some of the later maps contain little to no information about the match objective, to the point where two of the maps are not being analyzed at all. These are Hanamura and Towers of Doom.

On Hanamura Temple, the objective is a cart that spawns into the middle of the map. You have to push the cart towards the enemy base by standing near it. The cart only moves if your team is near it and the enemy team is not, so you have to fight off the enemy team in order to make progress. The only event I get on this map is which team currently controls the cart. Normally this would be fine because the cart should always move at the same speed. It does not. The cart moves more quickly or slowly depending on how many players are pushing it, which is something I cannot know. The difference in move speeds is large enough that the difference between pushing the cart on the same path usually varies by about 40%, and in extreme cases where the teams are pushing it back and forth for a long time, this percentage can grow unbounded.

On Towers of Doom, there are four altars on the map at different locations. The enemy's Core is invincible, and can only be damaged by claiming the altars, or by being bad and completely ignoring the objective (great design, Blizzard). Every few minutes, a set of altars will become active. The rules for which altars can activate in what order are complicated for no reason. The only event I get on this map is when a Core takes damage from an altar being claimed. I do not know when the altars activate, which altars are active, which altars are claimed when damage is done, or who is claiming the altars. The rules for which sets of altars spawn make it such that I can only figure out what the first set of altars is. There's a 25% chance I can figure out the second set of altars. Beyond that it is impossible to know what is happening in the game.

The difference in quality of replay data suggests that the later maps were made by some dudes who had no clue what they were doing. I would bet Blizzard sexually harassed some interns, then told them it would be fun for them to cobble together a map or two for this dead game. This also explains why these are the two worst maps in the game in terms of design.

# The Results
So, this is what most fo you came here for, right? There are a couple other websites out there that contain stats about players and heroes, but they were developed by people who only know how to make wordpress blogs, so the things they can do with their data are extremely limited. I, on the other hand, am a wizard, and one of my magic spells is data analysis.

## Game Balance
If you know anything about Blizzard, it should come as no surprise that Heroes of the Storm is very poorly balanced. This is something a lot of players have been saying since the alpha. The first thing that becomes obvious when you look at stats is how broken healing is. If your team has a healer and the enemy team does not, your chance to win the game is 95% from the loading screen. 92% of winning teams have more healing than the losing team. This is the single biggest indicator of victory.

Anyone who has mastered first-grade math already know why this is the case. Each healer usually outheals the top 1.4 damage dealers on the enemy team. Simply by picking a healer, your opponent's top DPS will deal 0 net damage over the course of the game, and another one of them has their damage cut in half permanently. Healers are also given tons of tools to make them unkillable, so focusing them is almost always a losing battle. This causes teamfights to usually last a very long time where almost no damage is actually being dealt by either team.

The hero with the most healing per game is actually Chen, who is not a healer. His "passive" is that he gains a 400 point shield every second for up to 4 seconds. This ability has a 4 second cooldown. It also fully refreshes his mana. One of Chen's talents makes this shield apply to his teammates, so every 4 seconds he can effectively shield his team for up to 10000 total damage. This shield counts as healing in the replay, resulting in the very large healing stat.

Most of the top heroes are permanently invincible tanks who have way too much damage. At the top of the list is Mal'Ganis, who is basically a way more broken version of Riven from League of Legends. Following him is Dehaka, which is an assassin with 4 health bars, free skills, a 3 second stun, and a mapwide teleport. Following her is D.Va, who is a tank with 17 health bars and who gets her ultimate at level 1 and is permanently immune to CC. You get the idea. Blizzard's designers are so incompetent that there are multiple heroes in the game that can become permanently invincible.

The difference between the top hero and the bottom hero in terms of overall power is 355%. A single Malganis is almost as powerful as an entire team of Novas. This does **not** mean that Malganis can solo 4-and-a-half Novas, any more than it means a Queen can capture 9 pawns in 1 move. It just means that Malganis has as much damage, healing, and utility as 4-and-a-half Novas.

At the start of each match, each team gets to ban 3 heroes each (4 before hero selection and 2 more in the middle of hero selection), totalling 6 bans per game. In a balanced game, each hero should be banned about the same amount as any other hero, which would result in a 6/90 chance, or roughly 7%.

The top banned heroes are Brightwing, Johanna, Stukov, and Rehgar, which are each banned about 600% more often than they should, given that only 6 heroes get banned out of 90 total heroes in the game. Rehgar's win rate is nearly 60%, in a game where the theoretical max win rate is 55%. This has actually been the case for several years, but Blizzard is too busy raping their employees, and thus can't take 15 minutes to do their jobs. In the last "balance update" they did, they reduced Rehgar's life regeneration by a rate of 2 health per minute. This was the AAA solution to a healer who has every mechanic and heals way too much: reduce his health regen by 0.008%.

As of this writing, there are 25951 ranked games tracked by the system. Johanna has been banned in 16223 of them. Probius has been banned in 89 of them. That's a 18128% difference. Morons on the forum still claim the game is "decently balanced."

Anyway, here are the chances you win a game if you have a higher endgame stat than the other team:

| Stat  | Chance of Winning |
| ------------- | ------------- |
| Healing | 92% |
| StormStats rating | 83% |
| Mercenary Camps* | 76% |
| Hero Damage  | 75% |
| Siege Damage | 66% |
| Crowd Control | 62% |
| In-Game MMR  | 50%  |

*Mercenary camps are measured by the strength of the camp and by how long they push the lane before dying.

## How much does the matchmaker suck?
The accuracy of a rating system can be measured by how often you can predict match outcomes based on the ratings of the players. If I do this blindly, StormStats has an accuracy of 89%. However, about half the players in my system only have one game played, so their rating will almost always be positive if they won, or negative if they lost. This is a form of selection bias. I can mostly avoid this by excluding replays that contain such players. When I do this, the accuracy drops to 83%, which is still outstanding. This means if you give me the account names of the players on the loading screen, I can predict which team will win 83% of the time. I don't even need to look at which heroes are picked. There's still a little bias in that number, but you can show mathematically that the accuracy has a limit as the bias decreases, and that limit is just over 78%. That is to say, if each player in my system had an infinite number of games played, I would be able to predict matches over 78% of the time.

This proves that the matchmaker is worthless. A matchmaker that isn't shit will almost always create fair matches where both teams have about an equal chance to win, which would make your prediction rate much closer to 50%. If the matchmaker actually worked, my hit rate should be no higher than 55%. A perfect matchmaker would by definition make it impossible to get anything other than 50%, but limited player pool forces all matchmakers to create slightly unfair games in the interest of shorter wait times.

So, I have already proved that the matchmaker sucks, but I didn't stop there, because I wanted to quantify how much it sucks, so I did that too. A player's total StormStats rating is a weighted sum of their last 100 games, with more recent games counting more towards the overall sum. Each game typically gives between between -20 and +20 points, depending on performance, and usually will be higher if your team wins. The average player gets -6 points per match. The average player has a total rating of -8, and 2/3 of the player base is within 45 points of that in either direction (-53 to +37). With most of the community being within 45 points of the average, you would expect a working matchmaker to be able to create two teams whose total rating diff is about 1/5 of that.

It doesn't. The average rating diff between teams is 72, which signifies about a 90% chance to win. This means that **OVER HALF** of the games the matchmaker is creating are extremely unfair, with the better team having a 90% chance to win or better. A rating diff of 10 signifies a 60% chance to win, which I will call a "fair" game. Those only happen 15% of the time. That's one in seven games.

The replay analyzer can detect players who AFK pretty accurately. It never detects false positives, but it does miss some disconnects. One in four games has an afk player on it. This is true of both Quick Match and Storm League. This means that only one in every 35 games is a fair game (each team has a <60% chance to win) with no afk players. **The Heroes of the Storm matchmaker creates good games less than 3% of the time.**

## Other technical hurdles ##
Before I went public with the website, I was working on an uploader that could see the account IDs and heroes selected in draft mode. It was going to show each player's most popular/successful heroes, and suggest bans for your team to make, based on what's been selected so far. It's at this point HotS started crashing regularly. Even after a full reinstall of battle.net and the game, HotS has become unplayable, and that tool has been abandoned.

Only one other person has made a tool like this, and his works by taking a screenshot of the game, and using image recognition to get the player names from the text on screen. This is just one example of the lengths fans have to go to in order to squeeze basic funtionality out of a game that should just have that functionality to begin with.

I had more plans for additional features, but with HotS being unplayable, and Blizzard being a worthless company full of drunk rapists (this isn't me ranting, that's actually true, google it), I don't have the motivation to do that. StormStats will exist in its current state, which is a few million times more accurate than everyone else's rating systems, so I'm ok with that.

## How to support esports ##

Imagine for a moment you wanted to make a competitive MOBA. What features would you put in it to support esports? A replay analyzer, or the tools for players to build one perhaps. An API that can grab player data (HotS has this but it's hidden to third parties). Maybe even an API that stores all the replays company-side so players don't have to send replays to each other or build websites to do it. This would also allow players to watch anybody else's games whenever they want, including tournament games. Live observing for tournaments? Delayed observing for ladder games? How about proper game balance to allow good players to reach the top ranks without getting stomped by heroes who can literally cheat. Or maybe just have a game engine that doesn't crash every 5 minutes.

You just imagined the opposite of every MobA on the market. You don't make an esport by releasing a nonfunctional piece of garbage and putting "ESPORTS" on the homepage. You actually have to make something that's competitive, and skill-based, and watchable, with accessible stats. The reason there are entire TV channels about basketball is because basketball is a balanced, skill-based, watchable game with accessible stats. There are 0 dedicated channels to HotS, or League, or DotA, and there never will be as long as these greedy companies prioritize squeezing microtransactions out of them rather than simply making a game people want to play.

The good news is, now that StormStats won't take up much development time anymore, I can grab some friends and start building a MobA. My only worry is that players over the last 15 years have gotten too accustomed to cheating mechanics being in games. Almost every pvp game today has invincibility, stuns, teleports, and other mario kart mechanics to punish good players and make mistakes not matter. Casual players who think they're good keep expecting mechanics like this to show up in games. I've had conversations with literally hundreds of players where I try to explain to them why cheating shouldn't exist in pvp games. None of them understood. We now live in a dark age of pvp games where a majority of people don't think permanent invincibility is a form of cheating. Any truly competitive game that I or anyone else releases will likely fail for not letting you cheat in it.
