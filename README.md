# 🎮 Quality- and Diversity-Aware Super Mario Level Generation Using TD3

A Reinforcement Learning-based Procedural Content Generation project inspired by the research paper:

> **“Using Reinforcement Learning to Generate Levels of Super Mario Bros. with Quality and Diversity”**
> Nam et al., IEEE Transactions on Games

This project explores how **Deep Reinforcement Learning (DRL)** can be used to automatically generate Super Mario Bros.-style levels while considering both **level quality** and **level diversity**.

The implementation uses **Twin Delayed Deep Deterministic Policy Gradient (TD3)** together with a **Conditional GAN (CGAN)**, randomized initialization, and a diversity-aware greedy policy.

---

## 📌 Project Overview

Traditional procedural level generation often relies on manually designed rules or random generation. Such methods can produce playable levels, but controlling properties such as **difficulty, playability, and diversity** can be challenging.

This project formulates Mario level generation as a **Reinforcement Learning problem**.

The RL agent generates a sequence of actions. These actions are converted into level patterns using a Conditional GAN, and the generated patterns are evaluated according to several quality criteria.

The system attempts to generate levels that are:

* ✅ Playable
* 🎯 Appropriately difficult
* 🔄 Less monotonous
* 🌈 Diverse from one another

---

## 🧠 Main Idea

The overall generation pipeline is:

```text
Super Mario Level Dataset
          │
          ▼
   Level Preprocessing
          │
          ▼
  15 × 4 Level Patterns
          │
          ▼
     Conditional GAN
          │
          ▼
     Pattern Generation
          │
          ▼
    Pattern Matching
          │
          ▼
      Mario Level
          │
          ▼
       Evaluation
    ┌─────┼─────┐
    ▼     ▼     ▼
Playability Difficulty Monotony
    │     │     │
    └─────┼─────┘
          ▼
       Reward
          │
          ▼
        TD3
          │
          ▼
   Better Level Generation
```

For diversity, the project additionally uses:

```text
Randomized Initialization (RI)
             +
Diversity-Aware Greedy Policy (DAGP)
             │
             ▼
     More Diverse Levels
```

---

# 📚 Research Paper

The project is based on the ideas presented in:

**Using Reinforcement Learning to Generate Levels of Super Mario Bros. with Quality and Diversity**

The paper formulates procedural content generation as a Markov Decision Process and uses TD3 to generate Mario level patterns.

Important concepts adopted from the paper include:

* MDP-based level generation
* 15 × 4 level patterns
* Conditional GAN
* Pattern matching
* Playability evaluation
* Difficulty evaluation
* Monotony evaluation
* TD3
* Virtual Simulation concept
* Randomized Initialization
* Diversity-Aware Greedy Policy
* KL-divergence-based diversity evaluation

---

# 🗂️ Dataset

This implementation uses the publicly available **Video Game Level Corpus (VGLC)** as the source of Super Mario Bros. level data.

The dataset provides text-based representations of game levels, which can be converted into structured tile representations.

The project automatically downloads the required dataset during notebook execution.

Dataset:

**The Video Game Level Corpus (VGLC)**

Repository:

https://github.com/TheVGLC/TheVGLC

---

# 🧩 Level Representation

Each Mario level is converted into a three-channel representation.

```text
Channel 1 → Walls / Solid Tiles
Channel 2 → Enemies
Channel 3 → Holes / Empty Ground
```

The representation has the form:

```text
(3, 15, Width)
```

where:

* `3` = number of channels
* `15` = level height
* `Width` = level width

The level is divided into sliding windows of:

```text
15 × 4
```

Therefore, each pattern has the shape:

```text
(3, 15, 4)
```

---

# 🤖 Conditional GAN

A Conditional Generative Adversarial Network is used to generate level patterns.

The generator receives:

```text
Action vector
+
Previous level patterns
```

and produces:

```text
15 × 4 Mario level pattern
```

Conceptually:

```text
TD3 Action
   │
   ▼
┌───────────────┐
│    CGAN       │
│   Generator   │
└───────┬───────┘
        │
        ▼
   Generated
    Pattern
```

The discriminator attempts to distinguish generated patterns from patterns obtained from the dataset.

---

# 🎮 Reinforcement Learning

The project uses **TD3 (Twin Delayed Deep Deterministic Policy Gradient)**.

TD3 is an actor-critic reinforcement learning algorithm designed for continuous action spaces.

The actor produces a continuous action:

```text
a = [a1, a2, a3]
```

where each value is approximately in:

```text
[0, 1]
```

The action is then provided to the conditional generator.

---

