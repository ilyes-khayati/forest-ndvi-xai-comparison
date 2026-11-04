# forest-ndvi-xai-comparison

Do different model classes explain the same wavelet-decomposed NDVI forecast the
same way? XGBoost, Random Forest and Ridge regression are fitted to identical inputs
over an oak forest block in the Kroumirie mountains, Jendouba, Tunisia, and their
feature attributions compared.

This is the forest half of a two-site comparison. The olive half is
[`olive-ndvi-xai-comparison`](https://github.com/ilyes-khayati/olive-ndvi-xai-comparison);
both run notebooks 01–05 with the same code, so differences between the sites come
from the vegetation, not from the pipeline.

## Study area

`forest-jendouba-polygon.geojson`: a 1 km² block centred at 36.7586 N, 8.7016 E,
mean elevation 740 m (SRTM), 100% tree cover on ESA WorldCover v200.

- NDVI (MODIS, 250 m): the block covers 16 pixels' worth of area and touches 24.
- LST (MODIS, 1 km): it has the area of one pixel but straddles four, so LST includes
  some of the surrounding forest.
- Kroumirie has both cork oak (evergreen) and zeen oak (deciduous). NDVI has a
  persistent low in late winter, which suggests a deciduous component; the species
  composition of the block is not yet confirmed.

## Protocol

| Step | Choice |
|---|---|
| NDVI quality | per-pixel `SummaryQA` = 0 only; a composite is dropped when fewer than half its pixels are good |
| Decomposition | discrete Meyer wavelet at native cadence (NDVI level 5, LST 6, SPEI 4); trend and seasonal branches |
| Own history | lasso with time-series CV and the one-standard-error rule; calendar given to every model on the seasonal branches |
| Lagged climate | LST and SPEI 1–48 at lags 1–12, kept when the correlation with the stage-1 residual passes a Bonferroni bound with effective sample size; at most one drought lag per branch |
| Probes | LST lag 1 kept although the linear screen rejects it; a Gaussian noise probe |
| Models | XGBoost, Random Forest, Ridge; randomised search (100 draws) or grid on a five-fold TimeSeriesSplit |
| Evaluation | test 2022–2024 once; 20-seed noise floor; five rolling windows, each tuned on its own training years |
| Attribution | out-of-fold permutation, drop-column, family permutation, SHAP; agreement by weighted Kendall τ against each model's agreement with itself |
| Mechanism | predictor overlap with the calendar, clock-removal refits, LST climatology vs anomaly, SHAP dependence, SHAP interactions |

## Notebooks

    01  data acquisition        polygon extraction, per-pixel quality mask, SPEI 1-48 months
    02  wavelet decomposition   discrete Meyer at native cadence; look-ahead quantified
    03  feature derivation      every predictor justified from evidence; registry of decisions
    04  model training          XGBoost, Random Forest, Ridge; noise floor; rolling origin
    05  attribution             explanations across models, and what each model needs

Each notebook writes to `data/processed/`, `results/` and `figures/`. The extracted
series in `data/raw/` are committed (about 200 KB), so notebooks 02–05 run without an
Earth Engine account.

## Running it

```sh
git clone https://github.com/ilyes-khayati/forest-ndvi-xai-comparison.git
cd forest-ndvi-xai-comparison
python3.12 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python -m ipykernel install --user --name forest-xai
```

Open the notebooks with the `forest-xai` kernel and run 02 to 05 in order. Notebook
01 re-extracts the data from Google Earth Engine and needs an account:

```sh
earthengine authenticate
export EE_PROJECT=<your Earth Engine project id>
```

Notebook 04 takes about half an hour on a laptop, because each rolling window is
tuned again; notebook 05 a similar time.

## Known limitations

- The trend branch is built by a two-sided filter, so its scores include look-ahead
  (measured in notebook 02); no forecast-skill claim is made on it.
- NDVI over dense canopy is close to saturation, so month-to-month variation is small
  and a large share of it is sensor and atmospheric noise. Errors are small in NDVI
  units while R² is modest.
- One site. The result is a case study until repeated on other forest blocks.
