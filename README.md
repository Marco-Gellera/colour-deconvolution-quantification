# Color Deconvolution Quantification for Staining Kits of *in Vitro* Cell Models

**DOI:** [10.5281/zenodo.20747341](https://doi.org/10.5281/zenodo.20747341)
**Code licence:** [MIT](LICENSE_code.txt)  
**Data licence:** [CC-BY 4.0](LICENSE_data.txt)  
**Software:** [ImageJ 1.54t](https://imagej.net)

---

## Authors

**Marco Gellera** — PhD Student  
University of Milan, Department of Food, Environmental and Nutritional Sciences (DeFENS)  
ORCID: [0009-0001-8096-1148](https://orcid.org/0009-0001-8096-1148)

**Mirko Marino** — Fixed-term Research Fellow A  
University of Milan, Department of Food, Environmental and Nutritional Sciences (DeFENS)  
ORCID: [0000-0001-9913-4380](https://orcid.org/0000-0001-9913-4380)

**Cristian Del Bo'** — Associate Professor  
University of Milan, Department of Food, Environmental and Nutritional Sciences (DeFENS)  
ORCID: [0000-0001-7562-377X](https://orcid.org/0000-0001-7562-377X)

---

## Description

This repository provides an ImageJ macro for the automated, reproducible quantification of chromogenic staining intensity in brightfield microscopy images of *in vitro* cell models. The macro implements a **colour deconvolution-based approach** with interactive, user-defined stain vector calibration, enabling application across different staining kits without modification to the core code.

The method was originally developed in the context of an *in vitro* study on the effects of xanthine metabolites on D-galactose-induced senescence in Caco-2 cells, using a Senescence-Associated β-galactosidase (SA-β-gal) detection kit. However, the workflow can be adapted to other single-chromogen stainings producing a spectrally distinct signal on a brightfield background, provided that (i) images are acquired under consistent illumination and white balance settings, and (ii) the stain and background spectral signatures are sufficiently separated in RGB optical density space.

### Biological context

Cellular senescence is a state of stable cell cycle arrest associated with a characteristic secretory phenotype. SA-β-galactosidase activity, detectable at pH 6.0 via X-gal substrate conversion to a blue-green chromogenic product, is the most widely used histochemical marker of senescence. Accurate quantification of staining intensity — beyond binary positive/negative scoring — is required to capture graded changes in senescence burden across experimental conditions. The present macro addresses this need by providing a continuous, image-level metric (mean optical density of the deconvolved stain channel) suitable for statistical comparison across treatment groups.

---

## Requirements

| Software | Version tested | Notes |
|---|---|---|
| ImageJ | 1.54t | Also compatible with Fiji (ImageJ2) |
| Colour Deconvolution2 | ≥ 2.0 | Plugin by Gabriel Landini — see Installation |

**Operating system:** tested on Windows 10/11. Expected to be platform-independent.  
**Input format:** RGB TIFF images (8-bit per channel). JPEG input is not recommended due to lossy compression artefacts affecting optical density calculations.

---

## Installation

### 1. ImageJ
Download ImageJ from [https://imagej.net/ij/download.html](https://imagej.net/ij/download.html) or use the Fiji distribution from [https://fiji.sc](https://fiji.sc).

### 2. Colour Deconvolution2 plugin
1. Download the plugin `.jar` file from:  
   [https://blog.bham.ac.uk/intellimic/g-landini-software/colour-deconvolution-2/](https://blog.bham.ac.uk/intellimic/g-landini-software/colour-deconvolution-2/)
2. Copy the `.jar` file into the `ImageJ/plugins/` directory (or `Fiji/plugins/`).
3. Restart ImageJ. The plugin will appear under `Image → Color → Colour Deconvolution2`.

### 3. Macro installation
1. Open ImageJ.
2. Go to `Plugins → Macros → Install...`
3. Select `macro_colour_deconvolution_quantification.ijm`.  
   Alternatively, open the macro directly via `Plugins → Macros → Run...` for single-session use.

---

## Usage

### Input requirements
- All images must be **RGB TIFF** files stored in a single folder.
- Images must be acquired under **identical conditions** (magnification, exposure, white balance, staining protocol) within each batch. Cross-batch comparisons require re-calibration of stain vectors.

### Step-by-step workflow

**Step 1 — Launch the macro**  
Run the macro via `Plugins → Macros → macro_colour_deconvolution_quantification`. A dialog will prompt you to select the input folder containing the TIFF images.

**Step 2 — Stain vector calibration (interactive, performed once per batch)**  
The macro automatically opens the first TIFF in the folder as the calibration image.

- **ROI 1 (Stain vector):** Draw a region of interest (ROI) over an area of **pure, intense staining** — the most representative positively stained region, avoiding edges, debris, and mixed areas. Press OK.
- **ROI 2 (Background vector):** Draw a ROI over an area of **pure background** — unstained regions, or cell-free areas of the well in order to separate the signal from the background. Press OK.

The macro computes the optical density (OD) vector for each ROI, normalises both vectors, and calculates the third orthogonal vector via cross product. Calibrated vectors are displayed in a confirmation dialog and logged to the ImageJ Log window at full numerical precision.

**Step 3 — Batch processing**  
After confirmation, the macro processes all TIFF files in the selected folder sequentially. For each image:
1. Colour Deconvolution2 is applied using the calibrated vectors.
2. The `Colour_1` output channel (corresponding to the stain) is inverted.
3. Mean grey value, Integrated Density, and Area are measured over the entire image.

**Step 4 — Output**  
Results are saved as `results_staining.csv` in the input folder. The CSV header includes the calibrated stain vectors used, ensuring full traceability.

### Output file structure

```
# Stain vectors used for this batch:
# v1=[...] v2=[...] v3=[...]
Filename, Mean_Stain, IntDen_Stain, Area_Total
image_01.tiff, 42.31, 351084672, 8294400
image_02.tiff, 38.76, 321643776, 8294400
...
```

| Column | Description |
|---|---|
| `Filename` | Original image filename |
| `Mean_Stain` | Mean grey value of the deconvolved stain channel (primary outcome) |
| `IntDen_Stain` | Integrated Density = Area × Mean (useful for area-normalised comparisons) |
| `Area_Total` | Total measured area in pixels |

**Primary outcome variable:** `Mean_Stain`. This represents the mean intensity of the inverted deconvolved stain channel and is used as an image-level proxy of staining burden.This macro does not perform cell segmentation. Therefore, Mean_Stain should not be interpreted as per-cell staining intensity unless cell density and field composition are comparable across group. If confluency varies systematically between experimental groups, normalisation by cell area is recommended prior to statistical analysis.

---

## Methodological rationale

### Colour deconvolution
Colour deconvolution (Ruifrok & Johnston, 2001) decomposes a multi-stain brightfield image into independent single-stain optical density maps by solving a linear system in OD space. Unlike direct RGB channel subtraction, this approach accounts for spectral overlap between stains and provides a physically grounded separation. The method requires knowledge of the stain vectors — the unit OD vectors characterising each pure stain — which in this macro are estimated empirically from user-selected ROIs rather than from literature values, ensuring calibration to the specific imaging setup and staining batch.

### Stain vector computation
For each ROI, the mean RGB values are converted to optical density:

$$OD_c = -\log\left(\frac{I_c}{255} + \epsilon\right), \quad c \in \{R, G, B\}$$

where $\epsilon = 0.001$ prevents log(0) instability. The OD triplet is then L2-normalised to obtain the unit stain vector. The third vector is computed as the cross product of v1 and v2 and subsequently normalised, guaranteeing orthogonality and a complete basis for the deconvolution.

### Limitations
- The method assumes Beer-Lambert linearity between stain concentration and optical density, which may not hold at very high staining intensities (OD > 2.0).
- Stain vectors are batch-specific. Vectors calibrated on one experimental batch should not be applied to images acquired under different conditions without re-validation.
- The macro measures the entire image field of view. If images contain regions outside the well or large cell-free artefacts, a fixed ROI mask should be applied consistently across all images prior to batch processing.
- JPEG images should not be used as input, as loss of compression introduces systematic errors in OD calculations.

---

## Example data

The `example_images/` folder contains a representative subset of the dataset used during method development:

- `negative_controls/` — 11 images: Caco-2 cells without D-galactose treatment (SA-β-gal negative)
- `positive_controls/` — 12 images: Caco-2 cells with D-galactose treatment (SA-β-gal positive)

> **Note:** Example images (avaibale at https://doi.org/10.5281/zenodo.20747341) are archived on Zenodo under **embargo** until publication of the associated manuscript. They will be made publicly available upon paper acceptance. The macro and documentation are immediately available under the terms of their respective licences.

Expected output for the example dataset is provided in `example_output/results_example.csv`.

---

## Repository structure

```
├── macro_colour_deconvolution_quantification.ijm   # ImageJ macro (MIT licence)
├── README.md                                        # This file (CC-BY 4.0)
├── LICENSE_code.txt                                 # MIT licence (macro)
├── LICENSE_data.txt                                 # CC-BY 4.0 (documentation, data)
├── example_images/
│   ├── negative_controls/                           # 11 TIFF images (embargoed)
│   └── positive_controls/                           # 12 TIFF images (embargoed)
└── example_output/
    └── results_example.csv                          # Expected output
```

---

## Citation

If you use this macro in your research, please cite:

> Gellera M, Marino M, Del Bo' C. *Color Deconvolution Quantification for Staining Kits of in Vitro Cell Models* [Software]. Zenodo. https://doi.org/10.5281/zenodo.20747341

A BibTeX entry is provided below for convenience:

```bibtex
@software{gellera_xgal_macro,
  author       = {Gellera, Marco and Marino, Mirko and {Del Bo'}, Cristian},
  title        = {Color Deconvolution Quantification for Staining Kits 
                  of in Vitro Cell Models},
  year         = {2026},
  publisher    = {Zenodo},
  doi          = {10.5281/zenodo.20747341},
  url          = {https://doi.org/10.5281/zenodo.20747341}
}
```

---

## References

Ruifrok AC, Johnston DA. Quantification of histochemical staining by color deconvolution. *Analytical and Quantitative Cytology and Histology*. 2001;23(4):291–299.

Landini G. Colour Deconvolution2 plugin for ImageJ. University of Birmingham. Available at: https://blog.bham.ac.uk/intellimic/g-landini-software/colour-deconvolution-2/

Dimri GP, Lee X, Basile G, et al. A biomarker that identifies senescent human cells in culture and in aging skin in vivo. *Proceedings of the National Academy of Sciences*. 1995;92(20):9363–9367.

---

## Licence

- **Macro code** (`*.ijm`): [MIT License](LICENSE_code.txt)
- **Documentation, README, and example data** (`*.md`, `*.csv`, `*.tiff`): [Creative Commons Attribution 4.0 International (CC-BY 4.0)](LICENSE_data.txt)

Both licences require attribution. Users are requested to cite this repository when using the macro in published research (see Citation section above).
