<div align="center">

<img src="./pyker_engine/resources/images/black_red_logo.svg" alt="Logo" width=200px>

</div>

# pyker_engine

pyker_engine is a python library for running arbitrary configurations of community card poker games. This includes anything from simple Leduc or [Kuhn](https://en.wikipedia.org/wiki/Kuhn_poker) poker to full n-player [No Limit Texas Hold'em](https://en.wikipedia.org/wiki/Texas_hold_%27em) or [Pot Limit Omaha](https://en.wikipedia.org/wiki/Omaha_hold_%27em#Pot-limit_Omaha).

## Example

```python
import random

import pyker_engine

config = pyker_engine.configs.NOLIMIT_HOLDEM_6P_ENV
dealer = pyker_engine.Dealer(**config)
obs = dealer.reset()

while True:
    call = obs['call']
    min_raise = obs['min_raise']
    max_raise = obs['max_raise']

    rand = random.random()
    if rand < 0.1:
        bet = 0
    elif rand < 0.80:
        bet = call
    else:
        bet = random.randint(min_raise, max_raise)

    obs, rewards, done = dealer.step(bet)
    if all(done):
        break

print(rewards)
```

## Configuration

The type of poker game is defined using a configuration dictionary. See [configs.py](./pyker_engine/configs.py) for some example configurations. A configuration dictionary has to have the following key value structure:
* num_players
  * int: maximum number of players
* num_streets
  * int: number of streets
* blinds
  * int or list of ints: the blind distribution starting from the button e.g. [0, 1, 2, 0, 0, 0] for a 6 player 1-2 game
  * a single int is expanded to the number of players, settings blinds=0 will result in no blinds [0, 0, 0, 0, 0, 0]
* antes
  * int or list of ints: the ante distribution starting from the button, analog to the blind distribution
  * single ints are expanded to the number of players
* raise_sizes
  * float or str or list of floats or str: the maximum raise size as a list of values, one for each street
  * options are ints (for fixed raise sizes), float('inf') (for no limit raise sizes) or 'pot' for pot limit raise sizes
  * single values are expanded to the number of streets
* num_raises
  * float or list of floats: the maximum number of raises for each street
  * options are ints (for a fixed number of bets per round) or float('inf') for unlimited number of raises
  * single values are expanded to the number of streets
* num_suits
  * number of suits in the deck
* num_ranks
  * number of ranks in the deck
* num_hole_cards
  * number of hole cards for each player
* num_community_cards
  * number of community cards per street
* num_cards_for_hand
  * number of cards for a valid poker hand
* mandatory_num_hole_cards
  * number of hole cards which must be used for a poker hand
* start_stack
  * initial stack size

## API

pyker_engine adopts the [Open AI gym](https://github.com/openai/gym) interface. See [pyker_engine gym](https://github.com/fschlatt/pyker_gym.git) for a full pyker_engine gym environment. To deal a new hand, call `dealer.reset()`, which returns a dictionary of observations for the current active player. To advance the game, call `dealer.step({bet})` with an integer bet size. Invalid bet sizes are always rounded to the nearest valid bet size. When the bet lies exactly between 2 valid bet sizes, it is always rounded down. For example, if the minimum raise size is 10 and the bet is 5, the bet is rounded down to 0, i.e. call or fold.

## Universal Deuces

The hand evaluator is heavily inspired by the [deuces](https://github.com/worldveil/deuces/) library. The basic logic is identical, but the evaluator and lookup table are generalized to work for any deck configuration with number of ranks <= 13 and number of suits <= 4 and poker hands with 5 or less cards. See the poker [README](./pyker_engine/poker/README.md) for further details.

## Limitations

Even though the library is aimed to be as general as possible, it currently has a couple limitations:
* Only integer chip denominations are supported 
* The evaluator only works for sub decks of the standard 52 card deck as well as a maximum of 5 card poker hands
* Draw and stud poker games are not supported 