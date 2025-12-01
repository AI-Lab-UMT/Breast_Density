# An Efficient, Robust and Explainable Design for Assessing Breast Density in Mammographies

Breast density is a key factor in assessing cancer suspicion in mammographies: **the denser the breast, the harder it becomes to detect tumors**.  
The *Breast Imaging Reporting & Data System (BI-RADS)* of the American College of Radiology classifies mammographies into four density categories: **A, B, C, and D**, indicating increasing breast density.

This project addresses two classification problems:

- **Binary classification:** A/B vs. C/D (low vs. high breast density)  
- **Multi-class classification:** A vs. B vs. C vs. D  

All experiments were conducted on the **VinDr-Mammo** public dataset of 20,000 mammographic images, specifically using **7,622 cranio-caudal (CC) mammograms** acquired from a *Siemens Mammomat Inspiration* system.

----------------------------------------------------
## Proposed Markers for Density Assessment
----------------------------------------------------
We introduce **four histogram-based markers of pixel intensity** that show strong correlation with BI-RADS density categories:

1. **Histogram Bimodality** – captures the presence of dual intensity modes.  
2. **Heterogeneity in Bright Intensity Range** – measures local variations in dense tissues.  
3. **Ratio of Bright to Dark Heterogeneity** – contrasts dense and fatty tissue distributions.  
4. **Ratio of Very Bright to Very Dark Heterogeneity** – focuses on extreme intensity structures.

These markers are simple to compute, interpretable, and directly track morphological patterns linked to breast density.

----------------------------------------------------
## Classification Model
----------------------------------------------------
A small, lightweight deep learning model was used:

- **One input per marker**
- **One hidden layer with 16 ReLU units**
- **2-output head** for A/B vs. C/D (128 learnable parameters)
- **4-output head** for A/B/C/D (256 learnable parameters)

Despite its extremely small size compared to conventional CNNs (which often require hundreds of thousands to millions of parameters), the model achieved state-of-the-art performance.

----------------------------------------------------
## Results (5-Fold Cross Validation on 7,622 CC Mammograms)
----------------------------------------------------
### **A/B vs. C/D (Binary Classification)**  
- **Accuracy:** 94.25% ± 0.027  
- **F1 Score:** 93.86% ± 0.033  

### **A/B/C/D (4-Class Classification)**  
- **Accuracy:** 83.18% ± 0.100  
- **F1 Score:** 81.40% ± 0.117  

These results match or outperform more complex convolutional approaches reported in the literature, demonstrating the strength of our simple yet interpretable marker-based model.

----------------------------------------------------
## Why This Method Is Robust and Explainable
----------------------------------------------------
- **Explainability:**  
  Each marker has a clear physical interpretation related to tissue distribution, making the model transparent and clinically intuitive.

- **Reproducibility:**  
  The four markers are easily computed from intensity histograms, and the classifier is small enough to be implemented in any open-source ML framework.

- **Robustness:**  
  Cross-validation confirms consistent performance across subsets of the dataset.

- **Efficiency:**  
  Training is extremely fast due to the compact model size, enabling rapid experimentation and validation.

----------------------------------------------------
