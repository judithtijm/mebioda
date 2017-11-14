Inferences from tree shape
==========================

The birth/death process revisited
---------------------------------
**Nee, S, May, RM & Harvey, PH**, 1994. The reconstructed evolutionary process. 
_Philos Trans R Soc Lond B Biol Sci_ **344**:305-311

![](lecture2/birth-death.png)

- In the simplest case, there are two constant parameters: speciation rate (lambda, λ) and 
  extinction rate (mu, μ)
- [Maximum likelihood estimation](https://en.wikipedia.org/wiki/Maximum_likelihood_estimation)
  using the method of Nee et al. (1994) optimizes over `μ/λ` (or `d/b`, turnover) and
  `λ-μ` (`b-d`, net diversification)

Estimating birth/death parameters in ape
----------------------------------------
MLE of λ and μ can be obtained, for example, using `ape` in R:

```r
library(ape)
phy <- read.tree(file="PhytoPhylo.tre")

# make tree binary and ultrametric
binultra <- multi2di(force.ultrametric(phy, method = "extend"))

# fit birth/death
birthdeath(binultra)
```

Resulting in:

```
Estimation of Speciation and Extinction Rates
            with Birth-Death Models

     Phylogenetic tree: binultra 
        Number of tips: 31389 
              Deviance: -392698.6 
        Log-likelihood: 196349.3 
   Parameter estimates:
      d / b = 0.9279609   StdErr = 0.001968166 
      b - d = 0.02020561   StdErr = 0.0005033052 
   (b: speciation rate, d: extinction rate)
   Profile likelihood 95% confidence intervals:
      d / b: [0.9265351, 0.9293592]
      b - d: [0.01985037, 0.02056618]
```

Estimating birth/death parameters in phytools
---------------------------------------------

```r
library(phytools)
phy <- read.tree(file="PhytoPhylo.tre")

# make tree binary and ultrametric
binultra <- multi2di(force.ultrametric(phy, method = "extend"))

# fit birth/death
fit.bd(binultra)
```

Resulting in:

```
Fitted birth-death model:

ML(b/lambda) = 0.2805 
ML(d/mu) = 0.2603 
log(L) = 196349.2855 

Assumed sampling fraction (rho) = 1 

R thinks it has converged.
```

Quick sanity check
------------------

Save for some differences in rounding, the results are identical:
- Log likelihoods are ± identical: 196349.3 (`ape`), 196349.2855 (`phytools`)
- μ/λ = 0.9279609 (`ape`), 0.2603/0.2805 = 0.927985739750446 (`phytools`)
- λ-μ = 0.02020561 (`ape`), 0.2805-0.2603 = 0.0202 (`phytools`)

```r
library(phytools)
phy <- read.tree(file="PhytoPhylo.tre")

# make tree binary and ultrametric
binultra <- multi2di(force.ultrametric(phy, method = "extend"))

ltt.plot(binultra,log="y")
```

Resulting in:

![](lecture3/ltt.png)

Is rate constant through time?
------------------------------
**Stadler T**, 2011. Mammalian phylogeny reveals recent diversification rate shifts.
_PNAS_ **108**(15): 6187–6192

![](lecture3/sampling.jpg)

- Maybe diversification rates globally change as a function of some environmental 
  variable, e.g. climate
- In this model, speciation (λ) and extinction (μ) can have different value within 
  different time windows
- To allow for incomplete extant taxon sampling, an additional parameter (rho, ϱ) 
  captures the completeness of the sampling

```r
library(phytools)
library(TreePar)
phy <- read.tree(file="PhytoPhylo.tre")

# make tree binary and ultrametric
binultra <- multi2di(force.ultrametric(phy, method = "extend"))

# assume a near complete tree, rho[1]=0.95
rho <- c(0.95,1)

# set windows of 10myr, starting 0, ending 400myr
grid <- 10
start <- 0
end <- 400

# estimate time, lambda, mu
res <- bd.shifts.optim(x,c(rho,1),grid,start,end)[[2]]
```