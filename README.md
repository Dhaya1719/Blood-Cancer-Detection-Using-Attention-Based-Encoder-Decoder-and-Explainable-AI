# 🩸 Blood Cancer Detection Using Attention-Based Encoder–Decoder and Explainable AI

An automated **Leukemia detection system** that classifies microscopic blood smear images into **Leukemia** and **Normal** classes using an attention-enhanced encoder–decoder deep learning architecture. The system combines convolutional feature extraction, attention mechanisms, decoder-based feature refinement, and a classification head. **Grad-CAM** is used to provide visual explanations of model predictions.

---

## ✨ Features

* Automated Leukemia detection from blood-cell images
* Attention-based Encoder–Decoder architecture
* Binary classification:

  * Leukemia
  * Normal
* Class imbalance handling using class weights
* Data augmentation
* Stratified train-validation split
* Single-image prediction
* Confusion Matrix and Classification Report
* Grad-CAM Explainable AI
* Training and validation performance visualization

---

## 🏗️ Model Architecture

```text
                  Blood Cell Image
                         │
                         ▼
                  ┌─────────────┐
                  │   Encoder   │
                  │             │
                  │ Conv Blocks │
                  │ Downsampling│
                  └──────┬──────┘
                         │
                         ▼
                    Bottleneck
                         │
                         ▼
                  Attention Blocks
                         │
                         ▼
                  ┌─────────────┐
                  │   Decoder   │
                  │             │
                  │ Upsampling  │
                  │ Conv Blocks │
                  └──────┬──────┘
                         │
                         ▼
                 Refined Features
                         │
                         ▼
              Global Average Pooling
                         │
                         ▼
                 Dense Classification
                         │
                         ▼
                    Dropout
                         │
                         ▼
                  Sigmoid Output
                         │
                         ▼
                Leukemia / Normal
                         │
                         ▼
                     Grad-CAM
```

### Architecture Details

The model contains:

* Convolutional encoder blocks
* Downsampling through max pooling
* A 512-channel bottleneck
* Attention gates in the decoder
* Transposed convolution for upsampling
* Decoder convolution blocks
* Global Average Pooling
* Dense classification layer
* Dropout regularization
* Sigmoid binary classification output

The model contains approximately **7.95 million trainable parameters**.

> **Note:** The encoder–decoder architecture is used for feature learning and classification. It is not trained as a supervised segmentation network because the C-NMC classification dataset used in this project does not provide paired ground-truth segmentation masks.

---

## 📂 Dataset

### C-NMC Leukemia Dataset

The project uses the **C-NMC 2019 Leukemia Dataset**, which contains microscopic blood-cell images belonging to two classes:

* Leukemia
* Normal

### Dataset Distribution

| Dataset    | Number of Images |
| ---------- | ---------------: |
| Total      |           10,661 |
| Training   |            8,528 |
| Validation |            2,133 |

The dataset was divided using an **80:20 stratified train-validation split**.

### Class Distribution

| Class    | Total Images |
| -------- | -----------: |
| Leukemia |        7,272 |
| Normal   |        3,389 |

### Class Mapping

```text
Leukemia → 0
Normal   → 1
```

### Dataset Source

https://www.kaggle.com/datasets/shafayou/c-nmc-2019-dataset

---

## ⚙️ Data Preprocessing

The images are resized to:

```text
224 × 224 pixels
```

Pixel values are normalized to the range:

```text
0 – 1
```

Data augmentation is applied to the training images using:

* Rotation
* Zoom
* Horizontal flipping

The validation images are only rescaled and are not augmented.

---

## ⚖️ Class Imbalance Handling

The dataset contains an unequal number of Leukemia and Normal images. To reduce the effect of class imbalance, class weights were calculated using the training data.

The calculated weights were:

```text
Leukemia → 0.7330
Normal   → 1.5729
```

These class weights were provided during model training so that the minority class receives greater importance.

---

## 🧠 Attention Mechanism

Attention gates are incorporated into the decoder.

The attention mechanism learns to assign greater importance to informative spatial features while reducing the influence of less relevant features.

The general flow is:

```text
Encoder Feature Map
        +
Decoder Feature Map
        ↓
Attention Gate
        ↓
Refined Feature Map
        ↓
Decoder
```

This allows the model to selectively emphasize useful features during classification.

---

## 🏋️ Model Training

The model was trained using:

| Parameter               | Value               |
| ----------------------- | ------------------- |
| Image Size              | 224 × 224           |
| Batch Size              | 16                  |
| Optimizer               | Adam                |
| Learning Rate           | 0.0001              |
| Loss Function           | Binary Crossentropy |
| Maximum Epochs          | 20                  |
| Class Weights           | Yes                 |
| Early Stopping          | Yes                 |
| Learning Rate Reduction | Yes                 |
| Model Checkpointing     | Yes                 |

### Training Callbacks

The following callbacks were used:

* **EarlyStopping**
* **ReduceLROnPlateau**
* **ModelCheckpoint**

