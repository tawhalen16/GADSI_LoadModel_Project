# Recurrent Neural Networks in PJM Load Modeling

## Problem Statement
Platts produces multiple reports based on estimates of electrical load, including backcasts that attempt to remove the effect of weather as well short-term forecasts. These estimates largely rely on conventional regression models with some form of regularization. In particular, Platts released a new weekly forecast product covering total market trends in the PJM regional transmission organization, which estimates generation largely on the basis of forecast load.

In this project, I aimed to implement a more complex neural net model to determine whether the PJM load model could:
1. Significantly improve overall accuracy
1. Significantly improve peak hour accuracy
1. Retain short model runtimes to fit production timelines

## Summary

Current load models rely on various forms of LASSO regression, incorporating features for every relevant time frame (hour, day of week, month) as well as multiple measures of weather (temperature, dew point, wind speed, relative humidity). Including interaction and power effects, these models can include thousands of features in an attempt to capture higher level relationships to load. While LASSO models have proven fairly effective at forecasting load, power demand fundamentally violates assumptions of both linear regressions (via interdependence of observations) and autoregressive models (via multiple forms of seasonality).

![Seasonality Patterns](/Charts/Presentation%20-%20Seasonality%20Patterns.png?raw=true)

My hope was that the use of recurrent neural networks could serve in place of these extremely wide datasets, using deeper learning in place of extensive interaction variables. Recurrent neural nets have alread seen significant use in load forecasting, several of which can be found within the [Literature](Literature/) folder. With sufficient data, these models can mitigate concerns about data stationarity.

![Weather Relationships](/Charts/Presentation%20-%20Weather%20Relationships.png?raw=true)

In addition to the strong seasonality, the charts above illustrate the extremely strong relationships to temperature on an hourly basis, providing a solid foundation for these models without necessarily the need to collect additional data. The load and weather data used here comes primarily from S&P Global Platts' proprietary databases. As such, collections, cleaning, and querying scripts are not included in this repository. Similarly, the core files for the weekly model are also not included within the repository, though several files were used as the basis for scripts that are included.

## Results

The new validation process created for the existing LASSO model primarily illustrated that the current approach already offers a high degree of accuracy. High temperatures in September 2019 likely help explain the spike in percent error.

![Validation MAPE - LASSO](/Charts/Validation%20MAPE%20-%20LASSO.png?raw=true)

In contrast, the results from the initial runs of this project's RNN model proved to no better than the LASSO at best, and wildly unpredictable on a week-to-week basis.

![Validation MAPE - RNN](/Charts/Validation%20MAPE%20-%20RNN.png?raw=true)

Results for peak load predictions were very much the same, with large swings in peak load error.

![Validation MAPE - Peak Hour](/Charts/Validation%20MAPE%20-%20Peak%20Hour.png?raw=true)

## Conclusions

Given its high volatility and comparatively low accuracy, the initial approach seems unlikely to result in a model capable of challenging the existing LASSO regression. While neural nets can be prone to volatility, but the extremity of week-to-week movements could indicate further methodological errors, which cropped up on multiple occasions as documented in the project presentation above. Most notably, the temperature square term was mistakenly excluded from training set, with significant potential impact due the clear quadratic relationship between temperature and load. Volatility could also indicate the need for larger ensembles, which would likely violate the stated need for rapid training. This could be mitigated by relying on a pre-trained model updated less frequently, as can be seen in some of the research papers noted above.

## Next Steps

1. It would likely worth re-running the larger grid searches with corrected methodology and the inclusion of some additional interaction and power terms that had been excluded initially.
1. The requirement to train the model on a weekly basis is largely intended to simplify processes, but creating a pre-trained model could prove preferable assuming improved accuracy.
1. While LSTMs are slower to train, they could be worth revisiting in the event that a pre-trained model proves viable.
1. Some power and interaction terms within the LASSO appear to be implemented incorrectly, which could be limiting its accuracy. Correcting these would potentially raise the baseline required for an RNN to beat.

## Files and Folders

| File/Folder | Description |
|-|-|
| Charts | Data folder that holds all charts produced by the RNN grid search models, as well as supplemental charts created from this data or used from public sources |
| JupyterScripts | Data folder that holds all non-production scripts used for initial RNN analysis and comparison |
| `LoadModel_RNN_7_Validation.ipynb` | Final validation script for initial project, which trains and tests an RNN on 47 consecutive sets of data shifted by week |
| `LoadModel_LASSO_2_Validation.ipynb` | Validation script for original LASSO model, following same format as above |
| `Presentation Charts.ipynb` | Notebook for creating additional charts included above |
| PJM_Weekly_Model | Folder that holds all production scripts from current model, as well as sample data. Scripts are proprietary and thus withheld from repository |
| Outputs | Data folder that holds `.csv` data files with forecasted load values by model, as well as MSE and MAPE scores by model |
| Presentation | Data folder that holds latest presentation `.pptx` file on the projec |
| `PJM_Weekly_Model File Hierarchy.pptx` | Notebook checking database city names against external reference |

## Data Dictionary of `sample_base_data.csv`
| Columns | Type | Description |
|-|-|-|
| `Date` | datetime | date and time of load value |
| `value` | float | PJM RTO-level average hourly load in MW |
| `WWP` | float | winter weather parameter - estimates "real feel" for cold weather by combining temperature and wind speed - roughly comparable to Fahrenheit |
| `THI` | float | temperature humidity index - estimates "real feel" for warm weather by combining temperature and humidity index - roughly comparable to Fahrenheit |
| `Light` | float | light index - average amount of sunlight in given hour - ranges from 0 to 1 |
| `WWPLag` | float | winter weather parameter lagged 24 hours |
| `THILag` | float | temperature humidity index lagged 24 hours |
| `WWPSq` | float | square of winter weather parameter - possible methodological error in original model |
| `THISq` | float | square of temperature humidity index - possible methodological error in original model |
| `WWPLagSq` | float | square of winter weather parameter lagged 24 hours - possible methodological error in original model |
| `THILagSq` | float | square of temperature humidity index lagged 24 hours - possible methodological error in original model |
| `LoadLag` | float | PJM RTO-level average hourly load in MW lagged 168 hours |
| `LoadLagSq` | float | square of PJM RTO-level average hourly load in MW lagged 168 hours |
| `LoadLag3` | float | cube of PJM RTO-level average hourly load in MW lagged 168 hours |
| `Month` | int | categorical representation of month |
| `Day` | int | categorical representation of day of month |
| `WeekDay` | int | categorical representation of day of week |
| `Hour` | int | categorical representation of hour of day |
| `Holiday` | int | dummy variable for whether a day is a NERC holiday |