# What I Learned Making a Heroes of the Storm Replay Analyzer
Listen to my story. It's a story of intrigue, of disappointment, but mostly it's a story about how incompetent Blizzard is.

One day, frustrated at how worthless the ranking system in Heroes of the Storm is, I decided to make my own ranking system. There are several issues with the official ranking system the game uses:

## It's not a ranking system
A ranking system is defined as any system that orders players by some metric. A single player has a unique rank. Gold, Silver, and Bronze are ranks at the Olympics - only one competitor can have each medal. First place, second place, etc are ranks in racing games. Gold 3 is not a rank. Bronze 1 is not a rank. These are achievements. The game lumps players into these achievements, which are nothing more than a shiny icon with a feel-good word next to it.

## It scores solely by wins and losses
Rating systems that score players often use wins and losses as the main method for determining how players move in the ladder. This works for skill-based 1v1 games like chess, because better players are more likely to win a game, and they have total control over the outcome of the match. Heroes of the Storm is not a skill-based game (as we'll see later), and it is not a 1v1 game where you control 100% of your team's actions. You can be the best player in a match and still lose. You can be the worst player in a match and still win. This is because you have 4 teammates, any of which could be an AFK feeding dumbass. 

## It's designed to create unfair matches
The matchmaker in Heroes of the Storm assigns a matchmaking rating (MMR) to each player. The matchmaker tries to create "fair" matchups by getting 5 players together whose average MMR is near the average MMR of 5 other players. This means it is intentionally putting good players on the same team as bad players, to "even them out". It also gives you more or less points for wins based on whether you've been winning a lot recently. The problem with this approach is that you are biasing the outcomes of matches and forcing all players towards a 50% win rate. You are then using this forced 50% win rate to determine how good players are. The Heroes of the Storm matchmaker is actually much worse than if you were to grab 10 random players logged in and shuffle them around randomly between the teams. Yes, this would sometimes create unfair matchups (which is no worse than what they do now), but you would get a far more accurate MMR for each player over a much smaller number of games.

## It's designed to sell skins
The least impactful but most egregious flaw of the rating system is that Blizzard designed it to maximize microtransactions. No, really. they've even [patented it](https://patft.uspto.gov/netacgi/nph-Parser?Sect2=PTO1&Sect2=HITOFF&p=1&u=/netahtml/PTO/search-bool.html&r=1&f=G&l=50&d=PALL&RefSrch=yes&Query=PN/9789406).
