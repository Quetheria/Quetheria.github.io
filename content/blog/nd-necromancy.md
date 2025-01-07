---
title: "Non-deterministic Necromancy"
date: 2025-01-07T10:59:15-05:00
publishdate: 2024-11-14T10:52:12-05:00
categories: ["blog"]
tags: ["side projects", "mtg"]
archives: ["2024/11"]
keywords: ["mtg", "magic the gathering", "automata theory", "nondeterminisim"]
description: "Magic: The Gathering, Automata theory, and game engines"
draft: false
---


### Introduction

I've been playing Magic: the Gathering (or Magic, or MTG) for years, beginning in around 2016 (for the uninitiated, Magic is essentially a card game about wizards fighting).
This was mostly in high-school and has always been a pretty casual interest,
but certain corners of game design and strategy pique my interest.

Also, as a part of my coursework, I need to take a Theory of Computation course,
which I was also somewhat taken with.

These things seem entirely unrelated, however the MTG game engine has been proven Turing complete[^1], and strategy is reducible to the halting problem[^1].

### Overview

Very briefly:
This post covers some background knowledge in the Magic community, "Infinite Combos" and how they relate to termination, and the meat of the article; Four Horseman.

### Magic Community History

Magic has a 31 year history, and with that comes both a large number of cards,
and a series of evolutions to the community. Beginning in 1993, Magic predates
internet prevalence, causing most of the discussion of strategy etc to be in paper magazines such as Inquest[^2] and Scrye[^3].
In late 90s to early 10s, this evolved into the MTG Salvation[^4] forums and articles on
The Magic Dojo[^5]. This era then folds into the general social media discussion of the late 10s-20s.

### Game Concepts

We've not addressed game engine components yet, and this article is intended to be
more about the abstract than about low-level game design, but some points are going to be necessary. These are very broad and do not acknowledge the massive number of corner cases that are possible.
1. Magic is generally a two player game.
2. Players lose the game if they have 0 life or draw a card from an empty deck
3. Object in play are 'permanents' on the 'battlefield'
4. Permanents can be 'untapped' (unused) or 'tapped' (used), and become untapped at the beginning of their controllers turn
5. Cards in other zones (a players hand, the 'graveyard', the 'library' or deck)  are considered cards
6. Cards are either spells or lands
7. Lands can be played once per turn, without external effects this upper-bounds the resources a play has to the number of turns they have taken.
8. Players take actions that place spells and abilities on the 'stack'
9. The stack is a standard stack data structure ie; last in first out, with spells resolving in LIFO
10. Card abilities can overrule essentially any other game rule **This will be very important later**


### Four Horseman

In the early 2010s, on MTG Salvation, people would share their latest deck ideas, insight on new cards, and advice on game strategy.
Among these postings is one of my favorite; the 2011 post: "[deck]The Four Horseman"[^6]. This post details a *unique* combo (from combination) deck.