The best model was selected based on validation accuracy.

---

## 📊 Results

The trained Attention Encoder–Decoder model achieved:

| Metric       |      Score |
| ------------ | ---------: |
| **Accuracy** | **92.55%** |
| Precision    |     93.61% |
| Recall       |     82.15% |
| F1 Score     |     87.51% |

### Classification Report

```text
              precision    recall  f1-score   support

Leukemia          0.92      0.97      0.95      1455
Normal            0.94      0.82      0.88       678

accuracy                              0.93      2133
macro avg         0.93      0.90      0.91      2133
weighted avg      0.93      0.93      0.92      2133
```

---

## 🔲 Confusion Matrix

The validation confusion matrix was:

```text
                 Predicted
              Leukemia  Normal

Actual
Leukemia        1417      38
Normal           121     557
```

The model correctly classified:

* **1,417 Leukemia images**
* **557 Normal images**

---

## 🔍 Explainable AI with Grad-CAM

**Grad-CAM (Gradient-weighted Class Activation Mapping)** was used to visualize the regions influencing the model's prediction.

The Grad-CAM process is:

```text
Input Image
     ↓
Trained Model
     ↓
Deep Convolutional Features
     ↓
Gradients of Predicted Class
     ↓
Weighted Feature Maps
     ↓
ReLU
     ↓
Normalized Heatmap
     ↓
Overlay on Input Image
```

For this model, Grad-CAM was generated using the deep encoder feature layer:

```text
conv2d_8
```

with feature-map dimensions:

```text
14 × 14 × 512
```

The resulting heatmap provides a visual representation of the regions contributing to the model's classification.

---

## 🧪 Single Image Prediction

The model also supports prediction on individual blood-cell images.

Example output from the notebook:

```text
Leukemia Probability: 0.9696
Normal Probability: 0.0304

Final Prediction: Leukemia
```

The prediction pipeline performs:

```text
Input Image
     ↓
Resize to 224 × 224
     ↓
Normalize
     ↓
Model Prediction
     ↓
Calculate Class Probabilities
     ↓
Leukemia / Normal
```

---

## 📦 Trained Model

The trained model is not stored directly in this GitHub repository because of file-size limitations.

### Download Model

**Google Drive:**
YOUR_GOOGLE_DRIVE_LINK

After downloading the model, create a `models` folder in the project directory:

```text
models/
└── attention_encoder_decoder_classifier.keras
```

The trained model used for the final evaluation is saved as:

```text
attention_encoder_decoder_classifier.keras
```

---

## 🛠️ Technologies Used

* **Python**
* **TensorFlow**
* **Keras**
* **NumPy**
* **Pandas**
* **OpenCV**
* **Scikit-learn**
* **Matplotlib**
* **Seaborn**
* **Jupyter Notebook**
* **Kaggle GPU**

---

## 📁 Repository Structure

```text
Blood-Cancer-Detection/
│
├── Blood_Cancer_Detection.ipynb
├── README.md
├── requirements.txt
├── .gitignore
├── LICENSE
│
└── report/
    └── Final_Project_Report.pdf
```

The trained model is available separately through the Google Drive link provided above.

---

## 🚀 Installation

Clone the repository:

```bash
git clone https://github.com/<your-username>/Blood-Cancer-Detection.git
```

Navigate to the project directory:

```bash
cd Blood-Cancer-Detection
```

Install the required dependencies:

```bash
pip install -r requirements.txt
```

---

## ▶️ Usage

Open the Jupyter Notebook:

```bash
jupyter notebook Blood_Cancer_Detection.ipynb
```

Run the notebook cells sequentially.

For single-image prediction:

1. Download the trained model from Google Drive.
2. Create a `models` folder.
3. Place `attention_encoder_decoder_classifier.keras` inside the folder.
4. Run the prediction cells in the notebook.

---

## 📄 Project Report

The complete project report is available in:

```text
report/Final_Project_Report.pdf
```

The report covers:

* Introduction
* Dataset Description
* System Architecture
* Data Preprocessing
* Class Imbalance Handling
* Attention Encoder–Decoder Architecture
* Model Training
* Results and Evaluation
* Grad-CAM Explainable AI
* Single Image Prediction
* Conclusion

---

## 🎯 Project Highlights

* Developed an **Attention-Enhanced Encoder–Decoder Classification Network** for Leukemia detection.
* Trained and evaluated the model on **10,661 C-NMC blood-cell images**.
* Achieved **92.55% validation accuracy**.
* Used **class weighting** to address dataset imbalance.
* Integrated **attention mechanisms** for spatial feature refinement.
* Implemented **Grad-CAM** for interpretable model predictions.
* Developed a **single-image prediction pipeline** for Leukemia/Normal classification.

---

## ⚠️ Disclaimer

This project is developed for **educational and research purposes**. It is not intended to replace professional medical diagnosis, clinical testing, or medical decision-making.

---

## 📜 License

This project is intended for educational and research purposes.
