# MEDICAL-IMAGE-ANALYSIS
This project focuses on building a **deep learning–based multimodal medical imaging system** that analyzes **X-Ray and MRI scans together** to detect fractures or disease regions more accurately. Each imaging modality provides different information—X-rays capture bone structure clearly, while MRI shows soft tissues and internal abnormalities. By combining both, the system creates a more complete medical understanding compared to single-modality models.

The framework contains several important components:
1. Image Preprocessing & Enhancement
Before analysis, the uploaded scans are cleaned and standardized. The system performs:
* Noise removal
* Intensity normalization
* Contrast improvement
* Spatial alignment between X-Ray & MRI
This ensures both images match properly for multimodal fusion.

2. Segmentation Module
Using architectures like U-Net / nnU-Net, the system separates important regions (bones, tissues, or fractures).
Segmentation helps the model focus only on relevant medical structures and removes unwanted background.

3. Feature Extraction

CNN-based deep learning models extract meaningful patterns from the images:

* Texture
* Shape
* Edges
* Abnormal regions

Radiomics features are also used to strengthen the feature representation.

4. Feature Fusion

One of the core contributions is comparing multiple fusion strategies:

| Fusion Type             | Explanation                                     |
| ----------------------- | ----------------------------------------------- |
| **Early Fusion**        | Combines both images at the input stage.        |
| **Intermediate Fusion** | Extracts features separately, combines mid-way. |
| **Late Fusion**         | Combines predictions from separate models.      |

This helps identify which fusion strategy improves diagnosis the most.

 5. Multimodal Deep Learning Core

A dual-stream network processes X-ray and MRI independently, then merges them to make a final decision.
This improves learning of both structural and soft-tissue information.

6. Calibration Module

Deep learning models often give inaccurate confidence scores.
Model calibration techniques like:

* **Temperature Scaling**
* **Platt Scaling**
* **Matrix Scaling**

help convert raw model outputs into **reliable probabilities**, which is very important for medical decisions.

7. Visualization & Reporting

The system generates:

* Highlighted fracture/abnormal areas
* Enhanced medical images
* Prediction overlays
* Downloadable reports (Images + PDFs)

This makes the system usable for research, clinical analysis, and academic demonstrations.

2. HOW TO RUN THE CODE (Step-by-Step)

### **📌 Step 1: Clone the Repository**

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```
Step 2: Install Python Dependencies

Create a virtual environment (recommended):
```bash
python -m venv env
source env/bin/activate     # Mac / Linux
env\Scripts\activate        # Windows
```

Install required packages:

```bash
pip install -r requirements.txt
```

---
 Step 3: Set Up Environment Variables

If your model uses API keys, create a `.env` file:

```
GEMINI_API_KEY=your_api_key_here
```

---

Step 4: Run the Backend Model

```bash
python main.py
```

This loads:
✔ Preprocessing
✔ Segmentation
✔ Fusion
✔ Calibration
✔ Prediction pipeline

---

Step 5: Run the React Frontend (If Included)

Go to UI folder:

```bash
cd ui-react-app
npm install
npm start
```

This opens the UI where you can:
✔ Upload X-Ray
✔ Upload MRI
✔ Enhance Image
✔ Generate Predictions
✔ View overlays
✔ Download PDFs

