Prototype Tool for Quantifying GHG in Standing Timber using Verra Carbon
Standard Methodology; VM0010 8.1.1
================
SMurphy
2023-12-04

- [Description](#description)
- [1.1 Import Data](#import)
- [1.2 Audit Data](#review)
- [1.3 Tidy Data](#transform)
- [Compute GHG Equations](#compute-ghg-equations)
- [2.1 Biomass Volumes at Plot Level
  (Eq1)](#21-biomass-volumes-at-plot-level-eq1)
- [2.2 Species Biomass at Strata Level
  (Eq2)](#22-species-biomass-at-strata-level-eq2)
- [2.3 Carbon Stocks in Harvested Volumes
  (Eq3)](#23-carbon-stocks-in-harvested-volumes-eq3)
- [2.4 Carbon Stocks in Extracted Volumes
  (Eq4)](#24-carbon-stocks-in-extracted-volumes-eq4)
- [Additional Metrics](#additional-metrics)
- [Appendix I: Codebook Derived](#appendix1)

# Description

The following workflow was used to test a prototype tool for quantifying
greenhouse gas (GHG) emissions reduction. The objective is to illustrate
the process outlined in the VCS Methodology, specifically VM0010, with
an emphasis on simplicity of tool design over burdensome complexity. The
chosen subsection for digitalization is “8.1.1 Calculation of carbon
stocks in commercial timber volumes”. A condensed script including only
essential functions of this tool is reproduced in [Appendix
I](#appendix1).

A number of empricial assumptions were made including sourcing of
pre-existing inventory dataset with pre-defined format and values
published in the exercise’s document (‘Program Officer Technology
Solutions Seamus Murphy’). Using this table of observations, an excel
spreadsheet was copied into and imported into an R environment as the
dataframe ’`dataset_raw'` and commited to the project’s github
repository
[here](https://github.com/seamusrobertmurphy/verra-stage1-GHG-tool.git).
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

# 1.1 Import Data

Import `dataset_raw` and write copy to `dataset_tidy`. Seed is set to
`77777`.

``` r
set.seed(77777)
dataset_raw = read_excel("dataset_raw.xlsx")
write.csv(dataset_raw, "dataset_tidy.csv", row.names = FALSE)
dataset_tidy = read.csv("dataset_tidy.csv")
dataset_tidy
```

<div data-pagedtable="false">

<script data-pagedtable-source type="application/json">
{"columns":[{"label":["Stratum...i."],"name":[1],"type":["int"],"align":["right"]},{"label":["Plot..sp."],"name":[2],"type":["int"],"align":["right"]},{"label":["Species..j."],"name":[3],"type":["chr"],"align":["left"]},{"label":["Tree..l."],"name":[4],"type":["chr"],"align":["left"]},{"label":["Volume..V_.l.j.I.sp.."],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"1","2":"1","3":"Sp1","4":"t1","5":"3.30"},{"1":"1","2":"1","3":"Sp1","4":"t2","5":"4.80"},{"1":"1","2":"1","3":"Sp1","4":"t3","5":"4.08"},{"1":"1","2":"2","3":"Sp4","4":"t1","5":"1.50"},{"1":"1","2":"2","3":"Sp4","4":"t2","5":"1.68"},{"1":"2","2":"1","3":"Sp1","4":"t1","5":"1.38"},{"1":"2","2":"1","3":"Sp2","4":"t2","5":"3.24"},{"1":"2","2":"1","3":"Sp3","4":"t3","5":"3.72"},{"1":"2","2":"1","3":"sp4","4":"t4","5":"2.94"},{"1":"2","2":"1","3":"Sp5","4":"t5","5":"3.36"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>

</div>

# 1.2 Audit Data

Examine structure, scan for errors, and save `dataMaid` audit report for
later use in drafting codebook and instruction list ([Appendix
I](#appendix1)).

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

<div data-pagedtable="false">

<script data-pagedtable-source type="application/json">
{"columns":[{"label":["Species..j."],"name":[1],"type":["chr"],"align":["left"]},{"label":["n"],"name":[2],"type":["int"],"align":["right"]}],"data":[{"1":"Sp1","2":"4"},{"1":"Sp2","2":"1"},{"1":"Sp3","2":"1"},{"1":"Sp4","2":"2"},{"1":"Sp5","2":"1"},{"1":"sp4","2":"1"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>

</div>

# 1.3 Tidy Data

Tests identified one problematic entry in values of `Species..j.`
variable and multiple naming issues inherited during data import.
Variable relabelling ws carried out according to naming convention
stated in exercise document, the symbology used in subsection equations
1-4 of 8.1.1 of the [VM0010 Verra
Methodology](https://verra.org/wp-content/uploads/2018/03/VM0010-Methodology-for-IMF-LtPF-v1.3_0.pdf),
and syntax standards provided in the [Tidyverse style
guide](https://style.tidyverse.org/). This involved the following
changes.

- Correct the case-sensitive error from `sp4` to `Sp4`.
- Convert variables from characters to factors and integers.
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
dataset_tidy$species_j = as.factor(dataset_tidy$species_j)
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

dataset_tidy = dataset_tidy %>%
  group_by(stratum_i) %>%
  mutate(a_I_m2 = sum(a_sp_m2), a_I_ha = sum(a_sp))

dataset_tidy %>% 
  select(stratum_i, species_j, plot_sp, tree_l, volume) %>% 
  tbl_summary(
    by = species_j,
    statistic = list(all_continuous() ~ "{mean} ({sd})",
                     all_categorical() ~ "{n} / {N} ({p}%)"),
    digits = all_continuous() ~ 1,
    type = all_categorical() ~ "categorical",
    label = list(stratum_i ~ "Strata",
                 species_j ~ "Species", 
                 plot_sp ~ "Plot ID#", 
                 tree_l ~ "Tree ID#", 
                 volume ~ "Biomass Volume (m3)"), 
              missing_text ="Missing"
              )
```

preserve7c8e3ebb6f8caff1

``` r
write.csv(dataset_tidy, "dataset_tidy.csv", row.names = FALSE)
```

<br>

# Compute GHG Equations

# 2.1 Biomass Volumes at Plot Level (Eq1)

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

<div data-pagedtable="false">

<script data-pagedtable-source type="application/json">
{"columns":[{"label":["stratum_i"],"name":[1],"type":["fct"],"align":["left"]},{"label":["plot_sp"],"name":[2],"type":["int"],"align":["right"]},{"label":["species_j"],"name":[3],"type":["fct"],"align":["left"]},{"label":["vji_sp_m3"],"name":[4],"type":["dbl"],"align":["right"]}],"data":[{"1":"1","2":"1","3":"Sp1","4":"12.18"},{"1":"1","2":"2","3":"Sp4","4":"3.18"},{"1":"2","2":"1","3":"Sp1","4":"1.38"},{"1":"2","2":"1","3":"Sp2","4":"3.24"},{"1":"2","2":"1","3":"Sp3","4":"3.72"},{"1":"2","2":"1","3":"Sp4","4":"2.94"},{"1":"2","2":"1","3":"Sp5","4":"3.36"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>

</div>

<br>

# 2.2 Species Biomass at Strata Level (Eq2)

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

<div data-pagedtable="false">

<script data-pagedtable-source type="application/json">
{"columns":[{"label":["stratum_i"],"name":[1],"type":["fct"],"align":["left"]},{"label":["species_j"],"name":[2],"type":["fct"],"align":["left"]},{"label":["vji_sp_m3"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["vji_ha_m3"],"name":[4],"type":["dbl"],"align":["right"]}],"data":[{"1":"1","2":"Sp1","3":"12.18","4":"121.8"},{"1":"1","2":"Sp1","3":"12.18","4":"121.8"},{"1":"1","2":"Sp1","3":"12.18","4":"121.8"},{"1":"1","2":"Sp4","3":"3.18","4":"31.8"},{"1":"1","2":"Sp4","3":"3.18","4":"31.8"},{"1":"2","2":"Sp1","3":"1.38","4":"13.8"},{"1":"2","2":"Sp2","3":"3.24","4":"32.4"},{"1":"2","2":"Sp3","3":"3.72","4":"37.2"},{"1":"2","2":"Sp4","3":"2.94","4":"29.4"},{"1":"2","2":"Sp5","3":"3.36","4":"33.6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>

</div>

<br>

# 2.3 Carbon Stocks in Harvested Volumes (Eq3)

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
  chb_ha_tC = vji_ha_m3 * bcef_r * cf),
  by = .(stratum_i, species_j)
]
```

<div data-pagedtable="false">

<script data-pagedtable-source type="application/json">
{"columns":[{"label":["stratum_i"],"name":[1],"type":["fct"],"align":["left"]},{"label":["species_j"],"name":[2],"type":["fct"],"align":["left"]},{"label":["vji_sp_m3"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["vji_ha_m3"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["chb_ha_tC"],"name":[5],"type":["dbl"],"align":["right"]}],"data":[{"1":"1","2":"Sp1","3":"12.18","4":"121.8","5":"42.63"},{"1":"1","2":"Sp1","3":"12.18","4":"121.8","5":"42.63"},{"1":"1","2":"Sp1","3":"12.18","4":"121.8","5":"42.63"},{"1":"1","2":"Sp4","3":"3.18","4":"31.8","5":"11.13"},{"1":"1","2":"Sp4","3":"3.18","4":"31.8","5":"11.13"},{"1":"2","2":"Sp1","3":"1.38","4":"13.8","5":"4.83"},{"1":"2","2":"Sp2","3":"3.24","4":"32.4","5":"11.34"},{"1":"2","2":"Sp3","3":"3.72","4":"37.2","5":"13.02"},{"1":"2","2":"Sp4","3":"2.94","4":"29.4","5":"10.29"},{"1":"2","2":"Sp5","3":"3.36","4":"33.6","5":"11.76"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>

</div>

<br>

# 2.4 Carbon Stocks in Extracted Volumes (Eq4)

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
  cex_ha_tC = vji_ha_m3 * d * cf),
  by = .(stratum_i, species_j)
]
```

<div data-pagedtable="false">

<script data-pagedtable-source type="application/json">
{"columns":[{"label":["stratum_i"],"name":[1],"type":["fct"],"align":["left"]},{"label":["species_j"],"name":[2],"type":["fct"],"align":["left"]},{"label":["vji_sp_m3"],"name":[3],"type":["dbl"],"align":["right"]},{"label":["vji_ha_m3"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["chb_ha_tC"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["cex_ha_tC"],"name":[6],"type":["dbl"],"align":["right"]}],"data":[{"1":"1","2":"Sp1","3":"12.18","4":"121.8","5":"42.63","6":"30.45"},{"1":"1","2":"Sp1","3":"12.18","4":"121.8","5":"42.63","6":"30.45"},{"1":"1","2":"Sp1","3":"12.18","4":"121.8","5":"42.63","6":"30.45"},{"1":"1","2":"Sp4","3":"3.18","4":"31.8","5":"11.13","6":"7.95"},{"1":"1","2":"Sp4","3":"3.18","4":"31.8","5":"11.13","6":"7.95"},{"1":"2","2":"Sp1","3":"1.38","4":"13.8","5":"4.83","6":"3.45"},{"1":"2","2":"Sp2","3":"3.24","4":"32.4","5":"11.34","6":"8.10"},{"1":"2","2":"Sp3","3":"3.72","4":"37.2","5":"13.02","6":"9.30"},{"1":"2","2":"Sp4","3":"2.94","4":"29.4","5":"10.29","6":"7.35"},{"1":"2","2":"Sp5","3":"3.36","4":"33.6","5":"11.76","6":"8.40"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>

</div>

# Additional Metrics

``` r
# vJ_ha_m3: Mean volume by species across strata (m^3.ha-1)
dataset_tidy <- dataset_tidy %>%
  group_by(species_j) %>%
  mutate(vJ_ha_m3 = mean(vji_sp_m3) * 10) 
dataset_tidy[1:10,] %>%
  select(stratum_i, species_j, 
         plot_sp, vji_ha_m3, 
         chb_ha_tC, cex_ha_tC, 
         vJ_ha_m3) %>%
  kbl(caption = "Table with right-hand showing mean biomass volume per species across all strata (m3.ha-1)", escape = T) %>%
  kable_minimal(full_width=T)
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
dataset_tidy[1:10,] %>%
  select(stratum_i, species_j, 
         plot_sp, vji_ha_m3, 
         chb_ha_tC, cex_ha_tC, 
         vJ_ha_m3, vI_ha_m3) %>%
  kbl(caption = "Table with right-hand column showing mean biomass volume per strata across all species and plots (m3.ha-1)", escape = T) %>%
  kable_minimal(full_width=T)
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
dataset_tidy = dataset_tidy %>%
  group_by(plot_sp, stratum_i) %>%
  mutate(Vsp_ha_m3 = sum(volume) * 10)
dataset_tidy[1:10,] %>%
  select(stratum_i, 
         plot_sp, vji_ha_m3, 
         chb_ha_tC, cex_ha_tC, 
         vJ_ha_m3, vI_ha_m3,
         vI_ha_m3, Vsp_ha_m3) %>%
  kbl(caption = "Table with right-hand column showing total biomass per plot across species (m3.ha-1)", escape = T) %>%
  kable_minimal(full_width=T)
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

# Appendix I: Codebook Derived

``` r
htmltools::includeHTML("dataMaid_dataset_tidy.html")
```

preserveb9572dc7bfeb0476
