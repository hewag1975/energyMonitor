Energy monitor
================

## Introduction

Trying to reduce our private consumption of electricity and natural gas
I started to collect weekly readings from our gas and electricity meter.

Data is manually collected via a data sheet (`raw/energymonitor.ods`)
and then visualized using R and `ggplot2`.

## Data preparation

Data is imported from the spreadsheet (here LibreOffice Calc format
`ods`) as a `data.table` object, the latest (and yet empty) rows of the
template are removed and only the columns of interest are selected.

``` r
rdg = readODS::read_ods(
  "data/energymonitor.ods"
  , sheet = "data"
) |> 
  subset(!is.na(rdg_gas)) |> 
  subset(select = c("date", "rdg_gas", "rdg_pow")) |> 
  as.data.table()

rdg[, date := as.POSIXct(date, format = "%m/%d/%Y")]
rdg[, week := strftime(date, format = "%V")]

head(rdg)
```

    ##          date  rdg_gas rdg_pow week
    ## 1: 2022-07-01 19077.48 45337.2   26
    ## 2: 2022-07-08 19085.40 45369.2   27
    ## 3: 2022-07-15 19093.72 45400.5   28
    ## 4: 2022-07-22 19102.17 45431.5   29
    ## 5: 2022-07-29 19110.05 45462.1   30
    ## 6: 2022-08-05 19118.53 45495.3   31

The gas meter records the natural gas consumption continuously in cubic
meters. To convert cubic meters readings to kWh per week one has to

- calculate the difference of successive readings.
- multiply the difference with two factors, the calorific value (German:
  “Brennwert”) and the volume correction factor (German:
  “Zustandszahl”). If these are unknown, a good estimate can be achieved
  by multiplying the cubic meters with 10. In my case, I took both
  factors from the last invoice of my provider.

For electricity, consumption is recorded in kWh, so taking the
difference gives the consumption per week.

``` r
cbm2kwh = 0.9355 * 11.517 

rdg[, use_pow := c(NA_real_, diff(rdg_pow))]
rdg[, use_gas := c(NA_real_, diff(rdg_gas))]
rdg[, use_gas_kwh := use_gas * cbm2kwh]

head(rdg)
```

    ##          date  rdg_gas rdg_pow week use_pow use_gas use_gas_kwh
    ## 1: 2022-07-01 19077.48 45337.2   26      NA      NA          NA
    ## 2: 2022-07-08 19085.40 45369.2   27    32.0   7.916    85.28820
    ## 3: 2022-07-15 19093.72 45400.5   28    31.3   8.319    89.63018
    ## 4: 2022-07-22 19102.17 45431.5   29    31.0   8.451    91.05237
    ## 5: 2022-07-29 19110.05 45462.1   30    30.6   7.885    84.95420
    ## 6: 2022-08-05 19118.53 45495.3   31    33.2   8.481    91.37560
