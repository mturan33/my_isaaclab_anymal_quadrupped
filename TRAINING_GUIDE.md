# Anymal C PPO Eğitim Rehberi / Training Guide

Bu proje Anymal C dört ayaklı robotunu PPO (Proximal Policy Optimization) algoritması ile eğitmek için hazırlanmıştır.

This project is designed to train the Anymal C quadruped robot using the PPO (Proximal Policy Optimization) algorithm.

## Ortam Türleri / Environment Types

1. **Flat (Düz Zemin)**: `Isaac-MyAnymal-Flat-v0`
   - Düz zemin üzerinde hareket
   - Daha hızlı eğitim (500 iterasyon)
   - 48 boyutlu gözlem uzayı

2. **Rough (Engebeli Zemin)**: `Isaac-MyAnymal-Rough-v0`
   - Engebeli, zorlu arazi
   - Daha uzun eğitim (1500 iterasyon)
   - 235 boyutlu gözlem uzayı (yükseklik tarayıcı sensörü dahil)

## Eğitim Nasıl Başlatılır / How to Start Training

### Yöntem 1: Kolaylaştırılmış Script (Recommended)

```bash
# Düz zemin üzerinde eğitim
python train_anymal_ppo.py --terrain flat

# Engebeli zemin üzerinde eğitim
python train_anymal_ppo.py --terrain rough

# Daha fazla paralel ortam ile eğitim
python train_anymal_ppo.py --terrain flat --num_envs 128

# Headless modda (GUI olmadan)
python train_anymal_ppo.py --terrain flat --headless

# Checkpoint'ten devam etme
python train_anymal_ppo.py --terrain flat --resume
```

### Yöntem 2: Isaac Lab Script'i ile Doğrudan

```bash
# RSL-RL ile düz zemin eğitimi
python <ISAAC_LAB_PATH>/source/isaaclab_rl/rsl_rl/train.py \
    --task Isaac-MyAnymal-Flat-v0 \
    --num_envs 64 \
    --headless

# RSL-RL ile engebeli zemin eğitimi
python <ISAAC_LAB_PATH>/source/isaaclab_rl/rsl_rl/train.py \
    --task Isaac-MyAnymal-Rough-v0 \
    --num_envs 64 \
    --headless
```

### Yöntem 3: RL Games ile

```bash
python <ISAAC_LAB_PATH>/source/isaaclab_rl/rl_games/train.py \
    --task Isaac-MyAnymal-Flat-v0 \
    --headless
```

### Yöntem 4: SKRL ile

```bash
python <ISAAC_LAB_PATH>/source/isaaclab_rl/skrl/train.py \
    --task Isaac-MyAnymal-Flat-v0 \
    --headless
```

## PPO Hiperparametreleri / PPO Hyperparameters

### RSL-RL Konfigürasyonu (Default)

```python
# Network Architecture
Actor Hidden Layers: [512, 256, 128]
Critic Hidden Layers: [512, 256, 128]
Activation: ELU

# PPO Parameters
Learning Rate: 1.0e-3 (adaptive schedule)
Clip Param: 0.2
Entropy Coefficient: 0.005
Value Loss Coefficient: 1.0
Gamma (Discount): 0.99
Lambda (GAE): 0.95
Desired KL: 0.01
Max Grad Norm: 1.0

# Training
Num Steps per Env: 24
Num Learning Epochs: 5
Num Mini Batches: 4
Max Iterations: 500 (flat) / 1500 (rough)
```

## Ödül Fonksiyonu / Reward Function

Aşağıdaki bileşenlerden oluşan çok görevli ödül fonksiyonu:

Multi-task reward function with the following components:

1. **Doğrusal Hız Takibi** (lin_vel): Hedef XY hızını takip etme
2. **Yaw Açısal Hız** (yaw_rate): Dönüş hızını kontrol etme
3. **Z Hız Cezası** (z_vel): Dikey hareket cezası
4. **Açısal Hız Cezası** (ang_vel_xy): X/Y ekseni rotasyon cezası
5. **Eklem Torque Cezası** (dof_torques): Enerji verimliliği
6. **Eklem İvme Cezası** (dof_acc): Yumuşak hareket
7. **Aksiyon Oranı** (action_rate): Hareket yumuşaklığı
8. **Ayak Havada Kalma Süresi** (feet_air_time): Yürüyüş kalitesi
9. **İstenmeyen Temas** (undesired_contacts): Bacak teması cezası
10. **Düz Oryantasyon** (flat_orientation): Düz duruş ödülü

## Gözlem Uzayı / Observation Space

### Flat Terrain (48D)
- Root linear velocity (3D)
- Root angular velocity (3D)
- Projected gravity (3D)
- Velocity commands (3D)
- Joint positions (12D)
- Joint velocities (12D)
- Previous actions (12D)

### Rough Terrain (235D)
- Yukarıdakilerin hepsi + Height scanner data (187D)

## Aksiyon Uzayı / Action Space

- **Boyut**: 12 (her eklem için pozisyon hedefi)
- **Aralık**: [-1, 1] → [-0.5, 0.5] (default pozisyondan sapma)
- **Kontrol**: Pozisyon kontrolü (torque geri beslemeli)

## Sonuçları Görüntüleme / Viewing Results

Eğitim sırasında TensorBoard ile ilerlemeyi takip edebilirsiniz:

```bash
tensorboard --logdir logs/rsl_rl/my_anymal_flat_direct
```

## Eğitilmiş Modeli Test Etme / Testing Trained Model

```bash
python <ISAAC_LAB_PATH>/source/isaaclab_rl/rsl_rl/play.py \
    --task Isaac-MyAnymal-Flat-v0 \
    --num_envs 16 \
    --load_run <run_name>
```

## Dosya Yapısı / File Structure

```
my_quadruped_locomotion_isaaclab_anymal/
├── __init__.py                      # Gym environment registration
├── my_anymal_env.py                 # Main environment implementation
├── my_anymal_env_cfg.py             # Environment configurations
├── train_anymal_ppo.py              # Training wrapper script
├── robots/
│   └── anymal.py                    # Robot configuration
└── agents/
    ├── rsl_rl_ppo_cfg.py            # RSL-RL PPO configuration
    ├── rl_games_flat_ppo_cfg.yaml   # RL-Games flat terrain config
    ├── rl_games_rough_ppo_cfg.yaml  # RL-Games rough terrain config
    ├── skrl_flat_ppo_cfg.yaml       # SKRL flat terrain config
    └── skrl_rough_ppo_cfg.yaml      # SKRL rough terrain config
```

## Sorun Giderme / Troubleshooting

### Import Hatası
```
Error: Could not import Isaac Lab training module
```
**Çözüm**: Isaac Lab'in doğru kurulduğundan ve Python ortamının aktif olduğundan emin olun.

### CUDA/GPU Hatası
Eğer GPU kullanmıyorsanız, CPU modunda çalıştırabilirsiniz:
```bash
python train_anymal_ppo.py --terrain flat --device cpu
```

### Simülasyon Çok Yavaş
- `--num_envs` sayısını azaltın
- `--headless` modunu kullanın

## Referanslar / References

- [Isaac Lab Documentation](https://isaac-sim.github.io/IsaacLab/)
- [RSL-RL Repository](https://github.com/leggedrobotics/rsl_rl)
- [PPO Paper](https://arxiv.org/abs/1707.06347)