###### What is Combo
Quick sidebar on combo decks; combo decks have been around for a while, and intend to use some combination of card abilities to win the game in a non-standard way.
For example; "Splinter Twin"[^7], one of the more famous combo decks, uses [Splinter Twin](https://scryfall.com/card/roe/165/splinter-twin) and [Deceiver Exarch](https://scryfall.com/card/nph/33/deceiver-exarch)
to win very quickly; you place Twin on the Exarch, then use Twin's ability ("Create a token that’s a copy of this creature, except it has haste. Exile that token at the beginning of the next end step.") for the cost of
tapping the Exarch. A new copy of Exarch enters, and its ability ("Untap target permanent you control.") triggers, you target the original Exarch, and do this ad infinitum, then attack with infinite creatures (semantically we need to name
a number of repetitions of this loop so not *infinite*, just TREE(3)).

Splinter Twin is considered an "A + B" combo, as in you have element "A" (Twin), and element "B" (Exarch), and you win. Four Horseman is a few steps more complicated than this.

###### Deck Breakdow
First of all the majority of the deck is normal; lands to produce resources, good "interactive" cards to prevent us from dying too early and deck filtration to to draw our other cards.
The other 16 cards, however, are not as normal. In rough order of the steps of the combo; 4x [Basalt Monolith](https://scryfall.com/card/lea/231/basalt-monolith), 4x [Mesmeric Orb](https://scryfall.com/card/mrd/204/mesmeric-orb), 4x [Narcomoeba](https://scryfall.com/card/fut/54/narcomoeba),
and one of each of the titular four horseman; [Emrakul, the Aeons Torn](https://scryfall.com/card/roe/4/emrakul-the-aeons-torn), [Dread Return](https://scryfall.com/card/tsp/104/dread-return), [Sharuum the Hegemon](https://scryfall.com/card/ala/194/sharuum-the-hegemon), and [Blasting Station](https://scryfall.com/card/5dn/107/blasting-station). This might seem like a lot, especially compared to Twin, but it's "only" 7 unique cards, and we only really need 9 of them overall.

###### The Actual Combo
1. Don't die for the first few turns (see: interactive cards, hope)
2. Get Basalt Monolith and Mesmeric Orb in play
    - Monolith produces three mana and costs three mana to untap,  so we can tap and untap it as much as we want.
    - Orb causes players to put cards in their graveyard every time they untap a permanent.
    - This lets us "mill" as many cards from the top of our deck as we want.
3. Mill ourselves until we have 3x Narcomoeba in play
    - However, we have a copy of Emrakul, the Aeons Torn in our deck, which says "When this card is put into a graveyard from anywhere, its owner shuffles their graveyard into their library."
    - So of the roughly 50 cards left in our deck at this point in the game we can only "progress the game state on permutations of our deck s.t. a Narcomoeba is higher in the deck than Emrakul.
    - Or, again assuming 50 cards remaining, only 1225 of 50! possible permutations will be valid.
    - This is already non-deterministic
4. Once we have our jellyfish, keep milling until we have at least Dread Return and Sharuum in our graveyard.
    - Also having Blasting Station will help, but is not necessary
    - This, again, is non deterministic due to Emrakul
5. Cast Dread Return for the cost of sacrificing our three Narcomoebas, and target Sharuum
6. When Sharuum enters, there is a trigger to return an artifact to play
    - If we have Blasting Station in our graveyard, bring it back
    - Otherwise, keep milling until we have it. Again, this is non-deterministic.
7. Keep milling
    - Eventually hit Emrakul, shuffling our graveyard, including the Narcomoebas from earlier into our deck.
8. Mill even more, until we get a Narcomoeba in play
    - This will trigger Blasting Station.
    - We can activate Blasting Station by sacrificing the Narcomoeba, and finally dealing damage to our opponent.
    - This is **also** non-deterministic
9. Repeat step 8 until our opponent is dead.

###### Some Issues
So, in addition to the non-determinism (which will be addressed, in the next section), these are:
drawing cards we need in our deck, the Magic: The Gathering tournament rules, and the Magic tournament structure.

If we draw Emrakul, Dread Return, Sharuum, Blasting Station, or multiple Narcomoebas that's *Bad* <sup>*tm*</sup>.
Technically we can stop doing anything and discard these cards to maximum hand size, but this will take forever,
possibly allowing our opponent to win while we do nothing.

Because of the non-determinism, there can be long periods where we just flip cards over, shuffle, and repeat.
Doing this is incredibly boring for our opponent, and is considered "Slow Play"[^8] in the Magic tournament rules.
This means, if we try to do this during a sanctioned tournament, our opponent can call for a judge, and result in us being issued warnings, or game losses.
Also as a result of this, games that we win may take a long time to conclude. As essentially all tournaments are played in a Best-of-Three structure,
and we absolutely are not guaranteed to win every game, we may not have time to play all three games. This could force us into a draw or even a loss.


###### Nondeterminism
I've addressed the components of this combo that are non-deterministic and the game implications of non-determinism, but have not
addressed the automata theory implications of this. These are, quite frankly, uninteresting. The actual non-deterministic steps of
the loop are essentially reducible to rolling a die infinite times and determining if a 1 has been rolled. The main point of interest is
that this occurred organically, and within the bounds of a card game about wizards and dragons.






[^1]: *Magic: The Gathering is Turing Complete* : https://arxiv.org/abs/1904.09828
[^2]: Archived Inquest \#1: https://archive.org/details/inquest.001.1995-05
[^3]: Archived Scrye \#1:  https://archive.org/details/ScryeMagazineIssue1
[^4]: MTG Salvation: https://www.mtgsalvation.com/
[^5]: Dojo archive: http://www.classicdojo.org/
[^6]: The Four Horseman on MTGSalvation: https://www.mtgsalvation.com/forums/the-game/legacy-type-1-5/developing-legacy/181684-deck-the-four-horsemen
[^7]: Blue-Red Splinter Twin by Brian Braun-duin: https://www.mtggoldfish.com/deck/321379
[^8]: Slow Play: https://blogs.magicjudges.org/rules/ipg3-3/


#### Credits and Citations:
All cards are copyright of Wizards of the Coast and their artists, those being;
Izzy for Deceiver Exarch and Sharuum the Hegemon,
Goran Josic for Splinter Twin,
Jesper Myrfors for Grim Monolith,
David Martin for Mesmeric Orb,
Mark Tedin for Emrakul the Aeons Torn,
Kev Walker for Dread Return,
and Stephen Tappin for Blasting Station.


