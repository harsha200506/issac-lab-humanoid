# 🤖 Isaac Lab Humanoid Robot Training

Training humanoid robots using NVIDIA Isaac Lab and Reinforcement Learning (RL) — from scratch to walking!

![Isaac Lab](https://img.shields.io/badge/Isaac%20Lab-2.3.2-green)
![Isaac Sim](https://img.shields.io/badge/Isaac%20Sim-5.1.0-blue)
![Python](https://img.shields.io/badge/Python-3.11-yellow)
![GPU](https://img.shields.io/badge/GPU-RTX%205080-red)

---

## 🧠 What is This?

This project uses **Reinforcement Learning (RL)** to teach humanoid robots how to walk in simulation using NVIDIA Isaac Lab. The robot learns entirely on its own — no hand-coded walking movements. It figures out how to balance and walk just by receiving rewards and penalties.

Robots trained in this project:
- 🧍 **MuJoCo Humanoid** — Classic humanoid with custom slow stable walking
- 🤖 **Unitree G1** — Real-world humanoid robot model

---

## ⚙️ My Setup

| Component | Details |
|---|---|
| GPU | NVIDIA GeForce RTX 5080 |
| CPU | AMD Ryzen 9 9950X3D |
| RAM | 64 GB |
| OS | Ubuntu 24.04 |
| Isaac Lab | v2.3.2 |
| Isaac Sim | 5.1.0 |
| Python | 3.11 |
| RL Framework | SKRL + RSL-RL |

---

## 📁 Files in This Repo

| File | What it does |
|---|---|
| `humanoid_env_cfg.py` | Modified reward function for slow stable walking |
| `unitree.py` | Unitree robot configs with local G1 USD path fix |
| `README.md` | This file |

---

## 🏆 Reward Function (The Key Change)

The default humanoid sprints and falls. I modified the rewards to get **slow, stable, natural walking**:

| Reward Term | Default | Mine | Effect |
|---|---|---|---|
| `alive` | `2.0` | `5.0` | Survival is top priority |
| `progress` | `1.0` | `0.3` | Walk slowly, don't sprint |
| `upright` | `0.1` | `2.0` | Stay balanced and straight |
| `move_to_target` | `0.5` | `0.3` | Gentle forward direction |
| `action_l2` | `-0.01` | `-0.05` | Smoother, calmer movements |
| `energy` | `-0.005` | `-0.02` | Less energy waste |
| `joint_pos_limits` | `-0.25` | `-0.5` | Avoid extreme joint angles |
| `torso_height` min | `0.8m` | `0.9m` | Must stay more upright |
| `episode_length` | `16s` | `20s` | More time to learn balance |

---

## 🚀 How to Train

### Prerequisites
```bash
conda create -n env_isaaclab python=3.11
conda activate env_isaaclab
pip install -U torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128
pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com
git clone https://github.com/isaac-sim/IsaacLab.git
cd IsaacLab
./isaaclab.sh --install
./isaaclab.sh -i skrl
./isaaclab.sh -i rsl_rl
```

### Train Humanoid (Custom Slow Walk)
```bash
cd ~/isaac/IsaacLab
./isaaclab.sh -p scripts/reinforcement_learning/skrl/train.py \
  --task Isaac-Humanoid-v0 \
  --num_envs 2048 \
  --headless
```

### Watch Humanoid Walk
```bash
./isaaclab.sh -p scripts/reinforcement_learning/skrl/play.py \
  --task Isaac-Humanoid-v0 \
  --num_envs 32 \
  --checkpoint $(ls -t logs/skrl/humanoid/*/checkpoints/*.pt | head -1)
```

### Train Unitree G1
```bash
./isaaclab.sh -p scripts/reinforcement_learning/rsl_rl/train.py \
  --task Isaac-Velocity-Flat-G1-v0 \
  --num_envs 4096 \
  --headless
```

### Watch G1 Walk
```bash
./isaaclab.sh -p scripts/reinforcement_learning/rsl_rl/play.py \
  --task Isaac-Velocity-Flat-G1-Play-v0 \
  --num_envs 32 \
  --checkpoint $(ls -t logs/rsl_rl/g1_flat/*/model_*.pt | head -1)
```

---

## 🧠 How Reinforcement Learning Works

```
1. Robot does a random action
         ↓
2. Physics simulates what happens
         ↓
3. Reward function scores the result
   +5.0  staying upright
   +0.3  moving forward
   -0.05 jerky movements
         ↓
4. Neural network learns:
   "More of what got rewards"
   "Less of what got penalties"
         ↓
5. Repeat millions of times across
   4096 parallel environments
```

---

## 📈 Results

| Robot | Training Time | Environments | Result |
|---|---|---|---|
| Cartpole | 2 seconds | 16 | Balances perfectly |
| Humanoid | ~45 min | 2048 | Slow stable walking |
| Unitree G1 | ~60 min | 4096 | Natural locomotion |

---

## 🗺️ What's Next

- [ ] Add domain randomization for sim-to-real transfer
- [ ] Train G1 on rough terrain
- [ ] Try AMP motion capture walking
- [ ] Export policy to ONNX
- [ ] Deploy on physical robot

---

## 📚 References

- [Isaac Lab Documentation](https://isaac-sim.github.io/IsaacLab)
- [NVIDIA Isaac Lab GitHub](https://github.com/isaac-sim/IsaacLab)
- [Unitree Robotics](https://www.unitree.com)
- [SKRL Documentation](https://skrl.readthedocs.io)
- [RSL-RL](https://github.com/leggedrobotics/rsl_rl)

---

## 🙏 Acknowledgements

Built on top of [Isaac Lab](https://github.com/isaac-sim/IsaacLab) by NVIDIA.
Robot models provided by [Unitree Robotics](https://www.unitree.com).
