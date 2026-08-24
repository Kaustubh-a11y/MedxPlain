# MedXplain: Clinical Explainable AI (XAI) Suite

## 1. Executive Summary & Problem Statement

### **What is MedXplain?**
MedXplain is an end-to-end Clinical Decision Support System (CDSS) that pairs high-accuracy machine learning classifiers with **dual-surrogate Explainable AI (XAI)**. It turns "black-box" machine learning predictions into trustworthy, human-interpretable diagnostic insights for clinicians and medical researchers.

### **The Problem**
Standard ML models in healthcare achieve high predictive performance but lack transparency. Clinicians cannot safely rely on opaque risk scores without knowing **which clinical biomarkers drove the diagnosis**, **how confident the explanation is**, and **what intervention flips the risk**.

### **The Solution**
MedXplain integrates **SHAP** (Shapley Additive exPlanations), **LIME** (Local Interpretable Model-agnostic Explanations), a **Dual-XAI Consensus Agreement Index**, and **Prescriptive Counterfactual Reasoning** into a modern, real-time diagnostic workspace.

---

## 2. Machine Learning Architecture

### **Data & Disease Workspaces**
| Disease Workspace | Clinical Dataset | Best Model Architecture | Accuracy | ROC-AUC | Key Biomarkers |
| :--- | :--- | :--- | :---: | :---: | :--- |
| **Heart Disease** | UCI Cleveland Dataset | **Logistic Regression** | **91.7%** | **0.9531** | Fluoroscopy Vessels (`ca`), Thallium Scan (`thal`), Chest Pain (`cp`) |
| **Diabetes Onset** | NIDDK Pima Indians | **XGBoost Classifier** | **89.0%** | **0.9537** | Glucose Tolerance, Insulin, BMI, Age |
| **Kidney Disease** | UCI Chronic Kidney (CKD) | **Random Forest** | **100.0%** | **1.0000** | Serum Creatinine (`sc`), Hemoglobin (`hemo`), Packed Cell Vol (`pcv`) |

### **Why These Models?**
* **Logistic Regression (Heart)**: On small-to-medium clinical cohorts (UCI Cleveland: 303 cases), regularized L2 Logistic Regression outperforms deep/complex trees by preventing overfitting, maintaining low variance, and preserving smooth log-odds monotonicity.
* **XGBoost Classifier (Diabetes)**: Non-linear feature interactions between Glucose, Insulin, and BMI are best captured by gradient boosted decision trees.
* **Random Forest (CKD)**: CKD biomarkers possess clear physiological separability (e.g., Serum Creatinine > 3.5 mg/dL). Ensemble random forests achieve 100% test accuracy with zero variance across cross-validation folds.

### **Data Preprocessing & Validation**
* **Imputation**: Median/mode imputation for missing clinical values (`src/data_loader.py`).
* **Scaling**: Standardisation (`StandardScaler`) fitted strictly on training splits to prevent data leakage.
* **Imbalance Handling**: SMOTE oversampling applied on training folds when class imbalance occurs.
* **Validation Scheme**: Stratified 5-Fold Cross-Validation with an 80/20 out-of-sample holdout test split.

---

## 3. Explainability (XAI) & Prescriptive Engine

### **1. SHAP (Shapley Additive exPlanations)**
* **What**: Game-theoretic Shapley values (`shap.TreeExplainer` & `shap.LinearExplainer`).
* **Why**: Provides mathematically optimal, additive feature attributions satisfying Efficiency, Symmetry, Dummy, and Additivity properties.
* **How**: Calculates exact baseline output $E[f(x)]$ vs. individual feature contributions $\phi_i$, visualized via custom high-contrast waterfall bars.

### **2. LIME (Local Interpretable Model-agnostic Explanations)**
* **What**: Local surrogate linear perturbation (`lime.lime_tabular.LimeTabularExplainer`).
* **Why**: Validates SHAP attributions using local linear decision boundary approximations around the patient's feature vector.
* **How**: Generates 5,000 synthetic perturbations near the sample to output human-readable decision rules (e.g., `Serum Creatinine > 1.20`).

### **3. Dual-XAI Consensus Agreement Index**
* **What**: A novel confidence metric ($0.0 - 1.0$) evaluating agreement between SHAP and LIME.
* **Why**: High model accuracy does not guarantee explanation reliability. When SHAP and LIME agree on top risk drivers, clinician trust increases.
* **How**: Computes Jaccard/Top-K overlap ($k=3$) between SHAP and LIME attributions:
  $$\text{Consensus Score} = \frac{|\text{Top}_k(\text{SHAP}) \cap \text{Top}_k(\text{LIME})|}{k}$$

### **4. Actionable Clinical Counterfactuals**
* **What**: Prescriptive "what-must-change" intervention engine.
* **Why**: Knowing risk is insufficient; clinicians need minimum modification pathways to lower patient risk.
* **How**: Performs greedy univariate and bivariate search across feature bounds to find the minimal delta required to push risk below $50.0\%$.

