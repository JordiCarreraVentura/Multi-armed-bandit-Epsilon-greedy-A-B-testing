
# Multi-armed bandit, Epsilon-greedy A-B testing

[Source](http://stevehanov.ca/blog/?id=132)

> - We always keep track of the number of pulls of the lever and the amount of rewards we have received from that lever.
> - 10% of the time, we choose a lever at random.
> - The other 90% of the time, we choose the lever that has the highest expectation of rewards. If several meet this criterion, then we take the first one.
> - We initialize all three choices to 1 win out of 1 try. It doesn't really matter what we initialize them too, because the algorithm will adapt.
> - The randomization of 10% of trials forces the algorithm to explore the options. It is a trade-off between trying new things in hopes of something better, and sticking with what it knows will work. There are several variations of the epsilon-greedy strategy. In the epsilon-first strategy, you can explore 100% of the time in the beginning and once you have a good sample, switch to pure-greedy. 


```python
import random

# A UI object that presents a button with value X.
# A UI action, with the user's action (click on the UI object or no click).
# Add 1 to shown for value X.
# If click, add 1 to chosen for value X.
# If no click, continue.
# Print updated probs at user action T.


class EpsilonGreedy:
    
    def __init__(
        self,
        actions,
        randomization=0.1
    ):
        self.actions = [action for action in actions]
        self.randomization = randomization
        self.shown = {
            action: 0
            for action in self.actions
        }
        self.chosen = {
            action: 0
            for action in self.actions
        }
    
    def __call__(self):
        ratios = [
            (action, self.chosen[action] / float(self.shown[action]) if self.shown[action] else 0.0)
            for action in self.actions
        ]
        randomization_prob = random.random()
        if randomization_prob <= self.randomization:
            choice = random.choice(self.actions)
            return choice
        ratios.sort(key=lambda x: x[1])
        return ratios[-1][0]
    
    def __add__(self, observation):
        """observation: tuple: <option, action>
        option: int
        action: bool"""
        action, outcome = observation
        self.shown[action] += 1
        self.chosen[action] += 1 if outcome else 0
    
    def __str__(self):
        return '\n'.join(
            '%d:%d/%d (%.2f)' % (
                action,
                self.chosen[action],
                self.shown[action],
                (self.chosen[action] / float(self.shown[action])) if self.shown[action] else 0.0
            )
            for action in self.actions
        )
```


```python
SETTINGS = [0, 1, 2]

USERS = 10000

BIASES = [
    # option, click prob(, no-click prob: 1 - (click prob))
    (0, 0.25),
    (1, 0.1),
    (2, 0.5)
] + [
    (option, random.random())
    for option in range(3, random.randrange(6, 11))
]

OPTIONS = zip(*BIASES)[0]

PROBS = dict(BIASES)
```


```python

abeg = EpsilonGreedy(OPTIONS, randomization=0.2)

for user in range(USERS):
    choice = abeg()
    p = random.random()
    abeg + (choice, True if p < PROBS[choice] else False)
    if not user % int(10000 / 5):
        print abeg
        print

for bias in BIASES:
    print bias
```

    0:0/0 (0.00)
    1:0/0 (0.00)
    2:0/0 (0.00)
    3:0/0 (0.00)
    4:0/0 (0.00)
    5:0/0 (0.00)
    6:0/1 (0.00)
    
    0:14/71 (0.20)
    1:8/62 (0.13)
    2:28/57 (0.49)
    3:1387/1596 (0.87)
    4:11/83 (0.13)
    5:10/51 (0.20)
    6:8/81 (0.10)
    
    0:29/129 (0.22)
    1:11/119 (0.09)
    2:54/113 (0.48)
    3:2830/3238 (0.87)
    4:16/141 (0.11)
    5:24/129 (0.19)
    6:13/132 (0.10)
    
    0:38/181 (0.21)
    1:14/172 (0.08)
    2:81/170 (0.48)
    3:4304/4906 (0.88)
    4:22/197 (0.11)
    5:36/194 (0.19)
    6:15/181 (0.08)
    
    0:49/244 (0.20)
    1:22/235 (0.09)
    2:112/228 (0.49)
    3:5796/6563 (0.88)
    4:27/244 (0.11)
    5:50/252 (0.20)
    6:23/235 (0.10)
    
    (0, 0.25)
    (1, 0.1)
    (2, 0.5)
    (3, 0.8830050014050395)
    (4, 0.12134639115909485)
    (5, 0.2129897336803438)
    (6, 0.1107805589958264)

