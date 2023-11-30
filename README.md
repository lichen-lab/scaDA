
<!-- README.md is generated from README.Rmd. Please edit that file -->

# scaDA

Single-cell ATAC-seq sequencing data (scATAC-seq) has been a widely
adopted technology to investigate chromatin accessibility on the
single-cell level. Analyzing scATAC-seq can provide valuable insights
into identifying cell populations and revealing the epigenetic
heterogeneity across cell populations in different biological contexts.
One important aspect of scATAC-seq data analysis is performing
differential chromatin accessibility (DA) analysis, which will help
identify cell populations and reveal epigenetic heterogeneity. While
numerous differential expression methods have been proposed for
single-cell RNA sequencing data, DA methods for scATAC-seq data are
underdeveloped and remain a major challenge due to the high sparsity and
high dimensionality of the data. To fill the gap, we introduce a novel
and robust zero-inflated negative binomial framework named scaDA for DA
analysis. The model links the prevalence, mean and dispersion parameters
to covariates such as cell populations, treatment conditions, and batch
effect. The statistical inference is based on the EM algorithm and the
dispersion parameter is shrunk using an empirical Bayes method to
stabilize the parameter estimation by leveraging information from other
accessible chromatin regions in the genome. Consequently, we performed
both simulation studies and real data applications, which demonstrate
the superiority of scaDA compared to existing approaches.

## Installation

You can install the development version of scaDA from
[GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("fzhaouf/scaDA")
```

## Example

Two example datasets are included in the package to illustrate the
application of scaDA. The first dataset, Human Brain, exemplify how
scaDA identifies DA regions across various cell type from normal
samples. The AD dataset showcases the method’s capability to determine
DA regions within the same cell type under distinct states or
conditions.

# Human Brain

``` r
library(scaDA)
data("Hbrain", package = "scaDA") # load human brain dataset
```

The cell type composition is shown as following and we choose “RORB
ESR1” as interested cell type for demontration.

| cell type               | cell size | prop  |
|-------------------------|-----------|-------|
| Inh L4-6 SST B3GAT2     | 533       | 0.188 |
| Exc L5-6 RORB TTC12     | 502       | 0.177 |
| Exc L5-6 FEZF2 ABO      | 418       | 0.147 |
| Exc L5-6 FEZF2 EFTUD1P1 | 350       | 0.123 |
| Exc L3-5 RORB ESR1      | 204       | 0.072 |
| Exc L4-5 RORB FOLH1B    | 144       | 0.051 |
| Inh L1-4 LAMP5 LCP2     | 121       | 0.042 |
| Exc L5-6 THEMIS C1QL3   | 105       | 0.037 |
| Inh L2-3 VIP CASC6      | 105       | 0.037 |
| Exc L4-6 RORB SEMA3E    | 100       | 0.035 |

# construct scaDAdataset object

``` r
counts = Hbrain@assays$ATAC@counts
coldata = Hbrain@meta.data$celltype
scaDA.obj <- scaDAdatasetFromMatrix(count = as.matrix(counts), colData = data.frame(coldata))
```

# DA analysis pipeline

``` r
scaDA.obj <- estParams(scaDA.obj,celltype2 = "Inh L1-4 LAMP5 LCP2")
scaDA.obj <- shrinkDisp(scaDA.obj)
scaDA.obj <- optParams(scaDA.obj)
# report results in a dataframe
result = scaDA.obj@params$result
print(result[c(1:10),])
#>      tstats         pval          FDR       log2fc
#> 1  40.40036 8.763611e-09 4.031841e-07 -1.621236537
#> 2  17.29455 6.146924e-04 1.070779e-03 -0.299318081
#> 3  16.85549 7.567902e-04 1.238206e-03 -0.292836526
#> 4  58.29865 1.357203e-12 3.223685e-10 -3.309365963
#> 5  19.42362 2.234406e-04 5.535813e-04  0.407385310
#> 6  16.76987 7.880871e-04 1.276782e-03  0.168917984
#> 7  33.18800 2.939594e-07 7.425950e-06 -1.973135370
#> 8  18.77412 3.044304e-04 6.699012e-04 -0.484847828
#> 9  30.52778 1.068686e-06 1.995742e-05 -0.003500872
#> 10 14.88045 1.921719e-03 2.538540e-03  0.131522034
```