# 🧠 TD3 Architecture

The implemented network follows the general architecture inspired by the paper:

```text
Input
  │
  ▼
Conv 32
  │
  ▼
Conv 32
  │
  ▼
Pooling
  │
  ▼
Conv 64
  │
  ▼
Conv 64
  │
  ▼
Pooling
  │
  ▼
Fully Connected 256
  │
  ▼
Fully Connected 128
  │
  ▼
Fully Connected 128
  │
  ▼
Output
```

---

# 🎯 Reward Function

The reward combines multiple properties of a generated level.

The major components are:

### 1. Playability

A generated level should be possible to traverse.

The project uses a grid-based evaluation inspired by the paper's A*-based playability evaluation.

---

### 2. Difficulty

Difficulty is estimated using a stochastic Mario-like agent.

The evaluation considers hazards such as:

* Enemies
* Holes

A virtual damage measure is used to estimate the difficulty experienced by the agent.

---

### 3. Monotony

The project evaluates whether consecutive patterns are too similar.

The monotony calculation considers similarity between:

```text
Current pattern
Previous pattern
Pattern from 2 steps ago
Pattern from 3 steps ago
Pattern from 4 steps ago
```

This discourages repetitive level structures.

---

# 🔄 Randomized Initialization

Randomized Initialization (RI) is used to increase diversity between generated levels.

Instead of always starting the generation process from exactly the same initial state, different starting patterns can be selected.

Conceptually:

```text
             ┌── Start Pattern A
             │
Random Start ├── Start Pattern B
             │
             └── Start Pattern C
                    │
                    ▼
                   TD3
                    │
                    ▼
             Generated Levels
```

---

# 🌈 Diversity-Aware Greedy Policy

The project also implements a **Diversity-Aware Greedy Policy (DAGP)**.

Instead of simply selecting the action with the highest Q-value, multiple candidate actions are considered.

Candidates that are too similar to an already selected action can be rejected using a diversity threshold.

Conceptually:

```text
Candidate Actions
      │
      ├── Candidate 1
      ├── Candidate 2
      ├── Candidate 3
      ├── Candidate 4
      └── Candidate 5
              │
              ▼
       Diversity Check
              │
       ┌──────┴──────┐
       │             │
    Similar       Different
       │             │
    Reject          Keep
                     │
                     ▼
                Highest Q
                     │
                     ▼
              Selected Action
```

---

# 📊 Diversity Evaluation

The project evaluates generated levels using two main measures.

## Different Tile Ratio

The percentage of tiles that differ between two generated levels.

A higher value indicates greater structural difference between the compared levels.

---

## KL Divergence

KL divergence is used to compare the tile distributions of two levels.

The implementation calculates:

```text
KL(P || Q)
```

where `P` and `Q` represent the tile distributions of two levels.

Higher divergence indicates greater difference between their tile distributions.

---

# 🧪 Experiments

The notebook compares three generation strategies:

### 1. Random

Levels generated using randomly selected actions/patterns.

### 2. RL-RI

TD3-based generation with Randomized Initialization.

### 3. RL-RI + DAGP

TD3-based generation using:

* Randomized Initialization
* Diversity-Aware Greedy Policy

The generated levels are compared using both quality and diversity metrics.

---

# 📈 Evaluation Metrics

## Quality Metrics

The project evaluates:

| Metric       | Purpose                            |
| ------------ | ---------------------------------- |
| Playability  | Whether the level can be traversed |
| Difficulty   | Estimated challenge of the level   |
| Monotony     | Amount of repetitive structure     |
| Total Reward | Combined RL objective              |

## Diversity Metrics

| Metric               | Purpose                               |
| -------------------- | ------------------------------------- |
| Different Tile Ratio | Structural difference between levels  |
| KL Divergence        | Difference between tile distributions |

---

# 🛠️ Technologies Used

* Python
* NumPy
* Pandas
* PyTorch
* Matplotlib
* Jupyter Notebook
* Reinforcement Learning
* TD3
* Conditional GAN
* Procedural Content Generation
* Super Mario Bros. level representation
* KL Divergence

---

# 📁 Project Structure

```text
Super-Mario-PCGRL/
│
├── Super_Mario_PCGRL_TD3_Paper_Reproduction.ipynb
│
├── README.md
│
├── generated_levels/
│   ├── random/
│   ├── rl_ri/
│   └── rl_dagp/
│
├── results/
│   ├── quality_results.csv
│   ├── diversity_results.csv
│   └── plots/
│
└── models/
    ├── actor.pt
    ├── critic1.pt
    └── critic2.pt
```

