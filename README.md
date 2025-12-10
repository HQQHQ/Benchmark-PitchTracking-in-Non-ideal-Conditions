# Benchmark-PitchTracking-in-Non-ideal-Conditions

The **Presentation Slides** can be found [here](https://docs.google.com/presentation/d/18ffY_ufv9FvHBbwAjxbAiSEgZJcs7sq6YjqP5kJFEmE/edit?usp=sharing)

## Overview  
This repository contains the code and data pipelines for our **Pitch Tracking Benchmark Project**, developed as part of the Fall 2025 final project for the Music Information Retrieval course at NYU.  
The goal of this project is to evaluate how pitch tracking models behave under different real-world recording conditions. We focus on three types of augmentations—**noise**, **distortion**, and **tuning changes**—and apply them to the MedleyDB-Pitch dataset. The processed audio and updated annotations are then used to benchmark models such as **CREPE** and **BasicPitch**.

---

## Work Division  
In this final group project:

**Heqi** was responsible for the overall planning, the dataset processing pipeline, adding noise and tuning perturbations, and the final analysis of the results under Noised conditions.

**Linhan** handled the distortion part in the dataset processing, recorded the room-noise samples, and conducted the final analysis of the results under Tuning conditions.

**Sienna** took charge of all presentation-related preparation, refinement of the slides, and the final analysis of the results under Distortion conditions.

**Zixuan** was responsible for the data loading and model evaluation pipeline, outputting experiment results, and producing the visualizations for the results.

---

## Data Processing  

All processing is based on the pitch subset of the **MedleyDB** dataset  
👉 https://medleydb.weebly.com/  

All data processing codes can be found in [this folder](https://github.com/HQQHQ/Benchmark-PitchTracking-in-Non-ideal-Conditions/tree/main/DataProcess)

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

---

## Model Evaluation

We evaluate three pitch tracking algorithms on the augmented datasets:

1. **Librosa** (`librosa.pyin`)
   - Signal processing-based pitch tracking method
   - CPU implementation with multiprocessing support

2. **CREPE** (Convolutional Representation for Pitch Estimation)
   - Deep learning-based monophonic pitch tracking
   - Uses full model capacity

3. **Basic Pitch**
   - Lightweight polyphonic note transcription model developed by Spotify
   - Supports multipitch estimation

### Experimental Conditions

We evaluate all models under 6 experimental conditions:
- **Clean**: Original audio (baseline)
- **Distortion**: Audio distortion
- **Noise 5dB**: Added 5dB noise
- **Noise 15dB**: Added 15dB noise
- **Pitch Shift 25cents**: Pitch shifted by 25 cents
- **Pitch Shift 50cents**: Pitch shifted by 50 cents

---

## Usage

### Setup

Install required dependencies:

```bash
pip install librosa pandas numpy scipy matplotlib seaborn tensorflow crepe resampy pretty_midi
```

### Project Structure

```
Final_Project/
├── src/
│   ├── utils/                    # Processing scripts
│   │   ├── process_basic_pitch.py
│   │   └── process_crepe.py
│   ├── normalization/            # Normalization module
│   │   ├── format_converter.py
│   │   └── renormalize_all.py
│   ├── evaluation/               # Evaluation module
│   │   ├── evaluator.py
│   │   └── metrics.py
│   ├── librosa_tracker.py       # Librosa tracker
│   ├── basic-pitch-main/         # Basic Pitch library
│   └── crepe-master/             # CREPE library
├── evaluate/                     # Evaluation scripts
│   ├── evaluate_basic_pitch.py
│   ├── evaluate_crepe.py
│   ├── evaluate_librosa.py
│   └── plot_results.py          # Plotting script
├── results/
│   ├── predictions/              # Prediction results
│   ├── metrics/                  # Evaluation metrics CSV files
│   └── figures/                  # Generated plots
└── MedleyDB-Pitch/               # Dataset
```

**Note**: All scripts must be run from the project root directory.

### 1. Generate Predictions

#### Basic Pitch Predictions
```bash
python3 src/utils/process_basic_pitch.py
```
This processes all experimental conditions and generates:
- Raw predictions: `results/predictions/basic_pitch/`
- Normalized predictions: `results/predictions/basic_pitch_normalized/`

#### CREPE Predictions
```bash
python3 src/utils/process_crepe.py
```
Generates:
- Raw predictions: `results/predictions/crepe/`
- Normalized predictions: `results/predictions/crepe_normalized/`

#### Librosa Predictions
Librosa predictions are already completed. Results are in:
- `results/predictions/librosa/`
- `results/predictions/librosa_normalized/`

### 2. Evaluate Predictions

The evaluation scripts compute metrics for each audio file and save them as CSV files.

#### Evaluate Basic Pitch
```bash
python3 evaluate/evaluate_basic_pitch.py
```
Output: `results/metrics/basic_pitch_*.csv`

#### Evaluate CREPE
```bash
python3 evaluate/evaluate_crepe.py
```
Output: `results/metrics/crepe_*.csv`

#### Evaluate Librosa
```bash
python3 evaluate/evaluate_librosa.py
```
Output: `results/metrics/librosa_*.csv`

## Prediction Results

### Raw Output Formats

Each algorithm produces pitch tracking results with different characteristics:

#### Librosa Output

**Time resolution**: 10ms (hop_length=160 at 16kHz sample rate)

**Example output**:
```
0.0,0.0
0.01,0.0
0.02,0.0
0.07,1779.0953213981911
0.08,1779.0953213981911
```

- Uses `librosa.pyin` for pitch estimation
- Outputs pitch estimates at regular 10ms intervals
- Unvoiced segments are marked as `0.0`
- Time stamps are generated using `librosa.frames_to_time()`

#### CREPE Output

**Time resolution**: 10ms (step_size=10ms)

**Example output**:
```
0.0,354.324393703952
0.01,53.24611311825794
0.02,52.90697570930432
0.03,52.735132752185415
```

- Deep learning model outputs pitch estimates at 10ms intervals
- Uses full model capacity for best accuracy
- May output non-zero frequencies even in unvoiced regions (handled during normalization)
- Time stamps are generated by CREPE's internal processing

#### Basic Pitch Output

**Time resolution**: ~5.8ms (hop_size_ms=5.8)

**Example output**:
```
0.0,0.0
0.0058,0.0
0.0116,0.0
0.0174,0.0
```

- Polyphonic model that outputs note events (start_time, end_time, MIDI_pitch)
- Note events are converted to time series format
- For overlapping notes, the highest frequency (dominant pitch) is kept
- Time grid matches ground truth hop size (~5.8ms)
- Unvoiced segments are marked as `0.0`

### Normalization Process

Since each algorithm produces predictions at different time resolutions and formats, we normalize all predictions to match the ground truth timestamp format for fair comparison.

#### Normalization Steps

1. **Load Ground Truth Timestamps**
   - Extract timestamps from ground truth CSV files
   - These timestamps define the target time grid for alignment

2. **Align Predictions**
   - For each ground truth timestamp, find the nearest prediction time point
   - Use linear interpolation between voiced prediction points
   - Preserve voiced/unvoiced boundaries:
     - If nearest prediction point is voiced → interpolate frequency
     - If nearest prediction point is unvoiced → set frequency to 0.0

3. **Interpolation Method**
   - For voiced segments: Linear interpolation between neighboring voiced points
   - For unvoiced segments: Set to 0.0
   - Handles edge cases (only before/after points available)

4. **Output Format**
   - Normalized predictions match ground truth format exactly
   - Same timestamps, same time resolution
   - Ready for direct evaluation

#### Example Normalization

**Before normalization** (Librosa, 10ms resolution):
```
0.0,0.0
0.01,0.0
0.02,440.5
0.03,441.2
```

**After normalization** (aligned to ground truth timestamps, ~5.8ms resolution):
```
0.0,0.0
0.0058,0.0
0.0116,440.3
0.0174,440.8
0.0232,441.0
```

The normalization process ensures that:
- All predictions are evaluated on the same time grid
- Fair comparison between algorithms with different time resolutions
- Preserves the voicing information from each algorithm

### Prediction File Format

All prediction files (both raw and normalized) are saved as CSV files with two columns (no header):
```
time(seconds), frequency(Hz)
0.000, 440.0
0.010, 442.5
...
```

- **Time**: Timestamp in seconds
- **Frequency**: Pitch frequency in Hz. `0.0` indicates unvoiced segments

### Normalized Predictions

Normalized predictions (in `*_normalized/` directories) are aligned to ground truth timestamps and can be directly used for evaluation.

### Evaluation Metrics

Evaluation results are saved in `results/metrics/` directory. Each CSV file contains the following columns:

| Column | Description |
|--------|-------------|
| `filename` | Audio filename |
| `OA` | Overall Accuracy - Overall accuracy considering voiced/unvoiced |
| `RPA` | Raw Pitch Accuracy - Pitch accuracy within 50 cents tolerance |
| `RCA` | Raw Chroma Accuracy - Pitch class (chroma) accuracy |
| `VR` | Voicing Recall - Voicing recall rate |

**Metric Range**: All metrics range from 0-1, where 1 indicates perfect performance.

### View Results Statistics

Use the utility script to view result statistics:

```bash
python3 scripts/view_results.py results/metrics
```

Or use the bash script:

```bash
bash scripts/view_results.sh results/metrics
```

---



## Results Analysis

### Distortion Results

| ![Distortion delta overall metrics](ResultsForPresentation/Dist_Results/delta_overall_table_highlight.png "Delta overall metrics by distortion level and model (highlighted)") | ![Distortion overall metrics](ResultsForPresentation/Dist_Results/overall_table_highlight.png "Overall accuracy table across distortion sources and models (highlighted)") |
| --- | --- |

![Distortion metric means](ResultsForPresentation/Dist_Results/metrics_dot_mean.png "Mean metric performance per model under distortion levels")

![Distortion OA by level](ResultsForPresentation/Dist_Results/oa_level_dot_mean.png "Overall accuracy across distortion levels by model")

![Distortion OA by source](ResultsForPresentation/Dist_Results/oa_source_dot_mean.png "Overall accuracy for vocal vs instrumental sources under distortion")

Across all distortion levels, Basic Pitch consistently shows the highest robustness, maintaining stable accuracy with minimal performance drop. pYIN exhibits moderate robustness, degrading under heavy distortion but remaining more stable than CREPE. CREPE, despite its high pitch precision in clean conditions, is the most vulnerable to distortion—showing substantial decreases in overall accuracy and voicing recall. This robustness pattern (Basic Pitch > pYIN > CREPE) persists across both vocal and instrumental sources, confirming that model-specific weaknesses revealed in the metric profiles directly explain their behavior under non-ideal audio conditions.

### Noised Results

| ![Noise delta overall metrics](ResultsForPresentation/Noise_Results/delta_overall_table_highlight.png "Delta overall metrics by noise type and SNR (highlighted)") | ![Noise overall metrics](ResultsForPresentation/Noise_Results/overall_table_highlight.png "Overall accuracy table across noise conditions and models (highlighted)") |
| --- | --- |

![Noise metric means](ResultsForPresentation/Noise_Results/metrics_dot_mean.png "Mean metric performance per model under noise conditions")

![Noise OA by type](ResultsForPresentation/Noise_Results/oa_noise_dot_mean.png "Overall accuracy across room, street, and people noise by model")

![Noise OA by source](ResultsForPresentation/Noise_Results/oa_noise_source_dot_mean.png "Overall accuracy for vocal vs instrumental sources under noise")

![Noise OA by SNR](ResultsForPresentation/Noise_Results/oa_snr_dot_mean.png "Overall accuracy at 5 dB vs 15 dB SNR by model")

![Noise OA by source (overall)](ResultsForPresentation/Noise_Results/oa_source_dot_mean.png "Overall accuracy for source types aggregated across noise conditions")

When noise was added, the three models showed very different levels of robustness. Basic Pitch remained the most stable, with almost no drop in accuracy and even slight gains in voicing detection. CREPE, which performs well on clean audio, degraded sharply under noise, especially in voicing recall, showing strong sensitivity to disrupted harmonic patterns. Librosa showed moderate decreases across metrics but stayed more consistent than CREPE. Overall, Basic Pitch demonstrated the best robustness to noise, CREPE was accurate but fragile, and Librosa was steady but limited. This happens because Basic Pitch uses stable spectral features and note-level smoothing, CREPE relies heavily on clean periodicity cues, and Librosa’s simpler frequency estimation is less robust but also less easily destabilized.

### Tuned/Pitch Shifted Results

| ![Tuning delta overall metrics](ResultsForPresentation/Tuned_Results/delta_overall_table_highlight.png "Delta overall metrics by pitch-shift magnitude and model (highlighted)") | ![Tuning overall metrics](ResultsForPresentation/Tuned_Results/overall_table_highlight.png "Overall accuracy table across tuning shifts and models (highlighted)") |
| --- | --- |

![Tuning metric means](ResultsForPresentation/Tuned_Results/metrics_dot_mean.png "Mean metric performance per model under tuning shifts")

![Tuning OA by shift](ResultsForPresentation/Tuned_Results/oa_shift_dot_mean.png "Overall accuracy across pitch-shift magnitudes by model")

![Tuning OA by source](ResultsForPresentation/Tuned_Results/oa_shift_source_dot_mean.png "Overall accuracy for vocal vs instrumental sources under pitch shifts")

![Tuning OA by source (overall)](ResultsForPresentation/Tuned_Results/oa_source_dot_mean.png "Overall accuracy for source types aggregated across tuning shifts")

When pitch shifts were applied, all models showed a decrease in accuracy. Basic Pitch remained the most stable, maintaining high accuracy and voicing recall under tuning variations. CREPE achieved good results on clean audio but dropped significantly when detuned, showing high sensitivity to pitch changes. Librosa performed less accurately overall but stayed relatively consistent. Overall, Basic Pitch demonstrated the best robustness to tuning variation, while CREPE was precise but fragile, and Librosa was steady but limited. This happens because Basic Pitch integrates both spectral and temporal features, CREPE depends too much on clean harmonic patterns, and Librosa relies on simpler frequency estimation that’s less affected by noise.

---

## Additional Tools

### Renormalize All Predictions

If you need to renormalize all prediction results:

```bash
python3 src/normalization/renormalize_all.py
```

### Check Dataset Integrity

```bash
python3 scripts/check_dataset.py
```

---

## File Descriptions

### Core Modules

- `src/librosa_tracker.py` - Librosa pitch tracking implementation
- `src/basic-pitch-main/basic_pitch_tracker.py` - Basic Pitch wrapper
- `src/crepe-master/crepe_tracker.py` - CREPE wrapper
- `src/normalization/format_converter.py` - Prediction result normalization
- `src/evaluation/evaluator.py` - Evaluation main class
- `src/evaluation/metrics.py` - Evaluation metrics calculation

### Processing Scripts

- `src/utils/process_basic_pitch.py` - Basic Pitch batch processing
- `src/utils/process_crepe.py` - CREPE batch processing

### Evaluation Scripts

- `evaluate/evaluate_*.py` - Evaluation scripts for each algorithm
- `evaluate/plot_results.py` - Results visualization

---

## Notes

1. **Run Location**: All scripts must be run from the project root directory
2. **Paths**: The project uses relative paths, ensure you're in the correct directory
3. **GPU**: Basic Pitch and CREPE can use GPU acceleration (if TensorFlow GPU is configured)
4. **Memory**: Processing large numbers of files may require significant memory

---

## Results File Locations

- **Predictions**: `results/predictions/`
- **Evaluation Metrics**: `results/metrics/`
- **Plots**: `results/figures/`
