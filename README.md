# Antarctic Sea Ice Edge Modeling

Analysis, mathematical modeling, and visualization of Antarctic sea ice edge changes over time using daily observations at 360 longitudes.

## Project Overview

The project examines the position of the Antarctic sea ice edge over time. Its main goal is to describe the ice edge at individual longitudes, build a global model combining spatial and temporal information, and compare the modeled contours with observations.

The analysis is implemented in Python in a Jupyter notebook and covers four parts:

1. Identifying and visualizing the minimum observed sea ice extent at each longitude.
2. Fitting a separate time-dependent model for each of the 360 longitudes.
3. Building a single global model that captures spatial and temporal variation.
4. Animating and comparing the observed ice edge with both sets of predictions.

## Dataset

The input file is `daily_ice_edge.csv`, supplied for the course project. It contains a `Date` column and columns named `longitude_0E` through `longitude_359E`. The longitude columns hold latitude values describing the sea ice edge for the corresponding date and longitude.

Some observations are missing. The notebook excludes missing latitude values when fitting models and evaluating their fit.

**The input dataset is not included in this repository package.** Place `daily_ice_edge.csv` next to the notebook before running it. Check that you have permission to publish the course dataset before adding it to a public GitHub repository.

## Methodology

### 1. Minimum sea ice edge

For each longitude, the notebook selects the minimum recorded latitude and plots the resulting closed contour on a polar chart. This provides an overview of the southernmost observed ice-edge positions in the available series.

Output: `minimal_ice_extent.png`

### 2. Local Fourier models

An independent linear regression model is fitted for every longitude, using a linear time trend and annual sine/cosine terms:

\[
\widehat{\mathrm{Lat}}_i(t) = a_{0,i} + a_{1,i}t + a_{2,i}\sin\left(\frac{2\pi t}{365.25}\right) + a_{3,i}\cos\left(\frac{2\pi t}{365.25}\right)
\]

Here, \(t\) is the number of days since the first date in the dataset. Models are fitted only on available measurements at their respective longitudes.

Outputs: `ice_model_parameters.csv`, `modeled_daily_ice_edge.csv`

### 3. Global gradient boosting model

A `HistGradientBoostingRegressor` from scikit-learn is fitted to all available longitude–date–latitude observations. The predictor set includes:

- Time in days and annual sine/cosine terms.
- Longitude represented by sine and cosine harmonics (orders 1–8).
- Interactions between the longitude harmonics and seasonal terms.

The model estimates latitude as a nonlinear function of longitude and time, \(\widehat{\mathrm{Lat}} = f(\mathrm{Lon}, t)\). Latitude is the **predicted output**, not an input feature.

### 4. Comparison and animation

The notebook creates a polar animation showing three contours:

- Observed sea ice edge.
- Local Fourier model predictions.
- Global gradient boosting model predictions.

The animation takes every 10th row of the original dataset and exports the frames as a GIF.

Output: `ice_animation_both_methods.gif`

## Model Comparison

The following values are shown in the project notebook/supplied execution screenshot:

| Model | MAE | RMSE | R² |
| --- | ---: | ---: | ---: |
| Local Fourier model | 1.1866 | 1.6282 | 0.8448 |
| Global gradient boosting model | 0.7692 | 1.0569 | 0.9346 |

MAE and RMSE measure errors in **latitude degrees**, while R² measures goodness of fit. The global model has lower errors and a higher R² on the evaluated observations.

**Important:** These metrics are calculated on observations used during model fitting. No separate test set or time-based validation is implemented in the current notebook, so the results should not be interpreted as performance on unseen dates.

Output: `comparison_metrics.csv`

## Repository Structure

```text
antarctic-ice-edge-modeling/
├── Projekt_wersja2.ipynb        # Main analysis and modeling notebook
├── README.md                   # Project documentation
├── requirements.txt            # Python dependencies
└── .gitignore                  # Local/Jupyter files to exclude
```

Add `daily_ice_edge.csv` to the same directory before running the notebook. Running all notebook cells generates these files **in the current working directory**:

```text
minimal_ice_extent.png
ice_model_parameters.csv
modeled_daily_ice_edge.csv
ice_animation_both_methods.gif
comparison_metrics.csv
```

The generated outputs shown in the Jupyter screenshot were not provided as separate files and are not part of this package. You can add them to GitHub after generating or downloading them from JupyterLab.

## How to Run

1. Download or clone the repository.
2. Place `daily_ice_edge.csv` in the repository directory.
3. Install the required packages:

   ```bash
   python -m pip install -r requirements.txt
   ```

4. Start JupyterLab from the repository directory:

   ```bash
   jupyter lab
   ```

5. Open `Projekt_wersja2.ipynb` and run the cells from top to bottom.

Creating the GIF may take some time depending on the size of the input data and your computer.

## Technologies

- Python
- Pandas and NumPy
- Matplotlib and Pillow
- scikit-learn
- JupyterLab

## Limitations

- Missing latitude observations are excluded during fitting; their meaning is not inferred automatically.
- Both models are evaluated on training observations, without a time-based holdout set.
- The local model assumes a linear trend and a single annual seasonal harmonic.
- The global model predicts ice-edge latitude from longitude and time; it does not directly calculate total ice-covered area.

## Project Context

Academic course project on Antarctic sea ice edge modeling and visualization.
