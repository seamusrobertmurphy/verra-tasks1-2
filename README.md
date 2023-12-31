Technical Exercise for Program Officer Candidate; Verra Technology
Solutions
================
SMurphy
2023-12-04

- [Task 1](#task-1)
- [1.1 Import Data](#11-import-data)
- [1.2 Audit Data](#12-audit-data)
- [1.3 Tidy Data](#13-tidy-data)
- [2.1 Compute Biomass at Plot Level
  (Eq1)](#21-compute-biomass-at-plot-level-eq1)
- [2.2 Compute Biomass at Strata Level
  (Eq2)](#22-compute-biomass-at-strata-level-eq2)
- [2.3 Compute Carbon in Harvested Volumes
  (Eq3)](#23-compute-carbon-in-harvested-volumes-eq3)
- [2.4 Compute Carbon in Extracted Volumes
  (Eq4)](#24-compute-carbon-in-extracted-volumes-eq4)
- [3.1 Additional Metrics](#31-additional-metrics)
- [3.2 Regression Diagnostics](#32-regression-diagnostics)
- [Appendix I: Codebook Derived](#appendix1)
- [Task 2](#task-2)

## Task 1

***Prototype Tool for Quantifying GHG in Standing Timber Using
VSC-Methodology-VM0010***

The following workflow was used to test a prototype tool for quantifying
greenhouse gas (GHG) emissions reduction. The objective is to illustrate
the process outlined in the VCS Methodology, specifically VM0010, with
an emphasis on simplicity of tool design over burdensome complexity. The
chosen subsection for digitalization is “8.1.1 Calculation of carbon
stocks in commercial timber volumes”.

A number of empricial assumptions were made including sourcing of
pre-existing inventory dataset with pre-defined format and values
published in the exercise’s document (‘Program Officer Technology
Solutions Seamus Murphy’). Using this table of observations, an excel
spreadsheet was copied into and imported into an R environment as the
dataframe ’`dataset_raw'` and commited to the project’s github
repository [here](https://github.com/seamusrobertmurphy/verra-tasks1-2).
In this original, unchanged dataset, assumptions of values can be viewed
regarding sample strata, plot, species, tree, volume, species wood
characteristics, and plot area.

The following table presents a data dictionary of these values, along
with descriptions of their units, variable labels, types and file
structure. Documentation of the dataset as it was received in original
format ‘`dataset_raw.xlsx`’ and descriptions of output dataset
’`dataset_tidy.xlsx'` are intended to enable reproducability, encourage
collaboration and inform future procedures for data submissions from
clients.

<table style="width:99%;">
<colgroup>
<col style="width: 19%" />
<col style="width: 14%" />
<col style="width: 43%" />
<col style="width: 11%" />
<col style="width: 9%" />
</colgroup>
<tbody>
<tr class="odd">
<td><h4
id="label-original--transformed"><sub><strong>Label</strong> (original =&gt; transformed)</sub></h4></td>
<td><h4
id="file-location"><sub><strong>File location</strong></sub></h4></td>
<td><h4
id="variable-description"><sub><strong>Variable description</strong></sub></h4></td>
<td><h4
id="units-values"><sub><strong>Units, values</strong></sub></h4></td>
<td><h4 id="type"><sub><strong>Type</strong></sub></h4></td>
</tr>
<tr class="even">
<td><sub>“Volume.V.l.j.I.sp”=&gt; “volume”</sub></td>
<td><sub>‘dataset_raw’, ‘dataset_tidy’</sub></td>
<td><sub>Whole stem volume of living tree</sub></td>
<td><sub>m3</sub></td>
<td><sub>numeric</sub></td>
</tr>
<tr class="odd">
<td><sub>“Species..j.” =&gt; “species_j”</sub></td>
<td><sub>‘dataset_raw’, ‘dataset_tidy’</sub></td>
<td><sub>Tree species used at the species summary level.</sub></td>
<td><sub>Sp1–Sp5</sub></td>
<td><sub>character</sub></td>
</tr>
<tr class="even">
<td><sub>“Stratum...i.” =&gt; ’stratum_i”</sub></td>
<td><sub>‘dataset_raw’, ‘dataset_tidy’</sub></td>
<td><sub>Free form text field identifying stratum of each plot.</sub></td>
<td><sub>1, 2</sub></td>
<td><sub>integer &gt; Factor</sub></td>
</tr>
<tr class="odd">
<td><sub>“Plot..sp.” =&gt; “plot_sp”</sub></td>
<td><sub>‘dataset_raw’, ‘dataset_tidy’</sub></td>
<td><sub>Numerical identifier unique only within its stratum.</sub></td>
<td><sub>1, 2, 3</sub></td>
<td><sub>integer</sub></td>
</tr>
<tr class="even">
<td><sub>“Tree..l.” =&gt; “tree_l”</sub></td>
<td><sub>‘dataset_raw’, ‘dataset_tidy’</sub></td>
<td><sub>Text field identifying tree observation unique only specific plot family of associated stratum.</sub></td>
<td><sub>t1 - t5</sub></td>
<td><sub>character&gt;numeric</sub></td>
</tr>
<tr class="odd">
<td><sub>“bcef_r”</sub></td>
<td><sub>‘dataset_raw’, ‘dataset_tidy’</sub></td>
<td><sub>Biomass conversion and expansion factor used to derive carbon stocks from timber volume</sub></td>
<td><sub>%</sub></td>
<td><sub>numeric</sub></td>
</tr>
<tr class="even">
<td><sub>“cf”</sub></td>
<td><sub>‘dataset_raw’, ‘dataset_tidy’</sub></td>
<td><sub>Carbon factor used to derive carbon stocks from volumes of extracted  timber</sub></td>
<td><sub>%</sub></td>
<td><sub>numeric</sub></td>
</tr>
<tr class="odd">
<td><sub>“d”</sub></td>
<td><sub>‘dataset_tidy’,</sub></td>
<td><sub>Basic wood density represented as fraction of dry in tons to green volume</sub></td>
<td><sub>%</sub></td>
<td><sub>numeric</sub></td>
</tr>
<tr class="even">
<td><sub>“a_sp”</sub></td>
<td><sub>dataset_tidy</sub></td>
<td><sub>Area in hectares of individual sample plots</sub></td>
<td><sub>ha</sub></td>
<td><sub>numeric</sub></td>
</tr>
<tr class="odd">
<td><sub>“vji_sp_m3”</sub></td>
<td><sub>dataset_tidy</sub></td>
<td><sub>Sum volume of merchantable timber of a species from specific plots within specific stratum</sub></td>
<td><sub>m3</sub></td>
<td><sub>numeric</sub></td>
</tr>
<tr class="even">
<td><sub>“vji_ha_m3”</sub></td>
<td><sub>dataset_tidy</sub></td>
<td><sub>Mean volume per hectare of merchantable timber of a species in a specific stratum.</sub></td>
<td><sub>m3.ha-1</sub></td>
<td><sub>numeric</sub></td>
</tr>
<tr class="odd">
<td><sub>chb_ha_tC</sub></td>
<td><sub>dataset_tidy</sub></td>
<td><sub>Mean volume of carbon in extracted timber of species in stratum using ‘bcef_r’ and ‘cf’ factors.</sub></td>
<td><sub>tC.ha-1</sub></td>
<td><sub>numeric</sub></td>
</tr>
<tr class="even">
<td><sub>cex_ha_tC</sub></td>
<td><sub>dataset_tidy</sub></td>
<td><sub>Mean volume of carbon in extracted biomass of a species in a stratum using ‘d’ and ‘cf’ factors.</sub></td>
<td><sub>tC.ha-1</sub></td>
<td><sub>numeric</sub></td>
</tr>
</tbody>
</table>

## 1.1 Import Data

Import `dataset_raw` and write copy to `dataset_tidy`. Seed is set to
`77777`.

``` r
set.seed(77777)
dataset_raw <- read_excel("dataset_raw.xlsx")
write.csv(dataset_raw, "dataset_tidy.csv", row.names = FALSE)
dataset_tidy <- read.csv("dataset_tidy.csv")
dataset_tidy
```

    # A tibble: 10 × 5
       Stratum...i. Plot..sp. Species..j. Tree..l. Volume..V_.l.j.I.sp..
              <int>     <int> <chr>       <chr>                    <dbl>
     1            1         1 Sp1         t1                        3.3 
     2            1         1 Sp1         t2                        4.8 
     3            1         1 Sp1         t3                        4.08
     4            1         2 Sp4         t1                        1.5 
     5            1         2 Sp4         t2                        1.68
     6            2         1 Sp1         t1                        1.38
     7            2         1 Sp2         t2                        3.24
     8            2         1 Sp3         t3                        3.72
     9            2         1 sp4         t4                        2.94
    10            2         1 Sp5         t5                        3.36

## 1.2 Audit Data

Examine structure, scan for errors, and save audit report for input to
codebook ([Appendix I](#appendix1)).

``` r
str(dataset_tidy)
dplyr::count(dataset_tidy, Species..j.)

saveHTML(dataMaid::makeDataReport(
  dataset_tidy,
  output = "html",
  codebook = TRUE,
  onlyProblematic = TRUE,
  visuals = setVisuals(all = "basicVisual"),
  replace = TRUE
))
```

    'data.frame':   10 obs. of  5 variables:
     $ Stratum...i.         : int  1 1 1 1 1 2 2 2 2 2
     $ Plot..sp.            : int  1 1 1 2 2 1 1 1 1 1
     $ Species..j.          : chr  "Sp1" "Sp1" "Sp1" "Sp4" ...
     $ Tree..l.             : chr  "t1" "t2" "t3" "t1" ...
     $ Volume..V_.l.j.I.sp..: num  3.3 4.8 4.08 1.5 1.68 1.38 3.24 3.72 2.94 3.36

    # A tibble: 6 × 2
      Species..j.     n
      <chr>       <int>
    1 Sp1             4
    2 Sp2             1
    3 Sp3             1
    4 Sp4             2
    5 Sp5             1
    6 sp4             1

<img src="dataMaid_dataset_tidy.png" width="816" />

## 1.3 Tidy Data

Tests identified one problematic entry in values of `Species..j.`
variable and multiple naming issues inherited during data import.
Variable relabelling was carried out according to naming convention
stated in exercise document, the symbology used in equations 1-4 of
subsection 8.1.1 of the [VM0010 Verra
Methodology](https://verra.org/wp-content/uploads/2018/03/VM0010-Methodology-for-IMF-LtPF-v1.3_0.pdf),
and syntax standards provided in the [Tidyverse style
guide](https://style.tidyverse.org/). This involved the following
changes.

- Correct the case-sensitive error from `sp4` to `Sp4`.
- Convert or recode variables to reflec their class
- Remove spaces
- Use underscores between words
- Change to lowercase
- Warnings, messages turned to silent
- Space placed after commas
- Max 80 characters to chunks
- Comments limited to single line each
- `::` operator used to signpost installs & avoid package conflicts

``` r
data.table::setnames(dataset_tidy, old = "Stratum...i.", new = "stratum_i", skip_absent = TRUE)
data.table::setnames(dataset_tidy, old = "Species..j.", new = "species_j", skip_absent = TRUE)
data.table::setnames(dataset_tidy, old = "Plot..sp.", new = "plot_sp", skip_absent = TRUE)
data.table::setnames(dataset_tidy, old = "Tree..l.", new = "tree_l", skip_absent = TRUE)
data.table::setnames(dataset_tidy, old = "Volume..V_.l.j.I.sp..", new = "volume", skip_absent = TRUE)
dataset_tidy$species_j[dataset_tidy$species_j == "sp4"] <- "Sp4"
dataset_tidy$species_j <- as.factor(dataset_tidy$species_j)
dataset_tidy$stratum_i <- as.factor(dataset_tidy$stratum_i)
```

<br>

Derive new variable columns using values provided in the exercise
document. Compute additional variables of area estimates at different
units and scales:

- `bcef_r`: Biomass expansion factor applicable to wood removals (t.d.m
  m<sup>-3</sup>).
- `cf`: Carbon fraction of biomass of species (tCt d.m.<sup>-1</sup>)
- `d`: Basic wood density of species (t d.m. m<sup>-3</sup>)
- `a_sp`: Total area of plot in hectares (ha)
- `a_sp_m2`: Total area of plot in metres squared (m<sup>2</sup>)
- `a_I_m2`: Total area of stratum in metres squared (m<sup>2</sup>)
- `a_I_ha`: Total area of stratum in hectares (ha)

Tabulate, confirm and write to directory.

``` r
dataset_tidy$bcef_r <- 0.7
dataset_tidy$cf <- 0.5
dataset_tidy$d <- 0.5
dataset_tidy$a_sp <- 0.1
dataset_tidy$a_sp_m2 <- dataset_tidy$a_sp * 10000

dataset_tidy <- dataset_tidy %>%
  group_by(stratum_i) %>%
  mutate(a_I_m2 = sum(a_sp_m2), a_I_ha = sum(a_sp)) 

dataset_tidy
```

    # A tibble: 10 × 12
       stratum_i plot_sp species_j tree_l volume bcef_r    cf     d  a_sp a_sp_m2
       <fct>       <int> <fct>     <chr>   <dbl>  <dbl> <dbl> <dbl> <dbl>   <dbl>
     1 1               1 Sp1       t1       3.3     0.7   0.5   0.5   0.1    1000
     2 1               1 Sp1       t2       4.8     0.7   0.5   0.5   0.1    1000
     3 1               1 Sp1       t3       4.08    0.7   0.5   0.5   0.1    1000
     4 1               2 Sp4       t1       1.5     0.7   0.5   0.5   0.1    1000
     5 1               2 Sp4       t2       1.68    0.7   0.5   0.5   0.1    1000
     6 2               1 Sp1       t1       1.38    0.7   0.5   0.5   0.1    1000
     7 2               1 Sp2       t2       3.24    0.7   0.5   0.5   0.1    1000
     8 2               1 Sp3       t3       3.72    0.7   0.5   0.5   0.1    1000
     9 2               1 Sp4       t4       2.94    0.7   0.5   0.5   0.1    1000
    10 2               1 Sp5       t5       3.36    0.7   0.5   0.5   0.1    1000
    # ℹ 2 more variables: a_I_m2 <dbl>, a_I_ha <dbl>

``` r
write.csv(dataset_tidy, "dataset_tidy.csv", row.names = FALSE)
```

<br>

## 2.1 Compute Biomass at Plot Level (Eq1)

Note all four equations that follow derive estimates according to
specific species, whether measured at the plot level, stratum or
globally. Hence, caution is advised when computing initial, absolute
estimates and when deriving or applying hectare expansion factors.

On page 19 of the VM0010 Methodology, ‘equation 1’ provides calculation
of merchantable timber volume of a species at the plot level. This is
estimated in m<sup>3</sup> as follows:

<br>

$V_{j,i|BSL} = \sum_{l = 1}^{L} V_{l,j,i,sp}$

<br>

where $V_{j,i|BSL}$ refers to the <u>***sum***</u> of merchantable
volume of species *j* from plot *sp* in stratum *i.* For this, we run
the following:

``` r
dataset_tidy <- dataset_tidy %>%
  group_by(species_j, stratum_i, plot_sp) %>%
  mutate(vji_sp_m3 = sum(volume))

data.table::setDT(dataset_tidy)[, .(
  vji_sp_m3 = sum(volume)
),
by = .(stratum_i, plot_sp, species_j)
]
```

    # A tibble: 7 × 4
      stratum_i plot_sp species_j vji_sp_m3
      <fct>       <int> <fct>         <dbl>
    1 1               1 Sp1           12.2 
    2 1               2 Sp4            3.18
    3 2               1 Sp1            1.38
    4 2               1 Sp2            3.24
    5 2               1 Sp3            3.72
    6 2               1 Sp4            2.94
    7 2               1 Sp5            3.36

<br>

## 2.2 Compute Biomass at Strata Level (Eq2)

On page 19 of the VM0010 Methodology, ‘equation 2’ provides calculation
of merchantable timber volume of a species at the stratum level:

<br>

$V_{j,i|BSL} = \displaystyle \frac{1}{SP} \sum_{sp=1}^{sp} \displaystyle \frac{V_{j,i,sp}}{A_{sp}}$

<br>

where $V_{j,i|BSL}$ refers to the <u>***mean***</u> merchantable timber
volume of species *j* measured across all plots within stratum *i*.
Volume estimates are derived in terms of m<sup>3</sup>/ha<sup>-1</sup>
using an expansion factor derived as the ratio of plot area
1,000m<sup>2</sup> to 10,000m<sup>2</sup>, as follows.

``` r
dataset_tidy <- dataset_tidy %>%
  group_by(stratum_i, species_j) %>%
  mutate(vji_ha_m3 = mean(vji_sp_m3) * 10)

data.table::setDT(dataset_tidy)[, .(
  vji_sp_m3,
  vji_ha_m3 = mean(vji_sp_m3) * 10
),
by = .(stratum_i, species_j)
]
```

    # A tibble: 10 × 4
       stratum_i species_j vji_sp_m3 vji_ha_m3
       <fct>     <fct>         <dbl>     <dbl>
     1 1         Sp1           12.2      122. 
     2 1         Sp1           12.2      122. 
     3 1         Sp1           12.2      122. 
     4 1         Sp4            3.18      31.8
     5 1         Sp4            3.18      31.8
     6 2         Sp1            1.38      13.8
     7 2         Sp2            3.24      32.4
     8 2         Sp3            3.72      37.2
     9 2         Sp4            2.94      29.4
    10 2         Sp5            3.36      33.6

<br>

## 2.3 Compute Carbon in Harvested Volumes (Eq3)

On page 20 of the VM0010 Methodology, ‘equation 3’ provides calculation
of carbon stock of harvested timber volumes for species *i* in stratum
*i*.

In the absence of harvest plan data, carbon stocks of commercial timber
will be derived from the full inventory `dataset_tidy`.

This differs to ‘equation 3’ below, which measures mean carbon stock
from volumes of ‘harvested biomass’ $C_{HB,j,i|BSL}$ comprised of
volumes estimates from extracted timber that is destined off-site for
market sales, in addition to volume estimates from extracted timber that
is remaining on-site due to damage or use during site operations and
road construction. The difference in destinations of these carbon stocks
means that alternative carbon factors must be applied in their
calculation, as indicated in following two equations

<br>

$C_{HB,j,i|BSL} = (V_{EX,j,i|BSL} + V_{EX,INF,j,i|BSL} ) * BCEF_{R} * CF_{j}$

<br>

Carbon estimates are calculated in units of metric tons per hectare
(tC/ha<sup>-1</sup>) by multiplying mean volume per hectare of extracted
timber of species *i* from stratum *j* by the carbon fraction $CF$ and
biomass conversion and expansion factor $BCEF_{R}$ of that species.

``` r
dataset_tidy <- dataset_tidy %>%
  mutate(chb_ha_tC = vji_ha_m3 * bcef_r * cf)

data.table::setDT(dataset_tidy)[, .(
  vji_sp_m3,
  vji_ha_m3,
  chb_ha_tC = vji_ha_m3 * bcef_r * cf
),
by = .(stratum_i, species_j)
]
```

    # A tibble: 10 × 5
       stratum_i species_j vji_sp_m3 vji_ha_m3 chb_ha_tC
       <fct>     <fct>         <dbl>     <dbl>     <dbl>
     1 1         Sp1           12.2      122.      42.6 
     2 1         Sp1           12.2      122.      42.6 
     3 1         Sp1           12.2      122.      42.6 
     4 1         Sp4            3.18      31.8     11.1 
     5 1         Sp4            3.18      31.8     11.1 
     6 2         Sp1            1.38      13.8      4.83
     7 2         Sp2            3.24      32.4     11.3 
     8 2         Sp3            3.72      37.2     13.0 
     9 2         Sp4            2.94      29.4     10.3 
    10 2         Sp5            3.36      33.6     11.8 

<br>

## 2.4 Compute Carbon in Extracted Volumes (Eq4)

On page 21 of the VM0010 Methodology, ‘equation 4’ provides calculation
of mean estimates of carbon stocks of species *i* in stratum *j,* by
multiplying the species’ value of basic wood density $D$ by its carbon
fraction $CF$ by its mean volume of extracted biomass per hectare.
Estimates of carbon stocks are derived in terms of metric tons per
hectare tC.ha<sup>-1</sup> using the following formula:

<br>

$C_{EX,j,i|BSL} = (V_{EX,j,i|BSL} + V_{EX,INF,j,i|BSL} ) * D_{j} * CF_{j}$

<br>

``` r
dataset_tidy <- dataset_tidy %>%
  mutate(cex_ha_tC = vji_ha_m3 * d * cf)

data.table::setDT(dataset_tidy)[, .(
  vji_sp_m3,
  vji_ha_m3,
  chb_ha_tC,
  cex_ha_tC = vji_ha_m3 * d * cf
),
by = .(stratum_i, species_j)
]
```

    # A tibble: 10 × 6
       stratum_i species_j vji_sp_m3 vji_ha_m3 chb_ha_tC cex_ha_tC
       <fct>     <fct>         <dbl>     <dbl>     <dbl>     <dbl>
     1 1         Sp1           12.2      122.      42.6      30.4 
     2 1         Sp1           12.2      122.      42.6      30.4 
     3 1         Sp1           12.2      122.      42.6      30.4 
     4 1         Sp4            3.18      31.8     11.1       7.95
     5 1         Sp4            3.18      31.8     11.1       7.95
     6 2         Sp1            1.38      13.8      4.83      3.45
     7 2         Sp2            3.24      32.4     11.3       8.1 
     8 2         Sp3            3.72      37.2     13.0       9.3 
     9 2         Sp4            2.94      29.4     10.3       7.35
    10 2         Sp5            3.36      33.6     11.8       8.4 

## 3.1 Additional Metrics

``` r
# vJ_ha_m3: Mean volume by species across strata (m^3.ha-1)
dataset_tidy <- dataset_tidy %>%
  group_by(species_j) %>%
  mutate(vJ_ha_m3 = mean(vji_sp_m3) * 10)
dataset_tidy[1:10, ] %>%
  select(
    stratum_i, species_j,
    plot_sp, vji_ha_m3,
    chb_ha_tC, cex_ha_tC,
    vJ_ha_m3
  ) %>%
  kbl(caption = "Table with right-hand showing mean biomass volume per species across all strata (m3.ha-1)", escape = T) %>%
  kable_minimal(full_width = T)
```

<table class=" lightable-minimal" style="font-family: &quot;Trebuchet MS&quot;, verdana, sans-serif; margin-left: auto; margin-right: auto;">
<caption>
Table with right-hand showing mean biomass volume per species across all
strata (m3.ha-1)
</caption>
<thead>
<tr>
<th style="text-align:left;">
stratum_i
</th>
<th style="text-align:left;">
species_j
</th>
<th style="text-align:right;">
plot_sp
</th>
<th style="text-align:right;">
vji_ha_m3
</th>
<th style="text-align:right;">
chb_ha_tC
</th>
<th style="text-align:right;">
cex_ha_tC
</th>
<th style="text-align:right;">
vJ_ha_m3
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
Sp1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
121.8
</td>
<td style="text-align:right;">
42.63
</td>
<td style="text-align:right;">
30.45
</td>
<td style="text-align:right;">
94.8
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
Sp1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
121.8
</td>
<td style="text-align:right;">
42.63
</td>
<td style="text-align:right;">
30.45
</td>
<td style="text-align:right;">
94.8
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
Sp1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
121.8
</td>
<td style="text-align:right;">
42.63
</td>
<td style="text-align:right;">
30.45
</td>
<td style="text-align:right;">
94.8
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
Sp4
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
31.8
</td>
<td style="text-align:right;">
11.13
</td>
<td style="text-align:right;">
7.95
</td>
<td style="text-align:right;">
31.0
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
Sp4
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
31.8
</td>
<td style="text-align:right;">
11.13
</td>
<td style="text-align:right;">
7.95
</td>
<td style="text-align:right;">
31.0
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:left;">
Sp1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
13.8
</td>
<td style="text-align:right;">
4.83
</td>
<td style="text-align:right;">
3.45
</td>
<td style="text-align:right;">
94.8
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:left;">
Sp2
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
32.4
</td>
<td style="text-align:right;">
11.34
</td>
<td style="text-align:right;">
8.10
</td>
<td style="text-align:right;">
32.4
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:left;">
Sp3
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
37.2
</td>
<td style="text-align:right;">
13.02
</td>
<td style="text-align:right;">
9.30
</td>
<td style="text-align:right;">
37.2
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:left;">
Sp4
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
29.4
</td>
<td style="text-align:right;">
10.29
</td>
<td style="text-align:right;">
7.35
</td>
<td style="text-align:right;">
31.0
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:left;">
Sp5
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
33.6
</td>
<td style="text-align:right;">
11.76
</td>
<td style="text-align:right;">
8.40
</td>
<td style="text-align:right;">
33.6
</td>
</tr>
</tbody>
</table>

``` r
# VI_ha_m3: Mean volume by strata across species and plots (m^3.ha-1)
dataset_tidy <- dataset_tidy %>%
  group_by(stratum_i) %>%
  mutate(vI_ha_m3 = mean(vji_sp_m3) * 10)
dataset_tidy[1:10, ] %>%
  select(
    stratum_i, species_j,
    plot_sp, vji_ha_m3,
    chb_ha_tC, cex_ha_tC,
    vJ_ha_m3, vI_ha_m3
  ) %>%
  kbl(caption = "Table with right-hand column showing mean biomass volume per strata across all species and plots (m3.ha-1)", escape = T) %>%
  kable_minimal(full_width = T)
```

<table class=" lightable-minimal" style="font-family: &quot;Trebuchet MS&quot;, verdana, sans-serif; margin-left: auto; margin-right: auto;">
<caption>
Table with right-hand column showing mean biomass volume per strata
across all species and plots (m3.ha-1)
</caption>
<thead>
<tr>
<th style="text-align:left;">
stratum_i
</th>
<th style="text-align:left;">
species_j
</th>
<th style="text-align:right;">
plot_sp
</th>
<th style="text-align:right;">
vji_ha_m3
</th>
<th style="text-align:right;">
chb_ha_tC
</th>
<th style="text-align:right;">
cex_ha_tC
</th>
<th style="text-align:right;">
vJ_ha_m3
</th>
<th style="text-align:right;">
vI_ha_m3
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
Sp1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
121.8
</td>
<td style="text-align:right;">
42.63
</td>
<td style="text-align:right;">
30.45
</td>
<td style="text-align:right;">
94.8
</td>
<td style="text-align:right;">
85.80
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
Sp1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
121.8
</td>
<td style="text-align:right;">
42.63
</td>
<td style="text-align:right;">
30.45
</td>
<td style="text-align:right;">
94.8
</td>
<td style="text-align:right;">
85.80
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
Sp1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
121.8
</td>
<td style="text-align:right;">
42.63
</td>
<td style="text-align:right;">
30.45
</td>
<td style="text-align:right;">
94.8
</td>
<td style="text-align:right;">
85.80
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
Sp4
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
31.8
</td>
<td style="text-align:right;">
11.13
</td>
<td style="text-align:right;">
7.95
</td>
<td style="text-align:right;">
31.0
</td>
<td style="text-align:right;">
85.80
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:left;">
Sp4
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
31.8
</td>
<td style="text-align:right;">
11.13
</td>
<td style="text-align:right;">
7.95
</td>
<td style="text-align:right;">
31.0
</td>
<td style="text-align:right;">
85.80
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:left;">
Sp1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
13.8
</td>
<td style="text-align:right;">
4.83
</td>
<td style="text-align:right;">
3.45
</td>
<td style="text-align:right;">
94.8
</td>
<td style="text-align:right;">
29.28
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:left;">
Sp2
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
32.4
</td>
<td style="text-align:right;">
11.34
</td>
<td style="text-align:right;">
8.10
</td>
<td style="text-align:right;">
32.4
</td>
<td style="text-align:right;">
29.28
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:left;">
Sp3
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
37.2
</td>
<td style="text-align:right;">
13.02
</td>
<td style="text-align:right;">
9.30
</td>
<td style="text-align:right;">
37.2
</td>
<td style="text-align:right;">
29.28
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:left;">
Sp4
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
29.4
</td>
<td style="text-align:right;">
10.29
</td>
<td style="text-align:right;">
7.35
</td>
<td style="text-align:right;">
31.0
</td>
<td style="text-align:right;">
29.28
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:left;">
Sp5
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
33.6
</td>
<td style="text-align:right;">
11.76
</td>
<td style="text-align:right;">
8.40
</td>
<td style="text-align:right;">
33.6
</td>
<td style="text-align:right;">
29.28
</td>
</tr>
</tbody>
</table>

``` r
# Vsp_ha_n3: Total volume per plot (m^3.ha-1)
dataset_tidy <- dataset_tidy %>%
  group_by(plot_sp, stratum_i) %>%
  mutate(Vsp_ha_m3 = sum(volume) * 10)
dataset_tidy[1:10, ] %>%
  select(
    stratum_i,
    plot_sp, vji_ha_m3,
    chb_ha_tC, cex_ha_tC,
    vJ_ha_m3, vI_ha_m3,
    vI_ha_m3, Vsp_ha_m3
  ) %>%
  kbl(caption = "Table with right-hand column showing total biomass per plot across species (m3.ha-1)", escape = T) %>%
  kable_minimal(full_width = T)
```

<table class=" lightable-minimal" style="font-family: &quot;Trebuchet MS&quot;, verdana, sans-serif; margin-left: auto; margin-right: auto;">
<caption>
Table with right-hand column showing total biomass per plot across
species (m3.ha-1)
</caption>
<thead>
<tr>
<th style="text-align:left;">
stratum_i
</th>
<th style="text-align:right;">
plot_sp
</th>
<th style="text-align:right;">
vji_ha_m3
</th>
<th style="text-align:right;">
chb_ha_tC
</th>
<th style="text-align:right;">
cex_ha_tC
</th>
<th style="text-align:right;">
vJ_ha_m3
</th>
<th style="text-align:right;">
vI_ha_m3
</th>
<th style="text-align:right;">
Vsp_ha_m3
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
121.8
</td>
<td style="text-align:right;">
42.63
</td>
<td style="text-align:right;">
30.45
</td>
<td style="text-align:right;">
94.8
</td>
<td style="text-align:right;">
85.80
</td>
<td style="text-align:right;">
121.8
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
121.8
</td>
<td style="text-align:right;">
42.63
</td>
<td style="text-align:right;">
30.45
</td>
<td style="text-align:right;">
94.8
</td>
<td style="text-align:right;">
85.80
</td>
<td style="text-align:right;">
121.8
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
121.8
</td>
<td style="text-align:right;">
42.63
</td>
<td style="text-align:right;">
30.45
</td>
<td style="text-align:right;">
94.8
</td>
<td style="text-align:right;">
85.80
</td>
<td style="text-align:right;">
121.8
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
31.8
</td>
<td style="text-align:right;">
11.13
</td>
<td style="text-align:right;">
7.95
</td>
<td style="text-align:right;">
31.0
</td>
<td style="text-align:right;">
85.80
</td>
<td style="text-align:right;">
31.8
</td>
</tr>
<tr>
<td style="text-align:left;">
1
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
31.8
</td>
<td style="text-align:right;">
11.13
</td>
<td style="text-align:right;">
7.95
</td>
<td style="text-align:right;">
31.0
</td>
<td style="text-align:right;">
85.80
</td>
<td style="text-align:right;">
31.8
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
13.8
</td>
<td style="text-align:right;">
4.83
</td>
<td style="text-align:right;">
3.45
</td>
<td style="text-align:right;">
94.8
</td>
<td style="text-align:right;">
29.28
</td>
<td style="text-align:right;">
146.4
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
32.4
</td>
<td style="text-align:right;">
11.34
</td>
<td style="text-align:right;">
8.10
</td>
<td style="text-align:right;">
32.4
</td>
<td style="text-align:right;">
29.28
</td>
<td style="text-align:right;">
146.4
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
37.2
</td>
<td style="text-align:right;">
13.02
</td>
<td style="text-align:right;">
9.30
</td>
<td style="text-align:right;">
37.2
</td>
<td style="text-align:right;">
29.28
</td>
<td style="text-align:right;">
146.4
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
29.4
</td>
<td style="text-align:right;">
10.29
</td>
<td style="text-align:right;">
7.35
</td>
<td style="text-align:right;">
31.0
</td>
<td style="text-align:right;">
29.28
</td>
<td style="text-align:right;">
146.4
</td>
</tr>
<tr>
<td style="text-align:left;">
2
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
33.6
</td>
<td style="text-align:right;">
11.76
</td>
<td style="text-align:right;">
8.40
</td>
<td style="text-align:right;">
33.6
</td>
<td style="text-align:right;">
29.28
</td>
<td style="text-align:right;">
146.4
</td>
</tr>
</tbody>
</table>

``` r
# Save output dataset
write.csv(dataset_tidy, "dataset_tidy.csv", row.names = FALSE)
```

## 3.2 Regression Diagnostics

<u>*Section unfinished!:*</u> For demonstration purposes only, an
ordinary-least-squared (OLS) model was fitted with unscreened covariates
from the above dataset and deployed to generate predictions of total
standing timber over a site area of 100 hectares. From these
predictions, derived residuals were acquired to showcase a number of
diagnostic tools and accuracy metrics relevant to forest inventory
analyses.

Candidate models were then trained, tuned, and analysed using the
10k-fold cross-validation technique (10-repeat). This method was used to
present proposed estimates of model bias between training sample and
test sample for potential use in actual biomass estimation. It was also
used to explore levels of precision in terms of the variation of results
(RMSE). Using DescTools package in R, Theil’s U estimate of error also
examined level of unexplained variance in each model.

This stage was specifically focused around a diagnostic exercise,
reporting on techniques used in cross validation, tuning functions, and
predictor evaluation. For quicker review, sample results from the
following code chunks were gathered and presented in the three tables
directly below.

|              |          |                    |                |              |                  |
|--------------|----------|--------------------|----------------|--------------|------------------|
| **Variable** | **Mean** | **Std. Deviation** | **Std. Error** | **Skewness** | **Shapiro-Wilk** |
| stratum_i    | 1.5      | 0.53               | 0.17           | 0.00         | \*\*\*           |
| species_j    | 2.60     | 1.58               | 0.50           | 0.14         | \*               |
| volume       | 3.00     | 1.15               | 0.36           | 0.36         |                  |
| vji_ha_m3    | 57.54    | 44.77              | 14.16          | 0.69         | \*\*\*           |

|                    |                   |                |                 |                 |                |
|--------------------|-------------------|----------------|-----------------|-----------------|----------------|
|                    | **Breusch Pagan** | **Bonferroni** | **stratum_VIF** | **species_VIF** | **volume_VIF** |
| M1<sup>*OLS*</sup> | 1.078             | 1.212          | 1.078           | 1.212           | 1.131          |

| Model                  | Theil U | MAE<sup>*full.model*</sup> | RMSE<sup>*full.model*</sup> | MAE<sup>*cv.model*</sup> | RMSE<sup>*cv.model*</sup> |
|------------------------|---------|----------------------------|-----------------------------|--------------------------|---------------------------|
| SVM<sup>*linear*</sup> | 0.083   | 9.324                      | 11.575                      | 53.476                   | 65.683                    |
| SVM<sup>*radial*</sup> |         |                            |                             |                          |                           |
| RF<sup>*linear*</sup>  |         |                            |                             |                          |                           |

An additional set of packages used in this stage were loaded as listed
in the `requirements` variable below.

``` r
requirements <- install.packages(
  c(
    "oslrr", "ggplot2", "autoplotly", "BiocManager", "ggbio",
    "car", "psych", "useful", "caret",
    "BIOMASS", "rvest", "lmfor", "DescTools",
    "MLmetrics", "knitr", "phyloseq", "carData"
  ),
  dependencies = TRUE
)
requirements
```

``` r
# str(dataset_tidy)
dataset_tidy$stratum_i <- as.integer(dataset_tidy$stratum_i)
dataset_tidy$species_j <- as.integer(dataset_tidy$species_j)
model_lm1 <- lm(vji_ha_m3 ~ volume + species_j + stratum_i, data = dataset_tidy)
olsrr::ols_regress(vji_ha_m3 ~ volume + species_j + stratum_i, data = dataset_tidy)
```

                             Model Summary                          
    ---------------------------------------------------------------
    R                       0.968       RMSE                13.706 
    R-Squared               0.938       Coef. Var           23.820 
    Adj. R-Squared          0.906       MSE                187.853 
    Pred R-Squared          0.848       MAE                  9.281 
    ---------------------------------------------------------------
     RMSE: Root Mean Square Error 
     MSE: Mean Square Error 
     MAE: Mean Absolute Error 

                                   ANOVA                                 
    --------------------------------------------------------------------
                     Sum of                                             
                    Squares       DF    Mean Square      F         Sig. 
    --------------------------------------------------------------------
    Regression    16909.928        3       5636.643    30.006     5e-04 
    Residual       1127.116        6        187.853                     
    Total         18037.044        9                                    
    --------------------------------------------------------------------

                                        Parameter Estimates                                     
    -------------------------------------------------------------------------------------------
          model       Beta    Std. Error    Std. Beta      t        Sig       lower      upper 
    -------------------------------------------------------------------------------------------
    (Intercept)     84.003        20.840                  4.031    0.007     33.008    134.997 
         volume     21.765         4.239        0.557     5.135    0.002     11.393     32.138 
      species_j     -8.343         3.189       -0.294    -2.616    0.040    -16.145     -0.540 
      stratum_i    -46.712         8.999       -0.550    -5.191    0.002    -68.731    -24.692 
    -------------------------------------------------------------------------------------------

``` r
# psych::describe(dataset_tidy$vji_ha_m3)
# psych::describe(dataset_tidy$volume)
# psych::describe(dataset_tidy$species_j)
# psych::describe(dataset_tidy$stratum_i)

shapiro.test(dataset_tidy$stratum_i)
```


        Shapiro-Wilk normality test

    data:  dataset_tidy$stratum_i
    W = 0.65527, p-value = 0.000254

``` r
shapiro.test(dataset_tidy$species_j)
```


        Shapiro-Wilk normality test

    data:  dataset_tidy$species_j
    W = 0.83827, p-value = 0.04207

``` r
shapiro.test(dataset_tidy$volume)
```


        Shapiro-Wilk normality test

    data:  dataset_tidy$volume
    W = 0.9223, p-value = 0.3766

``` r
shapiro.test(dataset_tidy$vji_ha_m3)
```


        Shapiro-Wilk normality test

    data:  dataset_tidy$vji_ha_m3
    W = 0.69994, p-value = 0.0008795

|              |          |        |        |              |                  |
|--------------|----------|--------|--------|--------------|------------------|
| **Variable** | **Mean** | **SD** | **SE** | **Skewness** | **Shapiro-Wilk** |
| stratum_i    | 1.5      | 0.53   | 0.17   | 0.00         | \*\*\*           |
| species_j    | 2.60     | 1.58   | 0.50   | 0.14         | \*               |
| volume       | 3.00     | 1.15   | 0.36   | 0.36         |                  |
| vji_ha_m3    | 57.54    | 44.77  | 14.16  | 0.69         | \*\*\*           |

    -----------------------------------------------
           Test             Statistic       pvalue  
    -----------------------------------------------
    Shapiro-Wilk              0.918          0.3407 
    Kolmogorov-Smirnov        0.1812         0.8419 
    Cramer-von Mises          0.8333         0.0048 
    Anderson-Darling          0.3383         0.4225 
    -----------------------------------------------


     Breusch Pagan Test for Heteroskedasticity
     -----------------------------------------
     Ho: the variance is constant            
     Ha: the variance is not constant        

                    Data                  
     -------------------------------------
     Response : vji_ha_m3 
     Variables: fitted values of vji_ha_m3 

            Test Summary         
     ----------------------------
     DF            =    1 
     Chi2          =    0.7880034 
     Prob > Chi2   =    0.3747045 

    Tolerance and Variance Inflation Factor
    ---------------------------------------
      Variables Tolerance      VIF
    1    volume 0.8842968 1.130842
    2 species_j 0.8247492 1.212490
    3 stratum_i 0.9278966 1.077706


    Eigenvalue and Condition Index
    ------------------------------
      Eigenvalue Condition Index    intercept     volume  species_j    stratum_i
    1 3.63391993        1.000000 3.184201e-03 0.00687952 0.01296957 6.623171e-03
    2 0.25093610        3.805451 4.067109e-03 0.14908146 0.50064841 8.506720e-06
    3 0.08426133        6.567098 2.275283e-05 0.26599152 0.32764056 7.126742e-01
    4 0.03088264       10.847521 9.927259e-01 0.57804751 0.15874147 2.806942e-01

    # A tibble: 1 × 3
      studentized_residual unadjusted_p_val bonferroni_p_val
                     <dbl>            <dbl>            <dbl>
    1                 2.41           0.0609            0.609

    # A tibble: 3 × 3
      Variables Tolerance   VIF
      <chr>         <dbl> <dbl>
    1 volume        0.884  1.13
    2 species_j     0.825  1.21
    3 stratum_i     0.928  1.08

<img src="verra-task1_files/figure-gfm/unnamed-chunk-15-1.png" width="33%" /><img src="verra-task1_files/figure-gfm/unnamed-chunk-15-2.png" width="33%" /><img src="verra-task1_files/figure-gfm/unnamed-chunk-15-3.png" width="33%" /><img src="verra-task1_files/figure-gfm/unnamed-chunk-15-4.png" width="33%" /><img src="verra-task1_files/figure-gfm/unnamed-chunk-15-5.png" width="33%" /><img src="verra-task1_files/figure-gfm/unnamed-chunk-15-6.png" width="33%" /><img src="verra-task1_files/figure-gfm/unnamed-chunk-15-7.png" width="33%" />

|                    |                   |                |                 |                 |                |
|--------------------|-------------------|----------------|-----------------|-----------------|----------------|
|                    | **Breusch Pagan** | **Bonferroni** | **stratum_VIF** | **species_VIF** | **volume_VIF** |
| M1<sup>*OLS*</sup> | 1.078             | 1.212          | 1.078           | 1.212           | 1.131          |

Comparing model performances using cross validation.

``` r
# Split training and test data
dataset_tidy.samples <- createDataPartition(dataset_tidy$vji_ha_m3, p=0.70, list = FALSE)
dataset_tidy.train <- dataset_tidy[dataset_tidy.samples, ]
dataset_tidy.test <- dataset_tidy[-dataset_tidy.samples, ]
```

``` r
# Set training regime
model_training_10kfold <- trainControl(method = "repeatedcv", 
                                       number = 10, repeats = 10)
# animation of 10-kfold method:
knitr::include_graphics(path = "animation.gif")
```

![](animation.gif)<!-- -->

``` r
# Train full and test models: support vector machine*linear
svm_linear_train <- train(vji_ha_m3 ~ volume + species_j + stratum_i, 
                          data = dataset_tidy.train,
                          method = "svmLinear",
                          trControl = model_training_10kfold,
                          preProcess = c("center", "scale"),
                          tuneLength = 10
                          )

svm_linear_full <- train(vji_ha_m3 ~ volume + species_j + stratum_i, 
                          data = dataset_tidy,
                          method = "svmLinear",
                          trControl = model_training_10kfold,
                          preProcess = c("center", "scale"),
                          tuneLength = 10
                          )

# Train full and test models: randomForest regression tree
rf_train <- train(vji_ha_m3 ~ volume + species_j + stratum_i, 
                  data = dataset_tidy.train,
                  method = "rf", ntree = 1000,
                  metric = "RMSE",
                  trControl = model_training_10kfold,
                  importance = TRUE
                  )
```

    note: only 2 unique complexity parameters in default grid. Truncating the grid to 2 .

``` r
svm_linear_full_pred <- predict(svm_linear_full, data = dataset_tidy)
svm_linear_full_pred_mae <- mae(svm_linear_full_pred, dataset_tidy$vji_ha_m3)
svm_linear_full_pred_mae
```

    [1] 9.323846

``` r
svm_linear_full_pred_rmse <- rmse(svm_linear_full_pred, dataset_tidy$vji_ha_m3)
svm_linear_full_pred_rmse
```

    [1] 11.5753

``` r
#TheilU(dataset_tidy$vji_ha_m3, svm_linear_full_pred, type = 2)
TheilU(dataset_tidy$vji_ha_m3, svm_linear_full_pred, type = 1)
```

    [1] 0.08325566

``` r
svm_linear_test_pred <- predict(svm_linear_train, data = dataset_tidy.test)
svm_linear_test_pred_mae <- mae(svm_linear_test_pred, dataset_tidy.test$vji_ha_m3)
svm_linear_test_pred_mae
```

    [1] 56.6754

``` r
svm_linear_test_pred_rmse <- rmse(svm_linear_test_pred, dataset_tidy.test$vji_ha_m3)
svm_linear_test_pred_rmse
```

    [1] 66.84399

``` r
svm_linear_test_pred_rmse / svm_linear_full_pred_rmse
```

    [1] 5.774708

| Model                  | Theil U | MAE<sup>*full.model*</sup> | RMSE<sup>*full.model*</sup> | MAE<sup>*cv.model*</sup> | RMSE<sup>*cv.model*</sup> |
|------------------------|---------|----------------------------|-----------------------------|--------------------------|---------------------------|
| SVM<sup>*linear*</sup> | 0.083   | 9.324                      | 11.575                      | 53.476                   | 65.683                    |
| SVM<sup>*radial*</sup> |         |                            |                             |                          |                           |
| RF<sup>*linear*</sup>  |         |                            |                             |                          |                           |

## Appendix I: Codebook Derived

Output of auto-generated audit also stored in working directory as
`dataMaid_dataset_tidy.pdf`

``` r
knitr::include_graphics("dataMaid_dataset_tidy.png")
```

<img src="dataMaid_dataset_tidy.png" width="816" />

# Task 2

Prototype Tool for Mapping Wildfires Using the NASA-FIRMS and
Earth-Engine API’s

The digital tool for Task 2 was derived on the Google Colab platform
using python programming, a jupyter notebook and the Google Earth Engine
library. Using the following link, the tool can be viewed there in its
hosted environment with its interactive maps pre-loaded.

The Colab platform should also ensure simpler installation when sharing
externally, downloading locally, or running copies in different a Colab
project. Note that full editor settings were enabled with this link
allowing for code edits to be made by user.

Task 2 Output File Name & Webpage: `verra_task2.ipynb`

- <https://colab.research.google.com/drive/11izi_5-iYHOCWHwj17vWxWtlMPbglBiG?usp=sharing>

In addition, copies of the Colab notebook were committed to the
project’s github repository for easy cloning and revisions:

- <https://github.com/seamusrobertmurphy/verra-tasks1-2/blob/main/verra_task2.ipynb>
