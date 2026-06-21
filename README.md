```markdown
# Intelligent SOC Assistant: Hybrid Network Intrusion Detection with Explainable AI

## Project Overview

This project develops foundational components for an AI-driven Security Operations Centre (SOC) assistant. It employs a hybrid approach, combining unsupervised clustering for anomaly detection with supervised classification for known threat identification in large-scale network traffic datasets (CICIDS2017 and UNSW-NB15). A key focus is on interpretability, ensuring that model decisions can be understood and acted upon by security analysts.

## Table of Contents

1.  [Project Goals](#project-goals)
2.  [Datasets](#datasets)
3.  [Methodology](#methodology)
    *   [Data Ingestion & Initial Preparation](#data-ingestion-initial-preparation)
    *   [Exploratory Data Analysis (EDA) & Cleaning](#exploratory-data-analysis-eda-cleaning)
    *   [Advanced Preprocessing & Feature Engineering](#advanced-preprocessing-feature-engineering)
    *   [Vector Embedding Generation (Autoencoder)](#vector-embedding-generation-autoencoder)
    *   [K-Means Clustering & Visualization](#k-means-clustering-visualization)
    *   [Supervised Classification with XGBoost](#supervised-classification-with-xgboost)
    *   [Deep Learning Classification](#deep-learning-classification)
    *   [Model Interpretability with SHAP](#model-interpretability-with-shap)
4.  [Key Findings & Insights](#key-findings-insights)
5.  [Setup & Usage](#setup-usage)
6.  [Future Work](#future-work)
7.  [Contributors](#contributors)
8.  [License](#license)

## Project Goals

*   To build a robust system for detecting and categorizing network intrusions.
*   To handle large and imbalanced network traffic datasets efficiently.
*   To leverage both unsupervised (clustering) and supervised (classification) machine learning techniques.
*   To provide explainable AI insights for better understanding of model predictions.
*   To demonstrate the feasibility of an AI-driven SOC assistant for reducing alert fatigue and enhancing threat detection capabilities.

## Datasets

This project utilizes two prominent network intrusion detection datasets:

*   **CICIDS2017:** A comprehensive dataset capturing various modern attack scenarios, including DoS, DDoS, Brute Force, XSS, SQL Injection, and more, alongside benign traffic. It is known for its detailed flow features.
    *   **Source:** [[[Insert CICIDS2017 Download Link here](https://drive.google.com/file/d/14kQixoz8otuyXKE1_K8M2NZDqF15VaN4/view?usp=drive_link)]
*   **UNSW-NB15:** Contains nine types of attacks generated from real-world network traffic, categorizing them into normal and attack records. It features a mix of numerical and categorical features.
    *   **Source:** [[Insert UNSW-NB15 Download Link here](https://drive.google.com/file/d/1h_5X3aAXUBo6b6ueclTpDkVHmsRqqM01/view?usp=drive_link)]

## Methodology

### Data Ingestion & Initial Preparation

*   **Mounting Google Drive:** Seamless integration with Google Drive for dataset access.
*   **Unzipping:** Efficient extraction of zipped CSV files into the Colab environment.
*   **Combining CSVs:** A custom function (`combine_csv_files`) to concatenate multiple CSVs, clean column names (stripping whitespace, removing Byte Order Mark - BOM), and optimize memory usage by downcasting numerical data types.

### Exploratory Data Analysis (EDA) & Cleaning

*   **Initial Inspection:** Comprehensive `df.info()` and `df.describe()` to understand data structure, types, and summary statistics.
*   **Handling Missing Values (NaNs):**
    *   CICIDS2017: Rows with minimal NaNs were dropped.
    *   UNSW-NB15: Median imputation for numerical features and mode imputation for categorical features were applied to preserve data integrity due to extensive NaNs.
*   **Duplicate Removal:** Identified and removed duplicate rows from both datasets to ensure unique observations for model training.
*   **Target Variable Distribution:** Visualized the severe class imbalance in attack labels/categories using count plots, which is typical for network intrusion datasets.

### Advanced Preprocessing & Feature Engineering

*   **`load_clean_and_sample_dataframe` function:** A unified pipeline for:
    *   **Infinite Value Handling:** Conversion of `inf` to `NaN` and subsequent dropping.
    *   **High-Cardinality Feature Management:** Selective dropping of categorical columns with an excessive number of unique values to prevent feature explosion during one-hot encoding.
    *   **One-Hot Encoding:** Applied to suitable categorical features.
    *   **Stratified Sampling:** Crucially, implemented stratified random sampling to create smaller, manageable subsets (e.g., 5% for CICIDS2017, aggressive pre-sampling for UNSW-NB15) while perfectly preserving the original class distribution. This enables efficient model development and testing without memory exhaustion.
    *   **Feature Scaling:** `MinMaxScaler` applied to all numerical features, essential for neural networks and clustering algorithms.

### Vector Embedding Generation (Autoencoder)

*   **Deep Learning Autoencoder:** A TensorFlow/Keras-based Autoencoder model was designed and trained.
    *   **Architecture:** Consisted of an encoder (input -> dense layers -> bottleneck/embedding layer) and a decoder (bottleneck -> dense layers -> output layer).
    *   **Purpose:** To learn a compressed, low-dimensional (16-dimensional) representation (vector embeddings) of the high-dimensional network flow features. These embeddings capture the intrinsic relationships and patterns of network behaviors.

### K-Means Clustering & Visualization

*   **K-Means Application:** Applied K-Means clustering to the generated vector embeddings for both datasets.
*   **Dynamic Cluster Number:** The number of clusters was set to match the number of unique true attack labels to assess alignment.
*   **Silhouette Score:** Used to evaluate clustering quality, indicating how well-separated the clusters are (CICIDS2017: 0.6047, UNSW-NB15: 0.4716).
*   **PCA Visualization:** Principal Component Analysis (PCA) reduced the 16-dimensional embeddings to 2 dimensions for visualization. Dual scatter plots (colored by True Labels and by K-Means Cluster IDs) provided visual insights into the coherence and separation of learned clusters.

### Supervised Classification with XGBoost

*   **XGBoost Classifier:** Implemented an XGBoost model for multi-class classification on the CICIDS2017 dataset.
*   **Preprocessing for XGBoost:** Handled infinite values and NaNs by imputation (median) after train-test splitting to prevent data leakage. `LabelEncoder` converted categorical labels to numerical format.
*   **Performance Evaluation:** A detailed `classification_report` was generated, providing precision, recall, and F1-scores per class. The model showed high overall accuracy (99.98%), demonstrating strong predictive power, especially for majority classes.
*   **Visualization:** Bar plots illustrated class-wise precision, recall, and F1-scores for intuitive understanding.

### Deep Learning Classification

*   **Deep Learning Model:** Developed a Sequential Keras model with Dense and Dropout layers.
*   **`StandardScaler`:** Emphasized the critical role of `StandardScaler` for neural networks to ensure faster convergence and prevent feature dominance.
*   **Performance:** Achieved high training (98.59%) and validation (98.78%) accuracy with low loss, indicating robust performance and good generalization capabilities for unseen data.
*   **Plots:** Visualized training and validation accuracy and loss over epochs, confirming stable learning.

### Model Interpretability with SHAP

*   **SHAP (SHapley Additive exPlanations):** Applied SHAP to interpret the XGBoost model's predictions, providing crucial explainability for security applications.
*   **`shap.TreeExplainer`:** Utilized for optimized analysis of tree-based models.
*   **SHAP Summary Plots:** Generated Beeswarm and Bar plots to visualize:
    *   **Feature Importance:** Which features have the most impact on model output.
    *   **Feature Impact Direction:** How specific feature values (high/low) influence predictions (positive/negative impact).
*   **Insights:** Highlighted key influential features (e.g., `destination port`, `flow duration`, `total fwd packets`), enhancing trust and providing actionable intelligence for threat hunting.

## Key Findings & Insights

*   The hybrid approach combining Autoencoder-based embeddings with K-Means clustering effectively groups similar attack behaviors, offering valuable insights for anomaly detection.
*   Supervised models like XGBoost and Deep Learning achieve high accuracy in classifying known attack types, forming a strong backbone for real-time intrusion detection.
*   Class imbalance is a significant challenge, particularly for minority attack classes, often leading to lower recall for these rare events.
*   SHAP analysis provides critical explainability, turning black-box model predictions into actionable insights for security analysts.
*   Stratified sampling and memory optimization techniques are essential for handling large-scale network datasets in resource-constrained environments like Colab.

## Setup & Usage

### Local Setup (Recommended for full development)

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/USDMSAISummer26G8/intelligentSOC.git
    cd intelligentSOC
    ```
