# Gym Interface
FFAI implements the Open AI Gym interace for easy integration of machine learning algorithms.

Take a look at our [gym_example.py](examples/gym_example.py).

![FFAI Gym GUI](screenshots/gym.png?raw=true "FFAI Gym GUI")

## Observations
Observations are split in three parts:
1. 'board': two-dimensional feature leayers
2. 'state': a vector of normalized values (e.g. turn number, half, scores etc.) describing the game state
3. 'procedure' a one-hot vector describing which of 18 procedures the game is in. The game engine is structered as a stack of procedures. The top-most procedure in the stack is active.

### Observation: 'board'
The default feature layers in obs['board'] are:

0. OccupiedLayer()
1. OwnPlayerLayer()
2. OppPlayerLayer()
3. OwnTackleZoneLayer()
4. OppTackleZoneLayer()
5. UpLayer()
6. StunnedLayer()
7. UsedLayer()
8. AvailablePlayerLayer()
9. AvailablePositionLayer()
10. RollProbabilityLayer()
11. BlockDiceLayer()
12. ActivePlayerLayer()
13. TargetPlayerLayer()
14. MALayer()
15. STLayer()
16. AGLayer()
17. AVLayer()
18. MovemenLeftLayer()
19. BallLayer()
20. OwnHalfLayer()
21. OwnTouchdownLayer()
22. OppTouchdownLayer()
23. SkillLayer(Skill.BLOCK)
24. SkillLayer(Skill.DODGE)
25. SkillLayer(Skill.SURE_HANDS)
26. SkillLayer(Skill.CATCH)
27. SkillLayer(Skill.PASS)

Custom layers can be implemented like this:
```python
from ffai.ai import FeatureLayer
class MyCustomLayer(FeatureLayer):

    def produce(self, game):
        out = np.zeros((game.arena.height, game.arena.width))
        for y in range(len(game.state.pitch.board)):
            for x in range(len(game.state.pitch.board[0])):
                player = game.state.pitch.board[y][x]
                out[y][x] = 1.0 if player is not None and player.role.cost > 80000 else 0.0
        return out

    def name(self):
        return "expensive players"
```
and added to the environment's feature layers:
```python
env.layers.append(MyCustomLayer())
```

To visualize the feature layers, use the feature_layers option when calling render:
```python
env.render(feature_layers=True)
```

![FFAI Gym Feature Layers](screenshots/gym_layers.png?raw=true "FFAI Gym Feature Layers")


### Observation: 'state'
The 50 default normalized values in obs['state'] are:

0. 'half'
1. 'round'
2. 'is sweltering heat'
3. 'is very sunny'
4. 'is nice'
5. 'is pouring rain'
6. 'is blizzard'
7. 'is own turn'
8. 'is kicking first half'
9. 'is kicking this drive'
10. 'own reserves'
11. 'own kods'
12. 'own casualites'
13. 'opp reserves'
14. 'opp kods'
15. 'opp casualties'
16. 'own score'
17. 'own turns'
18. 'own starting rerolls'
19. 'own rerolls left'
20. 'own ass coaches'
21. 'own cheerleaders'
22. 'own bribes'
23. 'own babes'
24. 'own apothecary available'
25. 'own reroll available'
26. 'own fame'
27. 'opp score'
28. 'opp turns'
29. 'opp starting rerolls'
30. 'opp rerolls left'
31. 'opp ass coaches'
32. 'opp cheerleaders'
33. 'opp bribes'
34. 'opp babes'
35. 'opp apothecary available'
36. 'opp reroll available'
37. 'opp fame'
38. 'is blitz available'
39. 'is pass available'
40. 'is handoff available'
41. 'is foul available'
42. 'is blitz'
43. 'is quick snap'
44. 'is move action'
45. 'is block action'
46. 'is blitz action'
47. 'is pass action'
48. 'is handoff action'
49. 'is foul action'

### Observation: 'procedure'
The 19 procedures represented in the one-hot vector obs['procedure'] are:

0. StartGame
1. CoinTossFlip
2. CoinTossKickReceive
3. Setup
4. PlaceBall
5. HighKick
6. Touchback
7. Turn
8. PlayerAction
9. Block
10. Push
11. FollowUp
12. Apothecary
13. PassAction
14. Catch
15. Interception
16. GFI
17. Dodge
18. Pickup

## Action Types
Actions consists of 31 action types. Some action types, denoted by `<position>` also requires an x and y-coordinate.

0. ActionType.START_GAME
1. ActionType.HEADS
2. ActionType.TAILS
3. ActionType.KICK
4. ActionType.RECEIVE
5. ActionType.END_PLAYER_TURN
6. ActionType.USE_REROLL
7. ActionType.DONT_USE_REROLL
8. ActionType.END_TURN
9. ActionType.STAND_UP
10. ActionType.SELECT_ATTACKER_DOWN
11. ActionType.SELECT_BOTH_DOWN
12. ActionType.SELECT_PUSH
13. ActionType.SELECT_DEFENDER_STUMBLES
14. ActionType.SELECT_DEFENDER_DOWN
15. ActionType.SELECT_NONE
16. ActionType.PLACE_PLAYER`<Position>`
17. ActionType.PLACE_BALL`<Position>`
18. ActionType.PUSH`<Position>`
19. ActionType.FOLLOW_UP`<Position>` 
20. ActionType.SELECT_PLAYER`<Position>` (position of the player)
21. ActionType.MOVE`<Position>`
22. ActionType.BLOCK`<Position>`
23. ActionType.PASS`<Position>`
24. ActionType.FOUL`<Position>`
25. ActionType.HANDOFF`<Position>`
24. ActionType.LEAP
25. ActionType.START_MOVE`<Position>` (position of the player)
26. ActionType.START_BLOCK`<Position>` (position of the player)
27. ActionType.START_BLITZ`<Position>` (position of the player)
28. ActionType.START_PASS`<Position>` (position of the player)
29. ActionType.START_FOUL`<Position>` (position of the player)
30. ActionType.START_HANDOFF`<Position>` (position of the player)
31. ActionType.USE_SKILL
32. ActionType.DONT_USE_SKILL
33. ActionType.SETUP_FORMATION_WEDGE
34. ActionType.SETUP_FORMATION_LINE
35. ActionType.SETUP_FORMATION_SPREAD
36. ActionType.SETUP_FORMATION_ZONE

Actions are instantiated and used like this:
```python
action = {
    'action-type': 26,
    'x': 8,
    'y': 6
}
obs, reward, done, info = env.step(action)
```

## Rewards and Info
The default reward function only rewards for a win, draw or loss 1/0/-1.
However, the info object returned by the step function contains useful information for reward shaping:
```python
'cas_inflicted': {int},
'opp_cas_inflicted': {int},
'touchdowns': {int},
'opp_touchdowns': {int},
'half': {int},
'round': {int}
```
These values are commulative, such that 'cas_inflicted' refers to the total number of casualties inflicted by the team.

## Variants
FFAI comes with five environments with various difficulty:
* FFAI-v1: 11 players on a 26x15 pitch (traditional size)
* FFAI-7-v1: 7 players on a 20x11 pitch
* FFAI-5-v1: 5 players on a 16x8 pitch
* FFAI-3-v1: 3 players on a 12x5 pitch
* FFAI-1-v1: 1 player on a 4x3 pitch

This is how the FFAI-3-v1 environment looks:

![FFAI Gym GUI](screenshots/gym_3.png?raw=true "FFAI Gym GUI FFAI-3")

Please note that the rendering has been temporarily disabled due to a bug in tkinter.