### **5. Real-Time What-If Simulator**
* **What**: High-speed live parameter adjustment tool.
* **Why**: Allows clinicians to interactively slide glucose, blood pressure, or creatinine levels and observe immediate model probability recalibration.
* **How**: Connected via debounced API calls (`POST /api/simulate`) executing model inference in under 5ms.

---

## 4. Backend & API Services

### **Technologies Used**
* **FastAPI**: Ultra-fast Async Python web framework.
* **Uvicorn**: ASGI server serving API endpoints and static frontend assets on `http://127.0.0.1:8000`.
* **Pydantic v2**: Strict request/response schema validation (`api/schemas.py`).

### **Core Endpoints**
* `GET /api/config`: Returns disease metadata, biomarker schemas, ranges, and cohort presets.
* `POST /api/predict`: Returns predicted class, probability percentage, and risk tier.
* `POST /api/explain`: Computes SHAP attributions, LIME rules, consensus index, counterfactuals, and narrative summary.
* `POST /api/simulate`: High-speed endpoint for instant slider recalibration.
* `GET /api/metrics`: Returns out-of-sample benchmark metrics across all models.

---

## 5. Frontend & UI/UX Architecture

### **Technologies Used**
* **React 18 + Vite**: High-performance Single Page Application (SPA).
* **Vanilla CSS (CSS Custom Properties)**: Complete design system without heavy framework bloat.
* **Lucide React**: Clean medical & navigational icon set.

### **Design Language & Visual Philosophy**
* **Warm Off-White Canvas (`#F4F5F0`)**: Premium, non-fatiguing medical aesthetic inspired by modern clinical UI references.
* **Dark Navy Hero Blocks (`#0F1E3C`)**: High-contrast dark cards for hero metrics, circular risk gauges, and benchmark highlights.
* **Pear Yellow (`#C8E619`)**: Used strictly as the primary focal CTA and active indicator ("Rule of One Yellow").
* **Organic Liquid Bubble Animation**: Custom GPU-accelerated CSS keyframes (`@keyframes liquidRise` / `liquidMorph`) creating fluid ambient motion inside dark navy blocks.
* **Typography System**: Apple SF Pro Display (headings), SF Pro Text (body), and **SF Mono** (tabular numbers, SHAP values, and table figures to eliminate layout jitter).

### **Navigation & Layout**
* **Vertical Left Rail**: Minimalist side dock displaying the Zen Pebble logo (`logo.png`) and vertically centered `M E D X P L A I N` typography stack.
* **Frosted End-Wall Navbar**: Sticky glassmorphism header (`backdrop-filter: blur(20px)`) preventing scroll clashing when content passes underneath.

---

## 6. Project Structure

```
pbl/
├── api/
│   ├── server.py              # FastAPI server & static file host
│   └── schemas.py             # Pydantic data schemas
├── app/
│   ├── config.py              # Clinical disease configurations & feature metadata
│   ├── predictor.py           # Model inference pipeline
│   ├── explainer.py           # SHAP, LIME, Consensus, & Counterfactual engines
│   ├── main.py                # Streamlit alternative interface
│   └── views/                 # Streamlit UI views
├── dataset/                   # UCI Cleveland, Pima Diabetes, & CKD CSVs
├── frontend/
│   ├── public/logo.png        # Brand asset
│   ├── src/
│   │   ├── components/        # Navbar, VerticalRail, RiskGauge, ShapWaterfall, etc.
│   │   ├── views/             # HomeDashboard, DiseaseWorkspace, ModelBenchmarks
│   │   ├── services/api.js    # Axios/Fetch API client
│   │   └── index.css          # Design system & CSS custom properties
│   └── package.json
├── models/                    # Trained Scikit-Learn / XGBoost pkl artifacts
├── src/
│   ├── data_loader.py         # Data loading & preprocessing utilities
│   ├── model_pipeline.py      # Core model training & cross-validation script
│   └── train_*.py             # Model training entrypoints
├── explain.md                 # Complete system technical explanation
└── README.md                  # Quickstart guide
```

---

## 7. How to Run the Application

### **Start the Unified Server (FastAPI + React)**
```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Build Frontend (Production Bundle)
cd frontend
npm install
npm run build
cd ..

# 3. Launch Server
python -m uvicorn api.server:app --host 127.0.0.1 --port 8000
```
Open **[http://localhost:8000](http://localhost:8000)** in your browser.

---

## 8. Summary of Engineering Highlights
1. **Dual XAI Validation**: Combines game-theoretic SHAP with local linear LIME to quantify explanation consensus.
2. **Prescriptive Actionability**: Provides actionable counterfactual targets rather than passive probability scores.
3. **Sub-10ms Recalibration**: Debounced What-If simulation slider for instant interactive exploration.
4. **State-of-the-Art UX**: Modern Apple SF Pro typography, liquid animations, warm off-white palette, and frosted glass navigation.
