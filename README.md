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



BACKEND CODE:
import React, { useState, useCallback } from 'react';
import { analyzeMedicalScan } from './services/geminiService';
import type { AnalysisResult } from './types';
import ImageUploader from './components/ImageUploader';
import AnalysisDisplay from './components/AnalysisDisplay';
import Spinner from './components/Spinner';
import { BrainCircuitIcon, UploadCloudIcon, DownloadIcon, PdfIcon } from './components/Icons';

// Let TypeScript know about the global jsPDF library from the CDN
declare const jspdf: any;

// Helper function to convert file to base64
const fileToBase64 = (file: File): Promise<{ base64: string; mimeType: string }> => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => {
      const result = reader.result as string;
      const base64 = result.split(',')[1];
      resolve({ base64, mimeType: file.type });
    };
    reader.onerror = error => reject(error);
  });
};

export default function App(): React.ReactElement {
  const [selectedFile, setSelectedFile] = useState<File | null>(null);
  const [imageUrl, setImageUrl] = useState<string | null>(null);
  const [imageDimensions, setImageDimensions] = useState<{ width: number; height: number } | null>(null);
  const [analysisResult, setAnalysisResult] = useState<AnalysisResult | null>(null);
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const [isDownloading, setIsDownloading] = useState<boolean>(false);
  const [error, setError] = useState<string | null>(null);
  const [enhanceImage, setEnhanceImage] = useState<boolean>(false);

  const handleFileSelect = useCallback((file: File) => {
    setSelectedFile(file);
    setAnalysisResult(null);
    setError(null);
    
    const url = URL.createObjectURL(file);
    setImageUrl(url);

    const img = new Image();
    img.onload = () => {
        setImageDimensions({ width: img.width, height: img.height });
        // Do not revoke URL here, it's needed for display
    };
    img.onerror = () => {
      // Clean up if image fails to load
      URL.revokeObjectURL(url);
      setError("Failed to load image dimensions.");
    }
    img.src = url;
  }, []);

  const handleAnalyzeClick = async () => {
    if (!selectedFile || !imageDimensions) {
      setError("Please select an image file first and wait for it to load.");
      return;
    }

    setIsLoading(true);
    setError(null);
    setAnalysisResult(null);

    try {
      const { base64, mimeType } = await fileToBase64(selectedFile);
      const result = await analyzeMedicalScan(base64, mimeType, enhanceImage, imageDimensions);
      setAnalysisResult(result);
    } catch (err) {
      console.error("Analysis failed:", err);
      setError("Failed to analyze the image. The model may not be able to process this scan. Please try another one.");
    } finally {
      setIsLoading(false);
    }
  };
  
  const handleReset = () => {
    if (imageUrl) {
        URL.revokeObjectURL(imageUrl);
    }
    setSelectedFile(null);
    setImageUrl(null);
    setImageDimensions(null);
    setAnalysisResult(null);
    setError(null);
    setIsLoading(false);
    setIsDownloading(false);
    setEnhanceImage(false);
  };

  const createAnnotatedCanvas = useCallback(async (): Promise<HTMLCanvasElement | null> => {
    if (!imageUrl || !imageDimensions || !analysisResult) return null;
  
    const canvas = document.createElement('canvas');
    canvas.width = imageDimensions.width;
    canvas.height = imageDimensions.height;
    const ctx = canvas.getContext('2d');
    if (!ctx) return null;
  
    const img = new Image();
    img.crossOrigin = 'Anonymous';
  
    await new Promise<void>((resolve, reject) => {
      img.onload = () => resolve();
      img.onerror = reject;
      img.src = imageUrl;
    });
    
    ctx.drawImage(img, 0, 0, imageDimensions.width, imageDimensions.height);
    
    if (analysisResult.status === 'anomaly_detected') {
        analysisResult.findings.forEach(finding => {
            const { boundingBox, finding: findingName } = finding;
        
            ctx.fillStyle = 'rgba(239, 68, 68, 0.3)';
            ctx.fillRect(boundingBox.x, boundingBox.y, boundingBox.width, boundingBox.height);
            
            ctx.strokeStyle = '#dc2626';
            ctx.lineWidth = 2;
            ctx.strokeRect(boundingBox.x, boundingBox.y, boundingBox.width, boundingBox.height);
        
            const labelHeight = 20;
            const labelMargin = 2;
            ctx.font = 'bold 12px sans-serif';
            
            const placeLabelInside = boundingBox.y < labelHeight + (labelMargin * 2);
        
            const labelRectY = placeLabelInside 
                ? boundingBox.y + labelMargin 
                : boundingBox.y - labelHeight - labelMargin;
            
            const textMetrics = ctx.measureText(findingName);
            const labelWidth = textMetrics.width + 12;
            let labelRectX = boundingBox.x;
            if (labelRectX + labelWidth > imageDimensions.width) {
                labelRectX = imageDimensions.width - labelWidth - labelMargin;
            }
            if (labelRectX < 0) labelRectX = 0;
        
            const textX = labelRectX + 6;
        
            ctx.fillStyle = '#dc2626';
            ctx.fillRect(labelRectX, labelRectY, labelWidth, labelHeight);
            
            ctx.fillStyle = '#ffffff';
            ctx.textAlign = 'start';
            ctx.textBaseline = 'middle';
            ctx.fillText(findingName, textX, labelRectY + labelHeight / 2);
        });
    }

    return canvas;
  }, [imageUrl, imageDimensions, analysisResult]);

  const handleImageDownload = useCallback(async () => {
    if (!analysisResult) return;
    setIsDownloading(true);

    const canvas = await createAnnotatedCanvas();
    if (!canvas) {
        setError("Failed to create image for download.");
        setIsDownloading(false);
        return;
    }

    const link = document.createElement('a');
    const diseaseName = analysisResult.status === 'anomaly_detected' && analysisResult.findings.length > 0
      ? analysisResult.findings[0].finding.replace(/[\s/]/g, '_').toLowerCase()
      : 'healthy';
    link.download = analysis_report_${diseaseName}.png;
    link.href = canvas.toDataURL('image/png');
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    setIsDownloading(false);
  }, [createAnnotatedCanvas, analysisResult]);

  const handlePdfDownload = useCallback(async () => {
    if (!analysisResult) return;
    setIsDownloading(true);
    setError(null);

    try {
        const canvas = await createAnnotatedCanvas();
        if (!canvas) {
            throw new Error("Failed to create annotated image canvas.");
        }

        const { jsPDF } = jspdf;
        const doc = new jsPDF({ orientation: 'p', unit: 'pt', format: 'a4' });

        const page = { width: doc.internal.pageSize.getWidth(), margin: 40 };
        const contentWidth = page.width - page.margin * 2;
        let yPos = page.margin;

        doc.setFontSize(22).setFont('helvetica', 'bold');
        doc.text('Medical Scan Analysis Report', page.width / 2, yPos, { align: 'center' });
        yPos += 40;

        doc.setFontSize(14).setFont('helvetica', 'bold');
        doc.text('Analyzed Scan', page.margin, yPos);
        yPos += 20;

        const imageDataUrl = canvas.toDataURL('image/png');
        const imgProps = doc.getImageProperties(imageDataUrl);
        const imgAspectRatio = imgProps.width / imgProps.height;
        const imgDisplayWidth = contentWidth;
        const imgDisplayHeight = imgDisplayWidth / imgAspectRatio;
        doc.addImage(imageDataUrl, 'PNG', page.margin, yPos, imgDisplayWidth, imgDisplayHeight);
        yPos += imgDisplayHeight + 30;
        
        doc.setFontSize(14).setFont('helvetica', 'bold');
        doc.text('AI Analysis Findings', page.margin, yPos);
        yPos += 20;

        doc.setFontSize(12).setFont('helvetica', 'bold');
        doc.text('Summary Impression', page.margin, yPos);
        yPos += 15;

        doc.setFontSize(11).setFont('helvetica', 'normal');
        const summaryLines = doc.splitTextToSize(analysisResult.summary, contentWidth);
        doc.text(summaryLines, page.margin, yPos);
        yPos += (summaryLines.length * 12) + 20;

        if (analysisResult.status === 'anomaly_detected' && analysisResult.findings.length > 0) {
            doc.setFontSize(12).setFont('helvetica', 'bold');
            doc.text('Detailed Findings', page.margin, yPos);
            yPos += 15;
            
            analysisResult.findings.forEach((finding, index) => {
                if (yPos > doc.internal.pageSize.getHeight() - 100) { doc.addPage(); yPos = page.margin; }

                doc.setFont('helvetica', 'bold').setTextColor(178, 34, 34); // Red
                const findingTitle = doc.splitTextToSize(Finding ${index + 1}: ${finding.finding}, contentWidth);
                doc.text(findingTitle, page.margin, yPos);
                yPos += (findingTitle.length * 12) + 5;

                doc.setFont('helvetica', 'normal').setTextColor(0, 0, 0);
                const description = doc.splitTextToSize(finding.description, contentWidth);
                doc.text(description, page.margin, yPos);
                yPos += (description.length * 12) + 20;
            });
        }
        
        if (yPos > doc.internal.pageSize.getHeight() - 80) { doc.addPage(); yPos = page.margin; }
        doc.setLineWidth(0.5).line(page.margin, yPos, page.width - page.margin, yPos);
        yPos += 20;
        
        doc.setFontSize(8).setTextColor(128, 128, 128);
        const disclaimer = "Disclaimer: This analysis is generated by an AI for illustrative purposes and is not a medical diagnosis. Consult a qualified healthcare professional for any health concerns.";
        doc.text(doc.splitTextToSize(disclaimer, contentWidth), page.margin, yPos);
        
        const diseaseName = analysisResult.status === 'anomaly_detected' && analysisResult.findings.length > 0 ? analysisResult.findings[0].finding.replace(/[\s/]/g, '_').toLowerCase() : 'healthy';
        doc.save(analysis_report_${diseaseName}.pdf);

    } catch (err) {
        console.error("PDF Generation failed:", err);
        setError("Failed to generate PDF report.");
    } finally {
        setIsDownloading(false);
    }
}, [createAnnotatedCanvas, analysisResult]);


  return (
    <div className="min-h-screen bg-slate-100 flex flex-col items-center p-4 sm:p-6 lg:p-8">
      <div className="w-full max-w-5xl mx-auto">
        <header className="text-center mb-8">
          <div className="flex justify-center items-center gap-4 mb-2">
            <BrainCircuitIcon className="h-10 w-10 text-blue-600" />
            <h1 className="text-4xl font-bold text-slate-800 tracking-tight">
              Medical Scan Analyzer AI
            </h1>
          </div>
          <p className="text-slate-600 max-w-2xl mx-auto">
            Upload a medical scan to let our advanced AI identify, highlight, and describe potential anomalies.
          </p>
        </header>

        <main className="bg-white rounded-2xl shadow-lg p-6 md:p-8">
          {!selectedFile ? (
            <ImageUploader onFileSelect={handleFileSelect} />
          ) : (
            <div className="flex flex-col lg:flex-row gap-8">
              <div className="lg:w-1/2 flex flex-col items-center">
                 <div className="w-full max-w-md aspect-square bg-slate-100 rounded-lg overflow-hidden border border-slate-200 flex items-center justify-center">
                    {imageUrl && <img src={imageUrl} alt="Medical Scan" className={max-w-full max-h-full object-contain transition-all duration-300 ${enhanceImage ? 'image-enhanced' : ''}} />}
                </div>
                
                <div className="mt-4 flex items-center justify-center w-full max-w-md">
                  <input
                    type="checkbox"
                    id="enhance"
                    checked={enhanceImage}
                    onChange={(e) => setEnhanceImage(e.target.checked)}
                    disabled={isLoading}
                    className="h-4 w-4 rounded border-gray-300 text-blue-600 focus:ring-blue-500 disabled:opacity-50"
                  />
                  <label htmlFor="enhance" className="ml-2 block text-sm text-slate-700">
                    Enhance Image
                  </label>
                </div>

                <div className="flex items-center gap-4 mt-4 w-full max-w-md">
                    {analysisResult ? (
                         <div className="flex-grow flex gap-2">
                            <button
                                onClick={handleImageDownload}
                                disabled={isDownloading}
                                className="flex-1 inline-flex items-center justify-center gap-2 px-4 py-3 border border-transparent text-base font-medium rounded-md shadow-sm text-white bg-green-600 hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500 disabled:bg-slate-400 disabled:cursor-not-allowed transition-colors"
                            >
                                <DownloadIcon className="h-5 w-5" />
                                {isDownloading ? 'Saving...' : 'Download Image'}
                            </button>
                            <button
                                onClick={handlePdfDownload}
                                disabled={isDownloading}
                                className="flex-1 inline-flex items-center justify-center gap-2 px-4 py-3 border border-transparent text-base font-medium rounded-md shadow-sm text-white bg-red-600 hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-red-500 disabled:bg-slate-400 disabled:cursor-not-allowed transition-colors"
                            >
                                <PdfIcon className="h-5 w-5" />
                                {isDownloading ? 'Generating...' : 'Download PDF'}
                            </button>
                        </div>
                    ) : (
                        <button
                            onClick={handleAnalyzeClick}
                            disabled={isLoading || !imageDimensions}
                            className="flex-grow inline-flex items-center justify-center px-6 py-3 border border-transparent text-base font-medium rounded-md shadow-sm text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:bg-slate-400 disabled:cursor-not-allowed transition-colors"
                        >
                            {isLoading ? 'Analyzing...' : 'Analyze Scan'}
                        </button>
                    )}
                    <button
                        onClick={handleReset}
                        disabled={isLoading || isDownloading}
                        className="px-6 py-3 border border-slate-300 text-base font-medium rounded-md text-slate-700 bg-white hover:bg-slate-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50"
                    >
                        Reset
                    </button>
                </div>
              </div>

              <div className="lg:w-1/2">
                <div className="w-full h-full min-h-[300px] lg:min-h-0 bg-slate-50 rounded-lg border border-dashed border-slate-300 flex items-center justify-center p-4">
                  {isLoading && <Spinner />}
                  {error && <div className="text-center text-red-600 bg-red-50 p-4 rounded-lg">
                    <h3 className="font-semibold">Analysis Error</h3>
                    <p>{error}</p>
                    </div>}
                  {analysisResult && imageDimensions && imageUrl && (
                    <AnalysisDisplay 
                      imageUrl={imageUrl}
                      imageDimensions={imageDimensions}
                      result={analysisResult} 
                    />
                  )}
                  {!isLoading && !error && !analysisResult && (
                     <div className="text-center text-slate-500">
                        <UploadCloudIcon className="mx-auto h-12 w-12" />
                        <h3 className="mt-2 text-lg font-medium">Analysis Results</h3>
                        <p className="mt-1 text-sm">Click "Analyze Scan" to see the AI's findings here.</p>
                     </div>
                  )}
                </div>
              </div>
            </div>
          )}
        </main>
      </div>
    </div>
  );
}
