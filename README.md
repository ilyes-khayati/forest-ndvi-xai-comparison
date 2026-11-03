# forest-ndvi-xai-comparison

Do different model classes explain the same wavelet-decomposed NDVI signal the
same way in a forest? The question and the pipeline of `olive-ndvi-xai-comparison`,
applied to an oak forest block in the Kroumirie mountains, Jendouba, Tunisia.

## Study area

`forest-jendouba-polygon.geojson`: a 1 km² block centred at 36.7586 N, 8.7016 E,
chosen as fully tree-covered on ESA WorldCover v200, mean elevation 740 m (SRTM).
16 MODIS NDVI pixels at 250 m, and one MODIS LST pixel at 1 km.

## Notebooks

01  data acquisition        polygon extraction, MODIS quality screening, SPEI 1-48 months
02  wavelet decomposition   discrete Meyer at native cadence; look-ahead quantified
03  feature derivation      every predictor justified from evidence
04  model training          XGBoost, Random Forest, Ridge; noise floor; rolling origin
05  attribution             permutation, drop-column, grouped and SHAP, across models

The code is that of the olive repository with the site changed; the notes in the
notebooks are still to be written for the forest.

## Setup

Uses the virtual environment from `olive-ndvi-wavelet-xai`. Select the
`olive-venv` kernel.