2.  **Create a virtual environment (optional but recommended):**
    ```bash
    python -m venv venv
    source venv/bin/activate # On Windows, use `venv\Scripts\activate`
    ```
3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```
    (A `requirements.txt` file should be generated containing all necessary libraries like `pandas`, `numpy`, `scikit-learn`, `tensorflow`, `xgboost`, `shap`, `matplotlib`, `seaborn`, `plotly`, `kaleido`)

4.  **Download Datasets:** Download `CICIDS2017.zip` and `UNSW-NB15.zip` from their respective sources (linked above) and place them in `data/raw/`.

5.  **Run the Jupyter Notebook:** Open and run `notebooks/your_notebook_name.ipynb` (replace with your actual notebook name) in Jupyter Lab or VS Code.

### Colab Usage

1.  **Upload `your_notebook_name.ipynb`** to your Google Drive.
2.  **Open in Colab.**
3.  **Mount Google Drive:** Run the first code cell to mount your Google Drive.
4.  **Place Data:** Ensure `CICIDS2017.zip` and `UNSW-NB15.zip` are accessible in your Google Drive (e.g., in `My Drive/`).
5.  **Run all cells sequentially.** The notebook handles environment setup, data loading, preprocessing, model training, and visualization.

## Future Work

*   **Hyperparameter Optimization:** Conduct more extensive hyperparameter tuning for Autoencoder, K-Means, XGBoost, and Deep Learning models.
*   **Novelty Detection:** Explore advanced unsupervised techniques for detecting entirely new, unknown attack types.
*   **Real-time Processing:** Integrate the models with a real-time stream processing framework (e.g., Apache Flink, Kafka) for live network traffic analysis.
*   **Reinforcement Learning for Adaptive Defense:** Investigate using RL agents to learn optimal defensive strategies based on detected threats.
*   **More Advanced Embeddings:** Experiment with alternative embedding techniques, such as contrastive learning or transformer-based models for tabular data.
*   **Full End-to-End System:** Develop a complete SOC assistant prototype with a user interface for analysts.

## Contributors

*   [Ravisha Krishnamurthy]
*   [Ashish Kumar]
*   [Sarwesh Karande]



## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
```