The exact generated files may vary depending on the notebook configuration.

---

# 💻 Installation

Clone the repository:

```bash
git clone <YOUR_GITHUB_REPOSITORY_URL>
cd Super-Mario-PCGRL
```

Install the required Python packages:

```bash
pip install numpy pandas matplotlib scipy scikit-learn torch torchvision tqdm requests
```

Launch Jupyter Notebook:

```bash
jupyter notebook
```

Then open:

```text
Super_Mario_PCGRL_TD3_Paper_Reproduction.ipynb
```

---

# ▶️ How to Run

Run the notebook cells **sequentially from top to bottom**.

The notebook performs the following steps:

```text
1. Install / import dependencies
        ↓
2. Download VGLC dataset
        ↓
3. Parse Mario levels
        ↓
4. Convert levels into semantic channels
        ↓
5. Extract 15 × 4 patterns
        ↓
6. Build legal pattern database
        ↓
7. Train Conditional GAN
        ↓
8. Implement pattern matching
        ↓
9. Implement Mario level evaluation
        ↓
10. Implement TD3
        ↓
11. Train RL generator
        ↓
12. Generate levels
        ↓
13. Apply Randomized Initialization
        ↓
14. Apply DAGP
        ↓
15. Evaluate quality
        ↓
16. Evaluate diversity
        ↓
17. Generate comparison plots
```

---

# ⚙️ Training Configuration

For initial testing, the notebook uses a relatively small number of TD3 episodes so that the complete pipeline can be verified on a normal computer.

For example:

```python
TD3_EPISODES = 75
```

After confirming that the implementation works, the number of training episodes can be increased.

For example:

```python
TD3_EPISODES = 300
```

or:

```python
TD3_EPISODES = 500
```

Higher episode counts increase training time.

---

# ⚠️ Important Implementation Note

This project is a **research-paper-inspired reproduction**, not an exact numerical replication of the original paper.

The original paper uses its own experimental setup, including its original pattern dataset and Mario simulation/evaluation environment.

This implementation adapts the methodology to:

* Publicly available VGLC data
* A self-contained Conditional GAN
* A lightweight Mario-style grid simulator
* Python/PyTorch implementation suitable for notebook execution

Therefore, the numerical results from this project should **not be presented as the exact results reported in the original research paper**.

The appropriate description is:

> “A self-contained implementation inspired by the PCGRL framework proposed in the research paper, adapted to the publicly available VGLC dataset and a lightweight Mario level simulator.”

---

# 📌 Research Contribution

The project demonstrates how reinforcement learning can be combined with generative models for procedural game-level generation.

The main workflow combines:

```text
Conditional GAN
        +
TD3
        +
Randomized Initialization
        +
Diversity-Aware Policy
        +
Quality Evaluation
        +
Diversity Evaluation
```

This provides a framework for generating multiple Mario-style levels while evaluating both their gameplay-related properties and structural diversity.

---

# 🚀 Possible Future Improvements

The project can be extended in several directions:

### Better Mario Simulation

Replace the lightweight simulator with a more complete Mario game environment.

### A* Player Agent

Implement a closer reproduction of the A*-based player agent used in the original research.

### Virtual Simulation TD3

Implement the full **VS-TD3** training procedure described in the paper.

### More Evaluation Criteria

Additional level properties could be incorporated, such as:

* Coins
* Power-ups
* Enemy density
* Platform distribution
* Level length
* Jump difficulty

### Human Evaluation

Generated levels can be tested by human players and compared with automated difficulty measurements.

### Larger Training

Increasing the number of training episodes can provide more opportunities for the RL agent to learn useful generation policies.

---

# 📖 References

1. Nam et al., **“Using Reinforcement Learning to Generate Levels of Super Mario Bros. with Quality and Diversity,” IEEE Transactions on Games.**

2. The Video Game Level Corpus (VGLC):
   https://github.com/TheVGLC/TheVGLC

3. Mario AI Framework:
   https://github.com/amidos2006/Mario-AI-Framework

---

# 👨‍💻 Author

**Suraj Ravindra Khade**

B.E. Artificial Intelligence & Data Science
Smt. Indira Gandhi College of Engineering
University of Mumbai

---

## ⭐ Project Summary

> **This project implements a reinforcement-learning-based procedural content generation system for Super Mario Bros.-style levels. TD3 generates continuous actions that condition a GAN-based pattern generator, while playability, difficulty, and monotony are used to evaluate level quality. Randomized Initialization and a Diversity-Aware Greedy Policy are incorporated to generate more varied levels, with diversity measured using different-tile ratios and KL divergence.**
