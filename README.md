# Poker Handevaluation System

This project provides a custom implementation of a Texas Hold 'em handevaluation subsystem as described in chapter 5 of the original [Loki Poker Bot research paper](http://poker.cs.ualberta.ca/publications/papp.msc.pdf). For preflop play, fixed relative strength values are provided based upon simplified monte carlo simulations where all players call to the end of the hand. For postflop play, calculations are provided to determine immediate handstrength and future handpotential via complete scenario enumeration (given the opponent hand probability distributions as input).

From this handevaluation system, betting strategies can be derived and opponent models can be added to provide more realistic and adaptive hand probability distributions. The [university of Alberta Computer Poker Research Group](http://poker.cs.ualberta.ca/) has performed extensive research on both fronts and has succesfully used Poker as a testbed for AI and reinforcement learning techniques.

## Poker Hands and Hand Comparison
View the `src/poker/handranking` package. A `Card` object is defined and given a natural ordering. A `Hand` represents a set of cards that are known to one particular `Player` (and can include 2, 5, 6 or 7 cards for the game of Texas Hold 'em). The natural ordering between hands are dependent on the `handValue` and a distinction must be made between a `StartingHand` and a `RankableHand` to dermine the value of this field.

#### Starting Hand
A `StartingHand` consists of two cards representing the hole cards of a `Player` before any of the community cards have been dealt. Handvalues of starting hands are represented by Income Rates which are calculated based upon simplified monte carlo simulations where all players are given certain amounts of starting capital and subsequently call to the end of the hand. The simulations have been performed for a different number of active opponents and the obtained values have been hardcoded in `src/poker/handranking/util/HandRanker.java`. (Given the scope of this project, complete preflop to river scenario enumeration is computationally untractable. However, the current simplification provides a good baseline and allows for relative strength comparisons between different starting
hands).

#### Rankable Hand
In the game of Texas Hold 'em, a `RankableHand` consists of 5-7 cards representing a combination of two `Player` hole cards and 3-5 community cards. Rankable hands are comparable from an endgame perspective and their respective handvalues correspond to the strongest available 5-card subset. The 5-card ranking functionality and its corresponding value calculations are implemented by the `HandRanker`.

#### HandRanker
For preflop play, the `src/poker/poker/handrinking/util/HandRanker.java` provides hardcoded Income Rates (IR) for the 169 possible distinct starting hand combinations (depending on the number of active opponents: 1, 2-3 or 4+). For postflop play, the `HandRanker` detects the type of poker hand that corresponds to particular 5-card combinations. The complete handsprectrum -including kickers- is taken into account and return values are calculated in such a way that stronger card combinations correspond to higher return values.


## Hand Evaluation - Handstrength and Potential
A poker player should take both his current handstrength and future handpotential into account when making strategic betting decisions. Current handstrength can be defined as the expected chance of being currently ahead while positive (negative) handpotential corresponds to the expected chance of improving (regressing) from the losing (winning) hand to the winning (losing) hand. Note that the handpotential metrics are related to the implied and reverse implied odds poker concepts. An implementation of these metrics is provided in `src/poker/handranking/util/Player.java`: View the `calculateHandStatistics` method.

#### Handstrength
In order to calculate the handstrength against a single opponent, the expected starting hand probability distribution for this opponent must be provided. Given this opponent model, the handstrength calculator enumerates the opponent hole card combinations and calculates the players' chances of being ahead on the current board. For multi-opponent play a (relatively accurate) simplification is made by using an exponentiation to the number of active opponents. Additionally, in the Loki paper, suggestions are made on how to combine multiple opponent models / probability distributions into one consistent field array.

#### Handpotential
To determine positive and negative handpotential, one-step ahead (flop to turn, or turn to river) or two-step ahead (flop to river) enumerations of all possible outcomes are simulated. To accomplish this, a transitionmatrix is utilzed to log both the current state (win/loss/tie) for every possible opponent starting hand and the respective end state (win/loss/tie) for the corresponding one -or two step ahead simulations. Given this transitionmatrix and the opponent starting hand probabilities the expected future handpotentials can subsequently be determined.

#### HandFactory
The handfactory provides cashing of Rankable Hands and avoids excessive instantiation overhead (and corresponding handvalue function calls) for permutated hands that are basically identical from a hand valuation point of view. This is accomplished by using hashes based off the cards and results in a sgnificant speedup for the two-step ahead calculations.

#### Effective hand strength versus (effective) pot odds.
The practical usefulness of the previous concepts emerges when they are combined into the following equation:

- EHS = HS + (1-HS) x positive hanpotential - HS x negative handpotential

This value can potentially be compared against other measures such as pot odds and effective odds in order to determine expected values (EV) of betting decisions.

## Example
View `poker/Pokergame.java` and optionally uncomment line 262. Consider the following scenario.

- Starting hand: A\_hearts + Q\_hearts
- Flop: 3\_hearts + 4\_spades + J\_hearts
- Assume uniform probability distributions for the opponents. (Note that a more realistic opponent model can potentially be obtained when using a probability distribution based on the normalized income rate (IR) values)

One-step ahead simulation output:

- Handstrength 1 opponent: 0.5851063829787234
- HandStrength 5 opponents: 0.06857632055948792
- Positive Potential: 0.30112721417069244
- Negative Potential: 0.0993939393939394

Two-step ahead simulation output:

- Handstrength 1 opponent: 0.5851063829787234
- HandStrength 5 opponents: 0.06857632055948792
- Positive Potential: 0.5050257311126877
- Negative Potential: 0.1433984109873438

This example indicates that the player has a 58 percent chance of being ahead on the current board in the one opponent scenario, but he is almost certain to be behind when there are five other players in the game. The player also has a very high chance of improving his hand and obtaining the strongest hand in the scenario that he is behind on the current board. On the other hand, his chances of losing the showdown when he is currently already ahead are quite low (about 14.3 percent from flop to river). Calculating the EHS gives the player an expected showdown win rate of 87.8 percent and 54.9 percent in the 1 opponent and 5 opponent scenarios respectively.

## Licensing
This software is copyrighted by EssentialQuant ltd and is available for redistribution under the MIT license. View license.md for additional details.