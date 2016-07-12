# SnpArrays.jl

*Compressed storage for SNP data*

`SnpArrays.jl` implements the [SnpArray](@ref), [SnpData](@ref) and [HaplotypeArray](@ref) types for handling biallelic genotypes.

## Package Features

- Read and write [Plink binary files](http://pngu.mgh.harvard.edu/~purcell/plink/binary.shtml).  
- Calculate summary statistics (minor allele frequencies, minor allele, count of missing genotypes).  
- Simple and intuitive manipulation (subsetting, adjoining, assignment, imputation) of array of genotypes.  
- Generate random genotypes according to minor allele frequencies.  
- Filter an array of genotypes subject to minimum genotyping success rate per individual and per SNP.  
- Convert genotypes into dense or sparse arrays of real numbers (minor allele counts).  
- Calculate various empirical kinship matrices.  
- Extract principal components.  

## Installation

Use the Julia package manager to install VarianceComponentModels.jl.
```julia
Pkg.clone("git@github.com:OpenMendel/VarianceComponentModels.jl.git")
```
This package supports Julia `0.4`.

## Manual Outline

```@contents
Pages = [
    "man/variance_components.md",
    "man/cg10k.md",
]
Depth = 2
```
