# Paper

* **Title**: Hierarchical Deep Reinforcement Learning: Integrating Temporal Abstraction and Intrinsic Motivation
* **Authors**: Tejas D. Kulkarni, Karthik R. Narasimhan, Ardavan Saeedi, Joshua B. Tenenbaum
* **Link**: https://arxiv.org/abs/1604.06057
* **Tags**: Neural Network, reinforcement learning
* **Year**: 2016

# Summary

* What
  * They present a hierarchical method for reinforcement learning.
  * The method combines "long"-term goals with short-term action choices.

* How
  * They have two components:
    * Meta-Controller:
      * Responsible for the "long"-term goals.
      * Is trained to pick goals (based on the current state) that maximize (extrinsic) rewards, just like you would usually optimize to maximize rewards by picking good actions.
      * The Meta-Controller only picks goals when the Controller terminates or achieved the goal.
    * Controller:
      * Receives the current state and the current goal.
      * Has to pick a reward maximizing action based on those, just as the agent would usually do (only the goal is added here).
      * The reward is intrinsic. It comes from the Critic. The Critic gives reward whenever the current goal is reached.
  * For Montezuma's Revenge:
    * A goal is to reach a specific object.
    * The goal is encoded via a bitmask (as big as the game screen). The mask contains 1s wherever the object is.
    * They hand-extract the location of a few specific objects.
    * So basically:
      * The Meta-Controller picks the next object to reach via a Q-value function.
      * It receives extrinsic reward when objects have been reached in a specific sequence.
      * The Controller picks actions that lead to reaching the object based on a Q-value function. It iterates action-choosing until it terminates or reached the goal-object.
      * The Critic awards intrinsic reward to the Controller whenever the goal-object was reached.
    * They use CNNs for the Meta-Controller and the Controller, similar in architecture to the Atari-DQN paper (shallow CNNs).
    * They use two replay memories, one for the Meta-Controller (size 40k) and one for the Controller (size 1M).
    * Both follow an epsilon-greedy policy (for picking goals/actions). Epsilon starts at 1.0 and is annealed down to 0.1.
    * They use a discount factor / gamma of 0.9.
    * They train with SGD. 

* Results
  * Learns to play Montezuma's Revenge.
  * Learns to act well in a more abstract MDP with delayed rewards and where simple Q-learning failed.

--------------------

# Rough chapter-wise notes


* (1) Introduction
  * Basic problem: Learn goal directed behaviour from sparse feedbacks.
  * Challenges:
    * Explore state space efficiently
    * Create multiple levels of spatio-temporal abstractions
  * Their method: Combines deep reinforcement learning with hierarchical value functions.
  * Their agent is motivated to solve specific intrinsic goals.
  * Goals are defined in the space of entities and relations, which constraints the search space.
  * They define their value function as V(s, g) where s is the state and g is a goal.
  * First, their agent learns to solve intrinsically generated goals. Then it learns to chain these goals together.
  * Their model has two hiearchy levels:
    * Meta-Controller: Selects the current goal based on the current state.
    * Controller: Takes state s and goal g, then selects a good action based on s and g. The controller operates until g is achieved, then the meta-controller picks the next goal.
  * Meta-Controller gets extrinsic rewards, controller gets intrinsic rewards.
  * They use SGD to optimize the whole system (with respect to reward maximization).

* (3) Model
  * Basic setting: Action a out of all actions A, state s out of S, transition function T(s,a)->s', reward by state F(s)->R.
  * epsilon-greedy is good for local exploration, but it's not good at exploring very different areas of the state space.
  * They use intrinsically motivated goals to better explore the state space.
  * Sequences of goals are arranged to maximize the received extrinsic reward.
  * The agent learns one policy per goal.
  * Meta-Controller: Receives current state, chooses goal.
  * Controller: Receives current state and current goal, chooses action. Keeps choosing actions until goal is achieved or a terminal state is reached. Has the optimization target of maximizing cumulative reward.
  * Critic: Checks if current goal is achieved and if so provides intrinsic reward.
  * They use deep Q learning to train their model.
  * There are two Q-value functions. One for the controller and one for the meta-controller.
  * Both formulas are extended by the last chosen goal g.
  * The Q-value function of the meta-controller does not depend on the chosen action.
  * The Q-value function of the controller receives only intrinsic direct reward, not extrinsic direct reward.
  * Both Q-value functions are reprsented with DQNs.
  * Both are optimized to minimize MSE losses.
  * They use separate replay memories for the controller and meta-controller.
  * A memory is added for the meta-controller whenever the controller terminates.
  * Each new goal is picked by the meta-controller epsilon-greedy (based on the current state).
  * The controller picks actions epsilon-greedy (based on the current state and goal).
  * Both epsilons are annealed down.

* (4) Experiments
  * (4.1) Discrete MDP with delayed rewards
    * Basic MDP setting, following roughly: Several states (s1 to s6) organized in a chain. The agent can move left or right. It gets high reward if it moves to state s6 and then back to s1, otherwise it gets small reward per reached state.
    * They use their hierarchical method, but without neural nets.
    * Baseline is Q-learning without a hierarchy/intrinsic rewards.
    * Their method performs significantly better than the baseline.
  * (4.2) ATARI game with delayed rewards
    * They play Montezuma's Revenge with their method, because that game has very delayed rewards.
    * They use CNNs for the controller and meta-controller (architecture similar to the Atari-DQN paper).
    * The critic reacts to (entity1, relation, entity2) relationships. The entities are just objects visible in the game. The relation is (apparently ?) always "reached", i.e. whether object1 arrived at object2.
    * They extract the objects manually, i.e. assume the existance of a perfect unsupervised object detector.
    * They encode the goals apparently not as vectors, but instead just use a bitmask (game screen heightand width), which has 1s at the pixels that show the object.
    * Replay memory sizes: 1M for controller, 50k for meta-controller.
    * gamma=0.99
    * They first only train the controller (i.e. meta-controller completely random) and only then train both jointly.
    * Their method successfully learns to perform actions which lead to rewards with long delays.
    * It starts with easier goals and then learns harder goals.
