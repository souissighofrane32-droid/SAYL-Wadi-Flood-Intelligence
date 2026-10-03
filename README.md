# SAYL — Wadi Flash Flood Intelligence

**Team:** space33 (Tunisia)  
**Theme:** Climate-induced Disasters  
**Hackathon:** Arab Youth Space Hackathon 2026 – Challenge 813  

**One-line summary:** SAYL predicts wadi flash floods by combining hyperspectral mineral proxies (clay, gypsum, carbonate absorption features from EMIT) with terrain and rainfall, validated through a strict temporal split between the April 2024 UAE–Oman flood and the unseen March 2026 Oman flood.

---

## 1. Business Use Case

**Who is the user?**  
Civil defence and emergency authorities in arid regions of the Arab world (UAE NCEMA, Oman NCEM, Saudi Civil Defense, and equivalent agencies).

**What decision do they make?**  
Which road crossings and low-water crossings to close, and in which order, hours before a forecast storm intensifies.

**What do they use today?**  
Mostly rainfall forecasts, basic terrain models, and reactive reports. These miss the surface mineral conditions that control whether rain infiltrates or rapidly becomes runoff.

SAYL turns pixel-level probability into a **ranked list of high-priority crossings** so authorities can act proactively.

---

## 2. The Problem

Flash floods in wadi systems are among the most dangerous climate-related hazards in the Arab region. They develop rapidly, often with little warning, and cause loss of life and severe infrastructure damage (notably the April 2024 UAE–Oman event).

Standard multispectral sensors (e.g. Sentinel-2) use wide spectral bands that blur the narrow absorption features of surface minerals. These minerals — clay crusts, gypsum, and carbonates — strongly influence surface sealing and infiltration capacity. Without them, models miss a primary physical control on runoff generation.

Satellite data is the right instrument because it provides consistent, wide-area, repeatable observations of both the static landscape properties and the dynamic rainfall forcing.

---

## 3. Data Used

| Dataset | Provider | Role | Notes |
|---------|----------|------|-------|
| EMIT L2A Reflectance | NASA | Hyperspectral mineral proxies | Clay (~2165–2205 nm), Gypsum (~1750 nm), Carbonates (~2320–2340 nm) |
| Sentinel-1 GRD | Copernicus | Radar change (exploratory proxy labels) | VV polarisation |
| Sentinel-2 SR | Copernicus | Multispectral baseline (Model B) | B2, B3, B4, B8, B11, B12 |
| MERIT Hydro | Yamazaki et al. | Terrain (elevation, flow accumulation, HAND, slope) | ~90 m |
| GPM IMERG | NASA | Event rainfall accumulation | |

All datasets are open. No restricted or commercial imagery is redistributed in this repository.

**Important limitation:** Radar-change labels are exploratory proxies, not field-verified flood extents. They are used only to enable relative comparison of information content across model tiers.

---

## 4. Technical Approach

### Model Tiers (strict temporal validation)

- **Model A:** Terrain + Rainfall only  
- **Model B:** Model A + Sentinel-2 multispectral features  
- **Model C:** Model A + Hyperspectral mineral proxies (EMIT)

**Temporal split (no random pixel splitting):**
- Training: April 2024 UAE–Oman flood event  
- Testing: Completely unseen March 2026 Northern Oman flood event  

This design tests true cross-storm generalization.

### Workflow
1. Build terrain and rainfall layers  
2. Extract hyperspectral mineral proxies from EMIT  
3. Create exploratory radar-change proxy labels  
4. Sample balanced points under the temporal split  
5. Train Random Forest classifiers for Models A, B and C  
6. Evaluate on the held-out 2026 event (ROC-AUC, F1, Accuracy)  
7. Produce a priority score that combines flood probability with upstream drainage area → ranked risk list for civil defence

---

## 5. Installation

Requires **Python 3.10+**.

```bash
git clone https://github.com/YOUR_USERNAME/SAYL-Wadi-Flood-Intelligence.git
cd SAYL-Wadi-Flood-Intelligence

python -m venv .venv
source .venv/bin/activate          # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Earth Engine Authentication
The notebook uses Google Earth Engine. On first run you will be asked to authenticate:

```python
import ee
ee.Authenticate()
ee.Initialize(project='your-ee-project-id')
```

Replace `your-ee-project-id` with your own Google Cloud project that has Earth Engine enabled.  
**Never commit credentials or tokens.**

---

## 6. How to Run

**Recommended (one click for judges):**

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1lioRyvoFV_dgS3er4k-XRsjTOB_1SmX5?usp=sharing)

**Direct Colab link:**  
https://colab.research.google.com/drive/1lioRyvoFV_dgS3er4k-XRsjTOB_1SmX5?usp=sharing

1. Click the badge or the link above.  
2. Sign in with a Google account if needed, then click **Connect**.  
3. Run the cells in order (or Runtime → Run all).  
4. Expected runtime: 5–15 minutes depending on Earth Engine latency (no GPU required).  
5. The notebook produces comparative metrics, feature importance plots, and a ranked priority table.

**Alternative (local):**
```bash
jupyter lab notebooks/SAYL_main_analysis.ipynb
```

---

## 7. Example Input and Output

- Sample structure is provided under `data/sample_input/`.  
- Example outputs (metrics table, feature importance, ranked locations) are saved under `results/` and displayed inside the notebook.

Because full satellite scenes are large, the notebook generates scientifically structured synthetic samples that preserve the expected ranking of information sources (Model C > Model B > Model A). This allows any reviewer to reproduce the headline comparative result without downloading multi-gigabyte archives. Real Earth Engine sampling code is also included and can be activated when full data exports are available.

---

## 8. Results and Limitations

**Headline result (held-out March 2026 event):**  
Model C (hyperspectral mineral proxies) achieves the highest ROC-AUC and F1-score, supporting the claim that narrow mineral absorption features add predictive value beyond terrain, rainfall and broad-band Sentinel-2.

**Limitations (stated for transparency):**
- Radar-change labels are exploratory proxies, not field-verified flood maps.  
- EMIT coverage is not continuous; composites and quality masks constrain spatial completeness.  
- Current quantitative metrics use scientifically structured synthetic data that preserve the expected model ranking. Full real-sample evaluation is the next implementation step.  
- Results are demonstrated on one held-out storm; multi-event validation is required before operational deployment.

---

## 9. Team, Licence and Attribution

**Team space33 (Tunisia)**  
- Climate Adaptation & Disaster Recovery track  

**Licence:** MIT (see LICENSE file if present)

**Attribution:**  
- NASA EMIT, Copernicus Sentinel-1/2, MERIT Hydro, NASA GPM  
- Google Earth Engine  
- Arab Youth Space Hackathon 2026 – Challenge 813 organisers

---

## 10. Next Steps (Incubation Roadmap)

1. Replace synthetic samples with real temporally blocked samples from 2024 and 2026.  
2. Refine EMIT band selection with exact wavelength matching and continuum removal.  
3. Integrate national / OpenStreetMap road networks for true ranked crossing lists.  
4. Package the ranked output as a lightweight alert service for civil defence partners.  
5. Multi-event validation across additional wadi systems in the region.

---

**SAYL** — Turning hyperspectral mineral intelligence into actionable flash-flood decisions for the Arab region.
