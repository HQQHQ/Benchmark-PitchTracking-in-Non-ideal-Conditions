# Benchmark-PitchTracking-in-Non-ideal-Conditions

## Overview  
This repository contains the code and data pipelines for our **Pitch Tracking Benchmark Project**, developed as part of the Fall 2025 final project for the Music Information Retrieval course at NYU.  
The goal of this project is to evaluate how pitch tracking models behave under different real-world recording conditions. We focus on three types of augmentations—**noise**, **distortion**, and **tuning changes**—and apply them to the MedleyDB-Pitch dataset. The processed audio and updated annotations are then used to benchmark models such as **CREPE** and **BasicPitch**.

---

## Data Processing  

All processing is based on the pitch subset of the **MedleyDB** dataset  
👉 https://medleydb.weebly.com/  

For each augmentation, we maintain the original ratio of vocal and instrumental tracks.  
Vocal and instrumental tracks are separated and evenly split across the augmentation levels or catagories.

The following sections describe the three augmentation pipelines: distortion, noise, and tuning.

---

### 1. Distortion Augmentation  

We apply a tanh-based nonlinear distortion to each stem.  
The core formula used is:

Soft clip: y_out = tanh(g * y_in)

We implement LUFS-based loudness matching to keep perceived volume consistent with the original signal.

We define **three distortion levels**, corresponding to the following gain parameters:

- **Light**: \( g = 2.0 \)  
- **Medium**: \( g = 5.0 \)  
- **Heavy**: \( g = 7.5 \)

Each audio file is distorted once based on the level listed in the dataset manifest.  
File names and folder structures remain unchanged to preserve compatibility with `mirdata`.  
The Distorted dataset can be found [here](https://drive.google.com/drive/folders/1AkUr0t0VZ3CfEQ80TXEfF15WW-LerpeB?usp=sharing)

---

### 2. Noise Augmentation  

We add three types of environmental noise to each audio file:

1. **Room noise**  
   - Recorded in the NYU Education Building  
2. **Street noise**  
   - https://youtu.be/YF3pj_3mdMc?si=BSpCD-tEzvVaixAu  
3. **People noise**  
   - https://youtu.be/Q5jiitmLBOY?si=2MW1ZHiUiLLV_Pmw  

Noise is mixed into the audio at two signal-to-noise ratios:

- **15 dB SNR**  
- **5 dB SNR**

As with distortion, we preserve the original MedleyDB folder layout for seamless loading via `mirdata`.  
The Noised dataset can be found [here](https://drive.google.com/drive/folders/1Hx9Cab-a-cAM_Scs9dsTQX6YMCn495dz?usp=sharing)

---

### 3. Tuning Augmentation  

For tuning manipulation, we apply **global pitch shifts** to both the audio and the corresponding pitch CSV annotations.

We include 2 different levels of tuning:

- **±25 cents**  
- **±50 cents**  

This creates versions of each track that simulate real-world tuning drift while keeping time alignment intact.  
The Detuned dataset can be found [here](https://drive.google.com/drive/folders/1MmaDZANaJJ_qgiJKc1h_3Y4mac7xmzVu?usp=sharing)
