
# Heritability Analysis

As an application of the variance component model, this note demonstrates the workflow for heritability analysis in genetics, using a sample data set `cg10k` with **6,670** individuals and **630,860** SNPs. Person IDs and phenotype names are masked for privacy. `cg10k.bed`, `cg10k.bim`, and `cg10k.fam` is a set of Plink files in binary format. `cg10k_traits.txt` contains 13 phenotypes of the 6,670 individuals.


```julia
;ls cg10k*.*
```

    cg10k.bed
    cg10k.bim
    cg10k.fam
    cg10k_traits.txt


Machine information:


```julia
versioninfo()
```

    Julia Version 0.4.6
    Commit 2e358ce (2016-06-19 17:16 UTC)
    Platform Info:
      System: Darwin (x86_64-apple-darwin13.4.0)
      CPU: Intel(R) Core(TM) i7-3720QM CPU @ 2.60GHz
      WORD_SIZE: 64
      BLAS: libopenblas (USE64BITINT DYNAMIC_ARCH NO_AFFINITY Sandybridge)
      LAPACK: libopenblas64_
      LIBM: libopenlibm
      LLVM: libLLVM-3.3


## Read in binary SNP data

We will use the [`SnpArrays.jl`](https://github.com/Hua-Zhou/SnpArrays.jl) package to read in binary SNP data and compute the empirical kinship matrix. Issue `Pkg.clone("git@github.com:OpenMendel/SnpArrays.jl.git")` within `Julia` to install the `SnpArrays` package.


```julia
#Pkg.clone("git@github.com:OpenMendel/SnpArrays.jl.git")
using SnpArrays
```


```julia
# read in genotype data from Plink binary file (~50 secs on my laptop)
@time cg10k = SnpArray("cg10k")
```

     55.188439 seconds (514.43 k allocations: 1.003 GB, 0.02% gc time)





    6670x630860 SnpArrays.SnpArray{2}:
     (false,true)   (false,true)   (true,true)   …  (true,true)    (true,true) 
     (true,true)    (true,true)    (false,true)     (false,true)   (true,false)
     (true,true)    (true,true)    (false,true)     (true,true)    (true,true) 
     (true,true)    (true,true)    (true,true)      (false,true)   (true,true) 
     (true,true)    (true,true)    (true,true)      (true,true)    (false,true)
     (false,true)   (false,true)   (true,true)   …  (true,true)    (true,true) 
     (false,false)  (false,false)  (true,true)      (true,true)    (true,true) 
     (true,true)    (true,true)    (true,true)      (true,true)    (false,true)
     (true,true)    (true,true)    (false,true)     (true,true)    (true,true) 
     (true,true)    (true,true)    (true,true)      (false,true)   (true,true) 
     (true,true)    (true,true)    (false,true)  …  (true,true)    (true,true) 
     (false,true)   (false,true)   (true,true)      (true,true)    (false,true)
     (true,true)    (true,true)    (true,true)      (true,true)    (false,true)
     ⋮                                           ⋱                             
     (false,true)   (false,true)   (true,true)      (false,true)   (false,true)
     (false,true)   (false,true)   (false,true)     (false,true)   (true,true) 
     (true,true)    (true,true)    (false,true)  …  (false,true)   (true,true) 
     (false,true)   (false,true)   (true,true)      (true,true)    (false,true)
     (true,true)    (true,true)    (true,true)      (false,true)   (true,true) 
     (true,true)    (true,true)    (true,false)     (false,false)  (false,true)
     (true,true)    (true,true)    (true,true)      (true,true)    (false,true)
     (true,true)    (true,true)    (false,true)  …  (true,true)    (true,true) 
     (true,true)    (true,true)    (true,true)      (false,true)   (true,true) 
     (true,true)    (true,true)    (true,true)      (true,true)    (false,true)
     (false,true)   (false,true)   (true,true)      (true,true)    (true,true) 
     (true,true)    (true,true)    (true,true)      (true,true)    (true,true) 



## Summary statistics of SNP data


```julia
people, snps = size(cg10k)
```




    (6670,630860)




```julia
# summary statistics (~50 secs on my laptop)
@time maf, _, missings_by_snp, = summarize(cg10k);
```

     38.734869 seconds (21 allocations: 9.753 MB, 0.01% gc time)



```julia
# 5 number summary and average MAF (minor allele frequencies)
quantile(maf, [0.0 .25 .5 .75 1.0]), mean(maf)
```




    (
    1x5 Array{Float64,2}:
     0.00841726  0.124063  0.236953  0.364253  0.5,
    
    0.24536516625042462)




```julia
using Plots
pyplot()

histogram(maf, xlab = "Minor Allele Frequency (MAF)", label = "MAF")
```




<img src="data:image/png;base64,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2Ww2ApaPedp3nv7GcYoQV8Vo+/7Lm747c+aMli5dqsGDB5dXmbhMafrt7Nmzat++verXr6+MjAzNmjWrvMvFJUrTd+PHj9eCBQtUq1at8i4TxShN3505c0adOnVSx44d9fTTT6uoqMhtHgIWAKfffvtNsbGx6tu3r+655x5flwMPVK1aVZ999pmOHTumqKgoTZs2zdclwQOvv/66QkND1bt3b1n/uZnf4qZ+v9C4cWP961//0r59+7R9+3bt2rVLzz//vNt8BCxc1Y033qi8vDxnQrcsSzk5OQoNDfVxZbia0vTd2bNnFRsbqyZNmmjRokXlXSou4c3/c1WrVlVCQoI+/NDA48TgNU/77p///Kc2btyoZs2aOQfWbteunT7//PNyrxkXeNp31apV0/XXXy9JCg4OVmJionbt2uW2PgIWrurS0fYllTjaPioeT/vu3LlziouLU7169fTaa6/5olRcwtN+y8nJUUFBgSSpqKhIa9euVdeuXcu9XvyXp323YsUK5eTkKCsry3kH2hdffKF27dqVe824wNO+O3HihM6ePStJKiws1Lp169ShQwf3FZq9RAyV1aFDh6xbb73VatGihdW5c2fryy+/tCzLsp566inr1Vdfdc4XFRVlNWrUyHI4HFbTpk2tUaNG+apk/IcnfbdixQrLZrNZ7du3d77++Mc/+rLsgOdJv/3973+32rZta7Vt29aKioqyxo4da/3000++LBuW5/9eXoq7CCsGT/pu3bp1Vps2bax27dpZrVu3tqZMmWL99ttvbutiJHcAAADDOEUIAABgGAELAADAMAIWAACAYQQsAAAAwwhYAAAAhhGwAAAADCNgAQAAGEbAAgAAMIyABXhhzpw5stvtatq0abEPaO3evbvsdrtGjx7tskzt2rXLs8yr+vTTT2W32xUZGVlse0JCgqKiopzv33jjDdntdp0+fbpU27l8PWURHh4uu93u9nrhhReMrD8QffbZZ6pTp45Lv178XIt7dNJ7773nbM/JyXFrv9r36uL/P5e/Ln5HioqK1KpVK61YscLQHgLlr4qvCwD8VdWqVXXq1Cnt3LlTvXr1ck4/cuSIPvroI9WqVUs2m805fdy4cRo4cKAvSi3RypUrFRQUpMOHD2vPnj265ZZb3Oa5dB/KwuR6hg8frunTp7tM5+Hj3ps1a5YeeughhYSEuEyvVauW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SdLx48dLrA+oqLiLECijvn37qmrVqkpKSlJ2drYz/Hh6Wqlu3bou76tWraqffvpJkpSfny/LsnTDDTe4zFOnTh1Vr17dbbiEy+crycqVK1W3bl21a9fO+YN855136uWXX9amTZuuev3WRSdPntTp06dVtWpVtzabzaa8vDw1bty42OU++eSTYpeLiIi46nbr16+vDh06lNh++efgyfby8vIkSQ0aNLjiuq7k0kB58uRJLVq0SIsWLXKb7/LTa5d/B6pVq+Zyjd3p06fVtm3bK257+PDhmjp1qlJTUzV58mSlpKTo3nvv1e9+9zuP6y/JyJEjtWzZMoWFhSk2NrbE73Zpvld2u/2KfSj99/M0dQcqUJ4IWEAZVa1aVcOGDdNf/vIX9enTR/Xr1ze27rp168pms7n9Bf/jjz+qsLDQ7ciDJz9EP//8s/73f/9Xv/76a7G1rly50uOAFRISovr162vLli3Ftpf0WdSrV08tWrRwuc7oourVq3u07Su5/HPwZHuNGjWSdOFoycX/lqRjx465zH8xHF1+HdUPP/wgh8Phss0BAwZo4sSJbtss7Xho9erV09GjR684T40aNXT//fcrJSVF3bt31+eff67Fixdfdb0Oh0OnTp264nyxsbF69NFHdejQISUlJRU7j8nv1UUX/4C4PPQC/oCABRgwduxYnThxQuPGjTO63lq1aql9+/Zau3atpk6d6py+Zs0aSVKPHj1Kvc6Lp2tee+01tzvHUlJStGrVKuXn57sdVSnOnXfeqQULFqhq1aqlGki0T58+WrFihVq2bOkc++ha8mR7UVFRCgoK0ttvv6127do5p69bt85lvqZNm0q6cEND165dJV04WnXgwAF17tzZZZsHDx5U+/btZbeX7WqMPn36aOHChcrNzdWNN95Y4nzjxo1TcnKyHnnkEbVo0ULdu3e/4nodDoeioqL05ZdfXnG+Jk2a6OGHH9bJkyed+3w5k9+ri7788kvVqFFDrVu39ngZoKIgYAEGdO7c2W28HsuyvB776dLl5syZo8GDBys+Pl7333+/Dh06pD/96U+69957vfrhWblypcLDw4sNg8HBwVq+fLnWrFmj8ePHX3Vdffr00cCBA9WvXz/NnDlTUVFROnPmjL766it99913WrJkSbHLPfLII1q5cqV69eqlqVOn6sYbb9SJEyf0ySefqEmTJpo2bVqJ2/TmM/VkeyEhIZowYYKeffZZBQUFKTo6WmlpaTp8+LDLEbGmTZuqS5cumjt3rurUqSOHw6H58+erbt26LrXNnTtXnTt3Vt++fTV+/Hg1aNBA//73v7Vjxw717NnzitdGXe7hhx/Wm2++qZ49eyopKUnNmjXT4cOHlZmZqWeffdY5X9u2bdW5c2ft3LnTZfqV3HHHHVe84/Oi559//ortJr9XF3300Ufq0aNHsad2gYqOi9wBL9hstquejrt8nuKWKW4dl883cOBArV27VgcPHtTgwYP13HPP6aGHHnJ7jIgnpwePHz+u999/X/Hx8cW2R0VFqX379lq1apXHNb/11luaMGGCXn75ZfXv319jx47V9u3b1bt37xL3KSQkRB9//LHat2+vWbNmqW/fvnrkkUeUk5NT4hEST/ezuHZPt/fss89qwoQJeu655xQbGyu73a5nn33WLdStXLlSERERSkhI0MyZM/Xwww+rU6dOLttu3ry59uzZo3r16mnixInq16+fHn/8cf3yyy8uR8g82Y+QkBB9+OGH6tGjh2bOnKm7775bL7zwQrFHswYPHiyHw6EHH3zwqtuQpNGjR+urr77SoUOHPJq/uBpL+726dNmS/Pbbb9q0aZPL46YAf2KzvP0TGwACwIYNGzR06FBlZ2f7xeN4evbsqeDgYG3cuNHjZe666y61adOm2EFufWXVqlV64oknlJmZyREs+CVOEQJAJbBv3z7t2rVLu3fvLvWAsc8++6x69eqlxx9//IpDNpSXoqIi/c///I/mzZtHuILfImABwFX4wzABt9xyi+rWraunnnpKt99+e6mWvXRYhYrAbrfrq6++8nUZQJlwihAAAMAwLnIHAAAwjIAFAABgGAELAADAsP8P9Nx8JEUYWN4AAAAASUVORK5CYII=" />




```julia
# proportion of missing genotypes
sum(missings_by_snp) / length(cg10k)
```




    0.0013128198764010824




```julia
# proportion of rare SNPs with maf < 0.05
countnz(maf .< 0.05) / length(maf)
```




    0.07228069619249913



## Empirical kinship matrix

We estimate empirical kinship based on all SNPs by the genetic relation matrix (GRM). Missing genotypes are imputed on the fly by drawing according to the minor allele frequencies.


```julia
# GRM using all SNPs (~10 mins on my laptop)
srand(123)
@time Φgrm = grm(cg10k; method = :GRM)
```

    567.048450 seconds (4.21 G allocations: 64.513 GB, 1.11% gc time)





    6670x6670 Array{Float64,2}:
      0.502916      0.00329978   -0.000116213  …  -6.46286e-5   -0.00281229 
      0.00329978    0.49892      -0.00201992       0.000909871   0.00345573 
     -0.000116213  -0.00201992    0.493632         0.000294565  -0.000349854
      0.000933977  -0.00320391   -0.0018611       -0.00241682   -0.00127078 
     -7.75429e-5   -0.0036075     0.00181442       0.00213976   -0.00158382 
      0.00200371    0.000577386   0.0025455    …   0.000943753  -1.82994e-6 
      0.000558503   0.00241421   -0.0018782        0.001217     -0.00123924 
     -0.000659495   0.00319987   -0.00101496       0.00353646   -0.00024093 
     -0.00102619   -0.00120448   -0.00055462       0.00175586    0.00181899 
     -0.00136838    0.00211996    0.000119128     -0.00147305   -0.00105239 
     -0.00206144    0.000148818  -0.000475177  …  -0.000265522  -0.00106123 
      0.000951016   0.00167042    0.00183545      -0.000703658  -0.00313334 
      0.000330442  -0.000904147   0.00301478       0.000754772  -0.00127413 
      ⋮                                        ⋱                            
      0.00301137    0.00116042    0.00100426       6.67254e-6    0.00307069 
     -0.00214008    0.00270925   -0.00185054      -0.00109935    0.00366816 
      0.000546739  -0.00242646   -0.00305264   …  -0.000629014   0.00210779 
     -0.00422553   -0.0020713    -0.00109052      -0.000705804  -0.000508055
     -0.00318405   -0.00075385    0.00312377       0.00052883   -3.60969e-5 
      0.000430196  -0.00197163    0.00268545      -0.00633175   -0.00520337 
      0.00221429    0.000849792  -0.00101111      -0.000943129  -0.000624419
     -0.00229025   -0.000130598   0.000101853  …   0.000840136  -0.00230224 
     -0.00202917    0.00233007   -0.00131006       0.00197798   -0.000513771
     -0.000964907  -0.000872326  -7.06722e-5       0.00124702   -0.00295844 
     -6.46286e-5    0.000909871   0.000294565      0.500983      0.000525615
     -0.00281229    0.00345573   -0.000349854      0.000525615   0.500792   



## Phenotypes

Read in the phenotype data and compute descriptive statistics.


```julia
using DataFrames

cg10k_trait = readtable(
    "cg10k_traits.txt"; 
    separator = ' ',
    names = [:FID; :IID; :Trait1; :Trait2; :Trait3; :Trait4; :Trait5; :Trait6; 
             :Trait7; :Trait8; :Trait9; :Trait10; :Trait11; :Trait12; :Trait13],  
    eltypes = [UTF8String; UTF8String; Float64; Float64; Float64; Float64; Float64; 
               Float64; Float64; Float64; Float64; Float64; Float64; Float64; Float64]
    )
```




<table class="data-frame"><tr><th></th><th>FID</th><th>IID</th><th>Trait1</th><th>Trait2</th><th>Trait3</th><th>Trait4</th><th>Trait5</th><th>Trait6</th><th>Trait7</th><th>Trait8</th><th>Trait9</th><th>Trait10</th><th>Trait11</th><th>Trait12</th><th>Trait13</th></tr><tr><th>1</th><td>10002K</td><td>10002K</td><td>-1.81573145026234</td><td>-0.94615046147283</td><td>1.11363077580442</td><td>-2.09867121119159</td><td>0.744416614111748</td><td>0.00139171884080131</td><td>0.934732480409667</td><td>-1.22677315418103</td><td>1.1160784277875</td><td>-0.4436280335029</td><td>0.824465656443384</td><td>-1.02852542216546</td><td>-0.394049201727681</td></tr><tr><th>2</th><td>10004O</td><td>10004O</td><td>-1.24440094378729</td><td>0.109659992547179</td><td>0.467119394241789</td><td>-1.62131304097589</td><td>1.0566758355683</td><td>0.978946979419181</td><td>1.00014633946047</td><td>0.32487427140228</td><td>1.16232175219696</td><td>2.6922706948705</td><td>3.08263672461047</td><td>1.09064954786013</td><td>0.0256616415357438</td></tr><tr><th>3</th><td>10005Q</td><td>10005Q</td><td>1.45566914502305</td><td>1.53866932923243</td><td>1.09402959376555</td><td>0.586655272226893</td><td>-0.32796454430367</td><td>-0.30337709778827</td><td>-0.0334354881314741</td><td>-0.464463064285437</td><td>-0.3319396273436</td><td>-0.486839089635991</td><td>-1.10648681564373</td><td>-1.42015780427231</td><td>-0.687463456644413</td></tr><tr><th>4</th><td>10006S</td><td>10006S</td><td>-0.768809276698548</td><td>0.513490885514249</td><td>0.244263028382142</td><td>-1.31740254475691</td><td>1.19393774326845</td><td>1.17344127734288</td><td>1.08737426675232</td><td>0.536022583732261</td><td>0.802759240762068</td><td>0.234159411749815</td><td>0.394174866891074</td><td>-0.767365892476029</td><td>0.0635385761884935</td></tr><tr><th>5</th><td>10009Y</td><td>10009Y</td><td>-0.264415132547719</td><td>-0.348240421825694</td><td>-0.0239065083413606</td><td>0.00473915802244948</td><td>1.25619191712193</td><td>1.2038883667631</td><td>1.29800739042627</td><td>0.310113660247311</td><td>0.626159861059352</td><td>0.899289129831224</td><td>0.54996783350812</td><td>0.540687809542048</td><td>0.179675416046033</td></tr><tr><th>6</th><td>10010J</td><td>10010J</td><td>-1.37617270917293</td><td>-1.47191967744564</td><td>0.291179894254146</td><td>-0.803110740704731</td><td>-0.264239977442213</td><td>-0.260573027836772</td><td>-0.165372266287781</td><td>-0.219257294118362</td><td>1.04702422290318</td><td>-0.0985815534616482</td><td>0.947393438068448</td><td>0.594014812031438</td><td>0.245407436348479</td></tr><tr><th>7</th><td>10011L</td><td>10011L</td><td>0.1009416296374</td><td>-0.191615722103455</td><td>-0.567421321596677</td><td>0.378571487240382</td><td>-0.246656179817904</td><td>-0.608810750053858</td><td>0.189081058215596</td><td>-1.27077787326519</td><td>-0.452476199143965</td><td>0.702562877297724</td><td>0.332636218957179</td><td>0.0026916503626181</td><td>0.317117176705358</td></tr><tr><th>8</th><td>10013P</td><td>10013P</td><td>-0.319818276367464</td><td>1.35774480657283</td><td>0.818689545938528</td><td>-1.15565531644352</td><td>0.63448368102259</td><td>0.291461908634679</td><td>0.933323714954726</td><td>-0.741083289682492</td><td>0.647477683507572</td><td>-0.970877627077966</td><td>0.220861165411304</td><td>0.852512250237764</td><td>-0.225904624283945</td></tr><tr><th>9</th><td>10014R</td><td>10014R</td><td>-0.288334173342032</td><td>0.566082538090752</td><td>0.254958336116175</td><td>-0.652578302869714</td><td>0.668921559277347</td><td>0.978309199170558</td><td>0.122862966041938</td><td>1.4790926378214</td><td>0.0672132424173449</td><td>0.0795903917527827</td><td>0.167532455243232</td><td>0.246915579442139</td><td>0.539932616458363</td></tr><tr><th>10</th><td>10015T</td><td>10015T</td><td>-1.15759732583991</td><td>-0.781198583545165</td><td>-0.595807759833517</td><td>-1.00554980260402</td><td>0.789828885933321</td><td>0.571058413379044</td><td>0.951304176233755</td><td>-0.295962982984816</td><td>0.99042002479707</td><td>0.561309366988983</td><td>0.733100030623233</td><td>-1.73467772245684</td><td>-1.35278484330654</td></tr><tr><th>11</th><td>10017X</td><td>10017X</td><td>0.740569150459031</td><td>1.40873846755415</td><td>0.734689999440088</td><td>0.0208322841295094</td><td>-0.337440968561619</td><td>-0.458304040611395</td><td>-0.142582512772326</td><td>-0.580392297464107</td><td>-0.684684998101516</td><td>-0.00785381461893456</td><td>-0.712244337518008</td><td>-0.313345561230878</td><td>-0.345419463162219</td></tr><tr><th>12</th><td>10020M</td><td>10020M</td><td>-0.675892486454995</td><td>0.279892613829682</td><td>0.267915996308248</td><td>-1.04103665392985</td><td>0.910741715645888</td><td>0.866027618513171</td><td>1.07414431702005</td><td>0.0381751003538302</td><td>0.766355377018601</td><td>-0.340118016143495</td><td>-0.809013958505059</td><td>0.548521663785885</td><td>-0.0201828675962336</td></tr><tr><th>13</th><td>10022Q</td><td>10022Q</td><td>-0.795410435603455</td><td>-0.699989939762738</td><td>0.3991295030063</td><td>-0.510476261900736</td><td>1.51552245416844</td><td>1.28743032939467</td><td>1.53772393250903</td><td>0.133989160117702</td><td>1.02025736886037</td><td>0.499018733899186</td><td>-0.36948273277931</td><td>-1.10153460436318</td><td>-0.598132438886619</td></tr><tr><th>14</th><td>10023S</td><td>10023S</td><td>-0.193483122930324</td><td>-0.286021160323518</td><td>-0.691494225262995</td><td>0.0131581678700699</td><td>1.52337470686782</td><td>1.4010638072262</td><td>1.53114620451896</td><td>0.333066483478075</td><td>1.04372480381099</td><td>0.163206783570466</td><td>-0.422883765001728</td><td>-0.383527976713573</td><td>-0.489221907788158</td></tr><tr><th>15</th><td>10028C</td><td>10028C</td><td>0.151246203379718</td><td>2.09185108993614</td><td>2.03800472474384</td><td>-1.12474717143531</td><td>1.66557024390713</td><td>1.62535675109576</td><td>1.58751070483655</td><td>0.635852186043776</td><td>0.842577784605979</td><td>0.450761870778952</td><td>-1.39479033623028</td><td>-0.560984107567768</td><td>0.289349776549287</td></tr><tr><th>16</th><td>10031R</td><td>10031R</td><td>-0.464608740812712</td><td>0.36127694772303</td><td>1.2327673928287</td><td>-0.826033731086383</td><td>1.43475224709983</td><td>1.74451823818846</td><td>0.211096887484638</td><td>2.64816425140548</td><td>1.02511433146096</td><td>0.11975731603184</td><td>0.0596832073448267</td><td>-0.631231612661616</td><td>-0.207878671782927</td></tr><tr><th>17</th><td>10032T</td><td>10032T</td><td>-0.732977488012215</td><td>-0.526223425889779</td><td>0.61657871336593</td><td>-0.55447974332593</td><td>0.947484859025104</td><td>0.936833214138173</td><td>0.972516806335524</td><td>0.290251013865227</td><td>1.01285359725723</td><td>0.516207422283291</td><td>-0.0300689171988194</td><td>0.8787322524583</td><td>0.450254629309513</td></tr><tr><th>18</th><td>10034X</td><td>10034X</td><td>-0.167326459622119</td><td>0.175327165487237</td><td>0.287467725892572</td><td>-0.402652532084246</td><td>0.551181509418056</td><td>0.522204743290975</td><td>0.436837660094653</td><td>0.299564933845579</td><td>0.583109520896067</td><td>-0.704415820005353</td><td>-0.730810367994577</td><td>-1.95140580379896</td><td>-0.933504665700164</td></tr><tr><th>19</th><td>10035Z</td><td>10035Z</td><td>1.41159485787418</td><td>1.78722407901017</td><td>0.84397639585364</td><td>0.481278083772991</td><td>-0.0887673728508268</td><td>-0.49957757426858</td><td>0.304195897924847</td><td>-1.23884208383369</td><td>-0.153475724036624</td><td>-0.870486102788329</td><td>0.0955473331150403</td><td>-0.983708050882817</td><td>-0.3563445644514</td></tr><tr><th>20</th><td>10041U</td><td>10041U</td><td>-1.42997091652825</td><td>-0.490147045034213</td><td>0.272730237607695</td><td>-1.61029992954153</td><td>0.990787817197748</td><td>0.711687532608184</td><td>1.1885836012715</td><td>-0.371229188075638</td><td>1.24703459239952</td><td>-0.0389162332271516</td><td>0.883495749072872</td><td>2.58988026321017</td><td>3.33539552370368</td></tr><tr><th>21</th><td>10047G</td><td>10047G</td><td>-0.147247288176765</td><td>0.12328430415652</td><td>0.617549051912237</td><td>-0.18713077178262</td><td>0.256438107586694</td><td>0.17794983735083</td><td>0.412611806463263</td><td>-0.244809124559737</td><td>0.0947624806136492</td><td>0.723017223849532</td><td>-0.683948354633436</td><td>0.0873751276309269</td><td>-0.262209652750371</td></tr><tr><th>22</th><td>10051X</td><td>10051X</td><td>-0.187112676773894</td><td>-0.270777264595619</td><td>-1.01556818551606</td><td>0.0602850568600233</td><td>0.272419757757978</td><td>0.869133161879197</td><td>-0.657519461414234</td><td>2.32388522018189</td><td>-0.999936011525034</td><td>1.44671844178306</td><td>0.971157886040772</td><td>-0.358747904241515</td><td>-0.439657942096136</td></tr><tr><th>23</th><td>10052Z</td><td>10052Z</td><td>-1.82434047163768</td><td>-0.933480446068067</td><td>1.29474003766977</td><td>-1.94545221151036</td><td>0.33584651189654</td><td>0.359201654302844</td><td>0.513652924365886</td><td>-0.073197696696958</td><td>1.57139042812005</td><td>1.53329371326728</td><td>1.82076821859528</td><td>2.22740301867829</td><td>1.50063347195857</td></tr><tr><th>24</th><td>10056H</td><td>10056H</td><td>-2.29344084351335</td><td>-2.49161842344418</td><td>0.40383988742336</td><td>-2.36488074752948</td><td>1.4105254831956</td><td>1.42244117147792</td><td>1.17024166272172</td><td>0.84476650176855</td><td>1.79026875432495</td><td>0.648181858970515</td><td>-0.0857231057403538</td><td>-1.02789535292617</td><td>0.491288088952859</td></tr><tr><th>25</th><td>10057J</td><td>10057J</td><td>-0.434135932888305</td><td>0.740881989034652</td><td>0.699576357578518</td><td>-1.02405543187775</td><td>0.759529223983713</td><td>0.956656110895288</td><td>0.633299568656589</td><td>0.770733932268516</td><td>0.824988511714526</td><td>1.84287437634769</td><td>1.91045942063443</td><td>-0.502317207869366</td><td>0.132670133448219</td></tr><tr><th>26</th><td>10058L</td><td>10058L</td><td>-2.1920969546557</td><td>-2.49465664272271</td><td>0.354854763893431</td><td>-1.93155848635714</td><td>0.941979400289938</td><td>0.978917101414106</td><td>0.894860097289736</td><td>0.463239402831873</td><td>1.12537133317163</td><td>1.70528446191955</td><td>0.717792714479123</td><td>0.645888049108261</td><td>0.783968250169388</td></tr><tr><th>27</th><td>10060Y</td><td>10060Y</td><td>-1.46602269088422</td><td>-1.24921677101897</td><td>0.307977693653039</td><td>-1.55097364660989</td><td>0.618908494474798</td><td>0.662508171662042</td><td>0.475957173906078</td><td>0.484718674597707</td><td>0.401564892028249</td><td>0.55987973254026</td><td>-0.376938143754217</td><td>-0.933982629228218</td><td>0.390013151672955</td></tr><tr><th>28</th><td>10062C</td><td>10062C</td><td>-1.83317744236881</td><td>-1.53268787828701</td><td>2.55674262685865</td><td>-1.51827745783835</td><td>0.789409601746455</td><td>0.908747799728588</td><td>0.649971922941479</td><td>0.668373649931667</td><td>1.20058303519903</td><td>0.277963256075637</td><td>1.2504953198275</td><td>3.31370445071638</td><td>2.22035828885342</td></tr><tr><th>29</th><td>10064G</td><td>10064G</td><td>-0.784546628243178</td><td>0.276582579543931</td><td>3.01104958800057</td><td>-1.11978843206758</td><td>0.920823858422707</td><td>0.750217689886151</td><td>1.26153730009639</td><td>-0.403363882922417</td><td>0.400667296857811</td><td>-0.217597941303479</td><td>-0.724669537565068</td><td>-0.391945338467193</td><td>-0.650023936358253</td></tr><tr><th>30</th><td>10065I</td><td>10065I</td><td>0.464455916345135</td><td>1.3326356122229</td><td>-1.23059563374303</td><td>-0.357975958937414</td><td>1.18249746977104</td><td>1.54315938069757</td><td>-0.60339041154062</td><td>3.38308845958422</td><td>0.823740765148641</td><td>-0.129951318508883</td><td>-0.657979878422938</td><td>-0.499534924074273</td><td>-0.414476569095651</td></tr><tr><th>&vellip;</th><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td><td>&vellip;</td></tr></table>




```julia
describe(cg10k_trait)
```

    FID
    Length  6670
    Type    UTF8String
    NAs     0
    NA%     0.0%
    Unique  6670
    
    IID
    Length  6670
    Type    UTF8String
    NAs     0
    NA%     0.0%
    Unique  6670
    
    Trait1
    Min      -3.2041280147118
    1st Qu.  -0.645770976594801
    Median   0.12500996951180798
    Mean     0.0022113846331389903
    3rd Qu.  0.723315489763611
    Max      3.47939787136478
    NAs      0
    NA%      0.0%
    
    Trait2
    Min      -3.51165862877157
    1st Qu.  -0.6426205239938769
    Median   0.0335172506981786
    Mean     0.0013525291443179934
    3rd Qu.  0.6574666174104795
    Max      4.91342267449592
    NAs      0
    NA%      0.0%
    
    Trait3
    Min      -3.93843646263987
    1st Qu.  -0.6409067201835313
    Median   -0.000782161570259152
    Mean     -0.0012959062525954158
    3rd Qu.  0.6371084235689337
    Max      7.91629946619107
    NAs      0
    NA%      0.0%
    
    Trait4
    Min      -3.60840330795393
    1st Qu.  -0.5460856267376792
    Median   0.228165419346029
    Mean     0.0023089259432067487
    3rd Qu.  0.7152907338009037
    Max      3.12768818152017
    NAs      0
    NA%      0.0%
    
    Trait5
    Min      -4.14874907974159
    1st Qu.  -0.6907651815712426
    Median   0.03103429560265845
    Mean     -0.0017903947913742396
    3rd Qu.  0.7349158784775833
    Max      2.71718436484651
    NAs      0
    NA%      0.0%
    
    Trait6
    Min      -3.82479174931095
    1st Qu.  -0.6627963069407601
    Median   0.03624198629403995
    Mean     -0.001195980597331062
    3rd Qu.  0.7411755243742835
    Max      2.58972802240228
    NAs      0
    NA%      0.0%
    
    Trait7
    Min      -4.27245540828955
    1st Qu.  -0.6389232588654877
    Median   0.0698010018021233
    Mean     -0.0019890555724116853
    3rd Qu.  0.7104228734967848
    Max      2.65377857124275
    NAs      0
    NA%      0.0%
    
    Trait8
    Min      -5.62548796912517
    1st Qu.  -0.6015747053036895
    Median   -0.0386301401797661
    Mean     0.0006140754882985941
    3rd Qu.  0.5273417705306229
    Max      5.8057022359485
    NAs      0
    NA%      0.0%
    
    Trait9
    Min      -5.38196778211456
    1st Qu.  -0.6014287731518224
    Median   0.10657100636146799
    Mean     -0.0018096522573535152
    3rd Qu.  0.6985667613073132
    Max      2.57193558386964
    NAs      0
    NA%      0.0%
    
    Trait10
    Min      -3.54850550601412
    1st Qu.  -0.6336406665339003
    Median   -0.0966507436079331
    Mean     -0.0004370294352533275
    3rd Qu.  0.49861036457390195
    Max      6.53782005410551
    NAs      0
    NA%      0.0%
    
    Trait11
    Min      -3.26491021902041
    1st Qu.  -0.6736846396608624
    Median   -0.0680437088585371
    Mean     -0.0006159181111862523
    3rd Qu.  0.6554864114250585
    Max      4.26240968462615
    NAs      0
    NA%      0.0%
    
    Trait12
    Min      -8.85190902714652
    1st Qu.  -0.5396855871831098
    Median   -0.1410985990171995
    Mean     -0.0005887830910961934
    3rd Qu.  0.35077884186653374
    Max      13.2114017261714
    NAs      0
    NA%      0.0%
    
    Trait13
    Min      -5.59210353493304
    1st Qu.  -0.49228904714392474
    Median   -0.14102175804213551
    Mean     -0.0001512383146454028
    3rd Qu.  0.32480412746681697
    Max      24.174436145414
    NAs      0
    NA%      0.0%
    



```julia
Y = convert(Matrix{Float64}, cg10k_trait[:, 3:15])
histogram(Y, layout = 13)
```




<img src="data:image/png;base64,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+3Cgch+wJ7txcMLmQilwperh/wHgSqX1z2r/9lFPoKHefr154NMVALh//z48PDzs6g1Neno6vL29mXQ01j47mq/WaFJ6pcZvaZn9rs8YOKOxbduxxGofaIvCwkLLi5QC1NXVUa1bt6bu379PURRFPXjwgGrfvj1VUVHB+V1NTQ0VHh5OAdD/nOwvPDycqqmpUUKddL1qQn+6Xul/ul7pf87yZ65XiqxgtWvXDjExMVi1ahXGjh2LdevWoWPHjhbLoh06dMDOnTs5S8Q6zkGHDh3QoUMHRe+p65Xro+uVjhzoeqUjB+Z65UZRLF//MlJWVoZx48bhypUraNOmDbKzs9GlSxclbq3jwuh6pSMHul7pyIGuV00LxQZYOjo6Ojo6OjpNBc17ctfR0dHR0dHRcTacYoCVmZmJiIgIREdHo2vXrli4cKGg9P/4xz/QrVs3REREIDIyEj/88IPgMvzrX/9CbGwsPD098fbbbzuURqrX3unTpyMoKAju7u4oKSkRXOY7d+5g6NChCAsLQ1RUFBISElBRUSE4n4SEBERGRiI6Ohq9e/eW7BhPi4jRMTF65ageidEdMfoiRkek6EN2djbc3d15I9Y3daS2U6S8hJNqN9hIfe537tzB1KlTERoaioiICIwZM0ZSeVwJvX9Ur3+02xYqdoRCAteuXWP+f/36dSogIIDav3+/w+l37NhBXb9+naIoijp37hzl4+NjcXLDHmVlZdSRI0eo999/n5oxY4ZDafr370999913FEVR1I8//kh1795d0D0LCwupX3/9lQoMDKSOHDkiKC1FUdR///tfasuWLcznpUuXUv369ROcD1v+GzZsoDp37iw4D60jRsfE6JWjeiRGd8ToixgdEasPlZWVVK9evahevXpRmzZtcihNU0JqOyW1vaEh1W7QkHjuM2bMoKZPn858rqurE10eV0PvH9XrH+21hU6xgtW6dWvm/7RjO9qHiiMMGDCA8Tzt7++P9u3b49dffxVUhpCQEERERKBZM8cOXtJee2lfJsOGDcO5c+dw5swZh+8ZHx8PPz8/QeVk07x5c45Tuh49eqCqqkpwPmz519fX46mnnhJdJq0iRsfE6JUjeiRWd8ToixgdEaMPDx48wKRJk7BkyRI8+uijgsrYVJDSTpFob2hItRsAmed+69YtrFixAh999BFzrV27dqLyckX0/lEc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" />



## Pre-processing data for heritability analysis

To prepare variance component model fitting, we form an instance of `VarianceComponentVariate`. The two variance components are $(2\Phi, I)$.


```julia
using VarianceComponentModels

# form data as VarianceComponentVariate
cg10kdata = VarianceComponentVariate(Y, (2Φgrm, eye(size(Y, 1))))
fieldnames(cg10kdata)
```




    3-element Array{Symbol,1}:
     :Y
     :X
     :V




```julia
cg10kdata
```




    VarianceComponentModels.VarianceComponentVariate{Float64,2,Array{Float64,2},Array{Float64,2},Array{Float64,2}}(6670x13 Array{Float64,2}:
     -1.81573   -0.94615     1.11363    …   0.824466  -1.02853     -0.394049 
     -1.2444     0.10966     0.467119       3.08264    1.09065      0.0256616
      1.45567    1.53867     1.09403       -1.10649   -1.42016     -0.687463 
     -0.768809   0.513491    0.244263       0.394175  -0.767366     0.0635386
     -0.264415  -0.34824    -0.0239065      0.549968   0.540688     0.179675 
     -1.37617   -1.47192     0.29118    …   0.947393   0.594015     0.245407 
      0.100942  -0.191616   -0.567421       0.332636   0.00269165   0.317117 
     -0.319818   1.35774     0.81869        0.220861   0.852512    -0.225905 
     -0.288334   0.566083    0.254958       0.167532   0.246916     0.539933 
     -1.1576    -0.781199   -0.595808       0.7331    -1.73468     -1.35278  
      0.740569   1.40874     0.73469    …  -0.712244  -0.313346    -0.345419 
     -0.675892   0.279893    0.267916      -0.809014   0.548522    -0.0201829
     -0.79541   -0.69999     0.39913       -0.369483  -1.10153     -0.598132 
      ⋮                                 ⋱   ⋮                                
     -0.131005   0.425378   -1.09015        0.35674    0.456428     0.882577 
     -0.52427    1.04173     1.13749        0.366737   1.78286      1.90764  
      1.32516    0.905899    0.84261    …  -0.418756  -0.275519    -0.912778 
     -1.44368   -2.55708    -0.868193       1.31914   -1.44981     -1.77373  
     -1.8518    -1.25726     1.81724        0.770329  -0.0470789    1.50496  
     -0.810034   0.0896703   0.530939       0.757479   1.10001      1.29115  
     -1.22395   -1.48953    -2.95847        1.29209    0.697478     0.228819 
     -0.282847  -1.54129    -1.38819    …   1.00973   -0.362158    -1.55022  
      0.475008   1.46697     0.497403       0.141684   0.183218     0.122664 
     -0.408154  -0.325323    0.0850869     -0.2214    -0.575183     0.399583 
      0.886626   0.487408   -0.0977307     -0.985545  -0.636874    -0.439825 
     -1.24394    0.213697    2.74965        1.39201    0.299931     0.392809 ,6670x0 Array{Float64,2},(
    6670x6670 Array{Float64,2}:
      1.00583       0.00659955   -0.000232427  …  -0.000129257  -0.00562459 
      0.00659955    0.99784      -0.00403985       0.00181974    0.00691145 
     -0.000232427  -0.00403985    0.987264         0.00058913   -0.000699707
      0.00186795   -0.00640781   -0.00372219      -0.00483365   -0.00254155 
     -0.000155086  -0.00721501    0.00362883       0.00427952   -0.00316764 
      0.00400741    0.00115477    0.005091     …   0.00188751   -3.65987e-6 
      0.00111701    0.00482842   -0.00375641       0.00243399   -0.00247849 
     -0.00131899    0.00639975   -0.00202991       0.00707293   -0.00048186 
     -0.00205238   -0.00240896   -0.00110924       0.00351173    0.00363799 
     -0.00273677    0.00423992    0.000238256     -0.0029461    -0.00210478 
     -0.00412287    0.000297635  -0.000950353  …  -0.000531045  -0.00212246 
      0.00190203    0.00334083    0.0036709       -0.00140732   -0.00626668 
      0.000660883  -0.00180829    0.00602955       0.00150954   -0.00254826 
      ⋮                                        ⋱                            
      0.00602273    0.00232083    0.00200852       1.33451e-5    0.00614137 
     -0.00428016    0.0054185    -0.00370108      -0.00219871    0.00733631 
      0.00109348   -0.00485292   -0.00610528   …  -0.00125803    0.00421559 
     -0.00845106   -0.00414261   -0.00218104      -0.00141161   -0.00101611 
     -0.00636811   -0.0015077     0.00624753       0.00105766   -7.21938e-5 
      0.000860393  -0.00394326    0.0053709       -0.0126635    -0.0104067  
      0.00442858    0.00169958   -0.00202223      -0.00188626   -0.00124884 
     -0.0045805    -0.000261196   0.000203706  …   0.00168027   -0.00460447 
     -0.00405834    0.00466013   -0.00262013       0.00395595   -0.00102754 
     -0.00192981   -0.00174465   -0.000141344      0.00249404   -0.00591688 
     -0.000129257   0.00181974    0.00058913       1.00197       0.00105123 
     -0.00562459    0.00691145   -0.000699707      0.00105123    1.00158    ,
    
    6670x6670 Array{Float64,2}:
     1.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  …  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  1.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  1.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  1.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  1.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  1.0  0.0  0.0  …  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  1.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  …  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     ⋮                        ⋮              ⋱            ⋮                      
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  …  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     1.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  1.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  …  0.0  0.0  1.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  1.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  1.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  1.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0     0.0  0.0  0.0  0.0  0.0  0.0  1.0))



Before fitting the variance component model, we pre-compute the eigen-decomposition of $2\Phi_{\text{GRM}}$, the rotated responses, and the constant part in log-likelihood, and store them as a `TwoVarCompVariateRotate` instance, which is re-used in various variane component estimation procedures.


```julia
# pre-compute eigen-decomposition (~50 secs on my laptop)
@time cg10kdata_rotated = TwoVarCompVariateRotate(cg10kdata)
fieldnames(cg10kdata_rotated)
```

     55.887248 seconds (1.06 M allocations: 1.043 GB, 0.28% gc time)





    4-element Array{Symbol,1}:
     :Yrot    
     :Xrot    
     :eigval  
     :logdetV2



## Save intermediate results

We don't want to re-compute SnpArray and empirical kinship matrices again and again for heritibility analysis.


```julia
#using JLD
#@save "cg10k.jld"
#whos()
```

                     #210#wsession    280 bytes  JLD.JldWriteSession
                     #212#wsession    280 bytes  JLD.JldWriteSession
                              Base  34564 KB     Module
                           BinDeps    207 KB     Module
                             Blosc     37 KB     Module
                        ColorTypes    311 KB     Module
                            Colors    715 KB     Module
                            Compat    323 KB     Module
                             Conda     65 KB     Module
                              Core   6836 KB     Module
                        DataArrays    782 KB     Module
                        DataFrames   1822 KB     Module
                            Docile    414 KB     Module
                            FileIO    536 KB     Module
                 FixedPointNumbers     32 KB     Module
                   FixedSizeArrays    157 KB     Module
                              GZip    771 KB     Module
                              HDF5   3209 KB     Module
                            IJulia 2083312 KB     Module
                    IPythonDisplay     36 KB     Module
                             Ipopt     26 KB     Module
                  IterativeSolvers    485 KB     Module
                               JLD   1240 KB     Module
                              JSON    239 KB     Module
                            KNITRO    274 KB     Module
                      LaTeXStrings   3108 bytes  Module
                     LegacyStrings     12 KB     Module
                        MacroTools    123 KB     Module
                              Main 2145169 KB     Module
                      MathProgBase    302 KB     Module
                          Measures     15 KB     Module
                            Nettle     58 KB     Module
                         PlotUtils    441 KB     Module
                             Plots   2721 KB     Module
                            PyCall   1016 KB     Module
                            PyPlot   1252 KB     Module
                       RecipesBase    174 KB     Module
                          Reexport   3648 bytes  Module
                               SHA     50 KB     Module
                         SnpArrays    439 KB     Module
                 SortingAlgorithms     39 KB     Module
                         StatsBase    800 KB     Module
                         StatsFuns    286 KB     Module
                         URIParser    102 KB     Module
           VarianceComponentModels    302 KB     Module
                                 Y    677 KB     6670x13 Array{Float64,2}
                               ZMQ     81 KB     Module
                                 _     77 KB     630860-element BitArray{1}
                             cg10k 1027303 KB     6670x630860 SnpArrays.SnpArray{2}
                       cg10k_trait    978 KB     6670×15 DataFrames.DataFrame
                         cg10kdata 695816 KB     VarianceComponentModels.VarianceCo…
                 cg10kdata_rotated    729 KB     VarianceComponentModels.TwoVarComp…
                               maf   4928 KB     630860-element Array{Float64,1}
                   missings_by_snp   4928 KB     630860-element Array{Int64,1}
                            people      8 bytes  Int64
                              snps      8 bytes  Int64
                              Φgrm 347569 KB     6670x6670 Array{Float64,2}


To load workspace


```julia
#using SnpArrays, JLD, DataFrames, VarianceComponentModels, Plots
#pyplot()
#@load "cg10k.jld"
#whos()
```

## Heritability of single traits

We use Fisher scoring algorithm to fit variance component model for each single trait.


```julia
# heritability from single trait analysis
hST = zeros(13)
# standard errors of estimated heritability
hST_se = zeros(13)
# additive genetic effects
σ2a = zeros(13)
# enviromental effects
σ2e = zeros(13)

@time for trait in 1:13
    println(names(cg10k_trait)[trait + 2])
    # form data set for trait j
    traitj_data = TwoVarCompVariateRotate(cg10kdata_rotated.Yrot[:, trait], cg10kdata_rotated.Xrot, 
        cg10kdata_rotated.eigval, cg10kdata_rotated.logdetV2)
    # initialize model parameters
    traitj_model = VarianceComponentModel(traitj_data)
    # estimate variance components
    _, _, _, Σcov, _, _ = mle_fs!(traitj_model, traitj_data; solver=:Ipopt, verbose=false)
    σ2a[trait] = traitj_model.Σ[1][1]
    σ2e[trait] = traitj_model.Σ[2][1]
    @show σ2a[trait], σ2e[trait]
    h, hse = heritability(traitj_model.Σ, Σcov)
    hST[trait] = h[1]
    hST_se[trait] = hse[1]
end
```

    Trait1
    (σ2a[trait],σ2e[trait]) = (0.26104123217397407,0.7356884432614137)
    Trait2
    (σ2a[trait],σ2e[trait]) = (0.1887414738028781,0.8106899991616237)
    Trait3
    (σ2a[trait],σ2e[trait]) = (0.31857192765473236,0.6801458862875933)
    Trait4
    (σ2a[trait],σ2e[trait]) = (0.26556901333953215,0.7303588364945378)
    Trait5
    (σ2a[trait],σ2e[trait]) = (0.28123321193920503,0.7167989047155238)
    Trait6
    (σ2a[trait],σ2e[trait]) = (0.2829461149704314,0.716562953439665)
    Trait7
    (σ2a[trait],σ2e[trait]) = (0.2154385640394616,0.7816211121586024)
    Trait8
    (σ2a[trait],σ2e[trait]) = (0.19412648732666243,0.8055277649986169)
    Trait9
    (σ2a[trait],σ2e[trait]) = (0.24789561127297127,0.7504615853619782)
    Trait10
    (σ2a[trait],σ2e[trait]) = (0.10007455815563934,0.8998152773605567)
    Trait11
    (σ2a[trait],σ2e[trait]) = (0.1648677816930128,0.8338002257315535)
    Trait12
    (σ2a[trait],σ2e[trait]) = (0.08298660416199151,0.9158035668415299)
    Trait13
    (σ2a[trait],σ2e[trait]) = (0.05684248094793726,0.942365338132603)
      1.277847 seconds (27.43 M allocations: 449.203 MB, 6.96% gc time)



```julia
# heritability and standard errors
[hST'; hST_se']
```




    2x13 Array{Float64,2}:
     1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0
     1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0  1.0



## Pairwise traits

Joint analysis of multiple traits is subject to intensive research recently. Following code snippet does joint analysis of all pairs of traits, a total of 78 bivariate variane component models.


```julia
# additive genetic effects (2x2 psd matrices) from bavariate trait analysis;
Σa = Array{Matrix{Float64}}(13, 13)
# environmental effects (2x2 psd matrices) from bavariate trait analysis;
Σe = Array{Matrix{Float64}}(13, 13)

@time for i in 1:13
    for j in (i+1):13
        println(names(cg10k_trait)[i + 2], names(cg10k_trait)[j + 2])
        # form data set for (trait1, trait2)
        traitij_data = TwoVarCompVariateRotate(cg10kdata_rotated.Yrot[:, [i;j]], cg10kdata_rotated.Xrot, 
            cg10kdata_rotated.eigval, cg10kdata_rotated.logdetV2)
        # initialize model parameters
        traitij_model = VarianceComponentModel(traitij_data)
        # estimate variance components
        mle_fs!(traitij_model, traitij_data; solver=:Ipopt, verbose=false)
        Σa[i, j] = traitij_model.Σ[1]
        Σe[i, j] = traitij_model.Σ[2]
        @show Σa[i, j], Σe[i, j]
    end
end
```

    Trait1Trait2
    (Σa[i,j],Σe[i,j]) = (
    [0.26011943486601186 0.1762158250617613
     0.1762158250617613 0.18737615484007947],
    
    [0.7365894055260143 0.5838920954615305
     0.5838920954615305 0.8120331390284958])
    Trait1Trait3
    (Σa[i,j],Σe[i,j]) = (
    [0.2615639935561912 -0.013126818537540254
     -0.013126818537540254 0.3190566225472985],
    
    [0.7351802111599617 -0.12112674834388097
     -0.12112674834388097 0.6796789899243143])
    Trait1Trait4
    (Σa[i,j],Σe[i,j]) = (
    [0.2608796031577277 0.22261440559416618
     0.22261440559416618 0.2655808327606219],
    
    [0.735845998165887 0.5994353345883121
     0.5994353345883121 0.7303474474501164])
    Trait1Trait5
    (Σa[i,j],Σe[i,j]) = (
    [0.2607830377026644 -0.14701178378031446
     -0.14701178378031446 0.28187724546298826],
    
    [0.7359373011287004 -0.25458389055860414
     -0.25458389055860414 0.7161761242033678])
    Trait1Trait6
    (Σa[i,j],Σe[i,j]) = (
    [0.2607070355193131 -0.12935642592773852
     -0.12935642592773852 0.28318838486253467],
    
    [0.7360128807197885 -0.231361283307625
     -0.231361283307625 0.716329432342681])
    Trait1Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.26030750074784287 -0.1402575370655325
     -0.1402575370655325 0.2150805562425413],
    
    [0.7364055998763509 -0.19780547645064997
     -0.19780547645064997 0.7819851899285041])
    Trait1Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.2610345999103056 -0.03352962807192369
     -0.03352962807192369 0.19414307314249296],
    
    [0.7356949687057843 -0.12627246367180997
     -0.12627246367180997 0.8055115370762379])
    Trait1Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.2630163159971886 -0.20486492716336502
     -0.20486492716336502 0.24679565235659717],
    
    [0.7337944623091683 -0.30745013667607846
     -0.30745013667607846 0.7515442213452723])
    Trait1Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.26089807908065815 -0.0998175618122165
     -0.0998175618122165 0.09702328543308451],
    
    [0.7358279769605037 -0.30360875962256173
     -0.30360875962256173 0.9028534651901434])
    Trait1Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.2607397076711653 -0.13898341539579964
     -0.13898341539579964 0.1630626318546552],
    
    [0.735982002778579 -0.35917453215273204
     -0.35917453215273204 0.8355950431900447])
    Trait1Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.26306860075660077 -0.14553553813998849
     -0.14553553813998849 0.08051357250838533],
    
    [0.7337809595143813 -0.04169751340916241
     -0.04169751340916241 0.9183594535088752])
    Trait1Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.26234367607968223 -0.10889551714052524
     -0.10889551714052524 0.051294038316720095],
    
    [0.7344496461774143 -0.11399558206598119
     -0.11399558206598119 0.9479424008071721])
    Trait2Trait3
    (Σa[i,j],Σe[i,j]) = (
    [0.18901532602813093 0.14615743012019033
     0.14615743012019033 0.32052865893860505],
    
    [0.8104184413504798 0.09749923852684478
     0.09749923852684478 0.6782713240483099])
    Trait2Trait4
    (Σa[i,j],Σe[i,j]) = (
    [0.18839514990246123 0.07521464811372748
     0.07521464811372748 0.2655584804130378],
    
    [0.811030102837779 0.2204948315759956
     0.2204948315759956 0.7303691342564813])
    Trait2Trait5
    (Σa[i,j],Σe[i,j]) = (
    [0.18871644001954374 -0.01131401822579143
     -0.01131401822579143 0.2812465335267498],
    
    [0.810714524791035 -0.037010470173424806
     -0.037010470173424806 0.7167859986688715])
    Trait2Trait6
    (Σa[i,j],Σe[i,j]) = (
    [0.18877375983465075 -0.00310660369976054
     -0.00310660369976054 0.28301251325859456],
    
    [0.8106583657780116 -0.021182656859443615
     -0.021182656859443615 0.7164985874171053])
    Trait2Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.1883522257140152 -0.02995792853603445
     -0.02995792853603445 0.21518854248912628],
    
    [0.8110719442656862 -0.0013693865327215602
     -0.0013693865327215602 0.7818678818059402])
    Trait2Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.18926168900120724 0.03314229844995283
     0.03314229844995283 0.19466629556483447],
    
    [0.8101822287422975 -0.0326002707144356
     -0.0326002707144356 0.8050045055369501])
    Trait2Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.18728489562655376 -0.08541458777339436
     -0.08541458777339436 0.24671880340597185],
    
    [0.8121330456004312 -0.08087908481059602
     -0.08087908481059602 0.7516171286974279])
    Trait2Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.18896456296522746 -0.12531880400522133
     -0.12531880400522133 0.10012137187907984],
    
    [0.8104983819177826 -0.2710710218698844
     -0.2710710218698844 0.8998490679641975])
    Trait2Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.18776200371864812 -0.1184792033068952
     -0.1184792033068952 0.16627341912779045],
    
    [0.8116528153526548 -0.2955489949517167
     -0.2955489949517167 0.832437271728203])
    Trait2Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.1881906352068438 -0.09053833116422316
     -0.09053833116422316 0.08226341390094308],
    
    [0.8112716597648593 0.04542203421425378
     0.04542203421425378 0.9165863321461983])
    Trait2Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.18826030571129157 -0.07070412373895248
     -0.07070412373895248 0.05472389417033724],
    
    [0.8112166105484152 0.07379770160245963
     0.07379770160245963 0.944520839730775])
    Trait3Trait4
    (Σa[i,j],Σe[i,j]) = (
    [0.31852039539972626 -0.15433893723696576
     -0.15433893723696576 0.2647540990554418],
    
    [0.6801958865837233 -0.3034399519706689
     -0.3034399519706689 0.7311518515398231])
    Trait3Trait5
    (Σa[i,j],Σe[i,j]) = (
    [0.31896997787053516 0.18435446676116027
     0.18435446676116027 0.28250017723831466],
    
    [0.6797599597910079 0.3364105248337092
     0.3364105248337092 0.7155665412298342])
    Trait3Trait6
    (Σa[i,j],Σe[i,j]) = (
    [0.31956636442184144 0.16663988772517022
     0.16663988772517022 0.28503131823058603],
    
    [0.6791832508344973 0.2976976595437346
     0.2976976595437346 0.7145358500001604])
    Trait3Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.3185755051457358 0.16685216000502112
     0.16685216000502112 0.21523224225933885],
    
    [0.6801424314858139 0.3471388423850005
     0.3471388423850005 0.781823130995854])
    Trait3Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.3204992348069676 0.05753194037710734
     0.05753194037710734 0.1972448985403006],
    
    [0.6782830498092438 0.04425974188550198
     0.04425974188550198 0.802473783267546])
    Trait3Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.3187191200121132 0.13729240537787007
     0.13729240537787007 0.24697586633850704],
    
    [0.6800039145599909 0.26710543782880997
     0.26710543782880997 0.7513573840679394])
    Trait3Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.3189152132176088 -0.07863382344383743
     -0.07863382344383743 0.10110317193536325],
    
    [0.6798145588849664 -0.14078871656381564
     -0.14078871656381564 0.8987982713097735])
    Trait3Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.31782233045956043 -0.01798395917118423
     -0.01798395917118423 0.1647429211651019],
    
    [0.6808712744794143 -0.11416573111891
     -0.11416573111891 0.8339228729012033])
    Trait3Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.3208883401712095 0.08452483760012779
     0.08452483760012779 0.0869867503137195],
    
    [0.6779139477283616 0.034013271781512075
     0.034013271781512075 0.9118411342566285])
    Trait3Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.3230087933057131 0.11068106827250586
     0.11068106827250586 0.06117389942330572],
    
    [0.6759011385391952 -0.007296623887800238
     -0.007296623887800238 0.9380722549963341])
    Trait4Trait5
    (Σa[i,j],Σe[i,j]) = (
    [0.2656669903604965 -0.21584756082780146
     -0.21584756082780146 0.2829186620095413],
    
    [0.730254403023407 -0.3766752828931733
     -0.3766752828931733 0.7151643527059122])
    Trait4Trait6
    (Σa[i,j],Σe[i,j]) = (
    [0.26614305300780366 -0.20063378204208704
     -0.20063378204208704 0.2844419499811047],
    
    [0.7297943215361493 -0.3468040727735064
     -0.3468040727735064 0.7151119490184537])
    Trait4Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.26448980293893115 -0.18275157344067408
     -0.18275157344067408 0.2141168002370858],
    
    [0.7314145610176318 -0.32617199552839854
     -0.32617199552839854 0.7829351279943724])
    Trait4Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.2666939542160246 -0.0976354011540246
     -0.0976354011540246 0.19612608720215985],
    
    [0.7292655955853755 -0.1503604822556339
     -0.1503604822556339 0.8035709853977226])
    Trait4Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.27003652025126973 -0.22740698731778375
     -0.22740698731778375 0.24804582217578136],
    
    [0.7260245141082378 -0.415601435658805
     -0.415601435658805 0.7502976667302643])
    Trait4Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.26554294182732757 -0.03381072154054875
     -0.03381072154054875 0.09960982635744982],
    
    [0.7303952879672779 -0.22772490049245936
     -0.22772490049245936 0.9002752447796257])
    Trait4Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.2656276022060441 -0.09674003324481623
     -0.09674003324481623 0.16327409085920214],
    
    [0.7303019079384457 -0.2726111957438671
     -0.2726111957438671 0.8353713025178962])
    Trait4Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.2681636965467903 -0.14161261172764802
     -0.14161261172764802 0.08039677342809182],
    
    [0.7278825933087102 -0.08284654588272211
     -0.08284654588272211 0.918445560967837])
    Trait4Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.2661712320522607 -0.0980730872961898
     -0.0980730872961898 0.05401019173275427],
    
    [0.729774933270783 -0.22505950767232102
     -0.22505950767232102 0.9452044744873448])
    Trait5Trait6
    (Σa[i,j],Σe[i,j]) = (
    [0.28159165289454874 0.2808983718818519
     0.2808983718818519 0.28229805042654726],
    
    [0.7164553609549211 0.6603676496785938
     0.6603676496785938 0.717195064496623])
    Trait5Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.28081437389240205 0.23209986889788878
     0.23209986889788878 0.21166204275984282],
    
    [0.7172180317169196 0.6743038172587597
     0.6743038172587597 0.7853426270989494])
    Trait5Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.28134012176468737 0.16394787427626617
     0.16394787427626617 0.19270331039605396],
    
    [0.7167009109174287 0.2210321091237623
     0.2210321091237623 0.8069220956477239])
    Trait5Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.2838778024838298 0.2445317256807396
     0.2445317256807396 0.24129303734021107],
    
    [0.7142441395397906 0.5084169870307738
     0.5084169870307738 0.7568942338389598])
    Trait5Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.2818130149574279 -0.04621141466301363
     -0.04621141466301363 0.10148069053449221],
    
    [0.7162383352815093 -0.057211134789489596
     -0.057211134789489596 0.8984242888573427])
    Trait5Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.2804235419822887 0.02024954557852043
     0.02024954557852043 0.16400332024748288],
    
    [0.7175856258526226 -0.03524364741892967
     -0.03524364741892967 0.8346493308285122])
    Trait5Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.28142706977012344 0.06161306621995005
     0.06161306621995005 0.08271662802160216],
    
    [0.7166145561768477 0.0529286449372972
     0.0529286449372972 0.9160739286176958])
    Trait5Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.2822915336764342 0.0704220516897222
     0.0704220516897222 0.05694630996696085],
    
    [0.7157823114676733 0.052837445512878035
     0.052837445512878035 0.9422684292567837])
    Trait6Trait7
    (Σa[i,j],Σe[i,j]) = (
    [0.2829610760516547 0.2206563210574195
     0.2206563210574195 0.21385609770082856],
    
    [0.7165486719537999 0.5810829183833723
     0.5810829183833723 0.7831785715716468])
    Trait6Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.2829606227312136 0.18407962646923373
     0.18407962646923373 0.19237902249802602],
    
    [0.7165491133467123 0.43659738936447606
     0.43659738936447606 0.8072460050938695])
    Trait6Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.28497844062058 0.23443573773381146
     0.23443573773381146 0.2432071080956827],
    
    [0.7146005305265446 0.4768263391945174
     0.4768263391945174 0.7550279957473227])
    Trait6Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.28365475689672454 -0.04354848104335164
     -0.04354848104335164 0.10202532157941907],
    
    [0.7158768651032459 -0.05916812562776467
     -0.05916812562776467 0.8978859540391582])
    Trait6Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.2815224136585793 0.027999198497751236
     0.027999198497751236 0.16342991310056218],
    
    [0.7179464769867658 -0.05241060710545059
     -0.05241060710545059 0.8352130875914755])
    Trait6Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.2831143444111233 0.05713989944443876
     0.05713989944443876 0.08267893130031188],
    
    [0.7164030333442727 0.047919871306752966
     0.047919871306752966 0.9161120298157116])
    Trait6Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.28381996715653646 0.06112120329706666
     0.06112120329706666 0.057081689702462565],
    
    [0.7157217792749953 0.05326983474671377
     0.05326983474671377 0.942133354542685])
    Trait7Trait8
    (Σa[i,j],Σe[i,j]) = (
    [0.213856863605251 0.08845552117653824
     0.08845552117653824 0.19237305507049685],
    
    [0.7831777634597645 -0.05683305954905118
     -0.05683305954905118 0.807251867488134])
    Trait7Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.2187556960284994 0.21704186383100332
     0.21704186383100332 0.24414386503619973],
    
    [0.7784327750440135 0.46290096824972443
     0.46290096824972443 0.7541229287932897])
    Trait7Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.2162729624537394 -0.04211457255557115
     -0.04211457255557115 0.10209048393658933],
    
    [0.7808073891161575 -0.08590745272581501
     -0.08590745272581501 0.8978220136457029])
    Trait7Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.21406877656928827 0.020696687538012654
     0.020696687538012654 0.16347380945331402],
    
    [0.7829608191265603 -0.04814801124367421
     -0.04814801124367421 0.8351704867213441])
    Trait7Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.21493232506988705 0.07578743223300372
     0.07578743223300372 0.08087309077189964],
    
    [0.7821316768251394 0.03469555448124276
     0.03469555448124276 0.9179149818424026])
    Trait7Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.21595864338557708 0.07493726652342743
     0.07493726652342743 0.05459341128356046],
    
    [0.7811387334979083 0.038905793649911785
     0.038905793649911785 0.9446215738065965])
    Trait8Trait9
    (Σa[i,j],Σe[i,j]) = (
    [0.1945549993500238 0.11281615770315953
     0.11281615770315953 0.24724415191290416],
    
    [0.8051240570336379 0.18477843243741904
     0.18477843243741904 0.7510982278179416])
    Trait8Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.19444100261213867 -0.015634153642008396
     -0.015634153642008396 0.10042647889284885],
    
    [0.8052215712386805 0.011982589661319697
     0.011982589661319697 0.8994678453453964])
    Trait8Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.1938531643383019 0.02253246401818133
     0.02253246401818133 0.1646685438304724],
    
    [0.8057962945716937 -0.02727473686745877
     -0.02727473686745877 0.8339969682996413])
    Trait8Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.19395121849291824 -0.0028760121862793694
     -0.0028760121862793694 0.08285727475931896],
    
    [0.8056997963655848 0.03361278793070539
     0.03361278793070539 0.9159318672878024])
    Trait8Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.19397976155497032 0.00407866657222457
     0.00407866657222457 0.0569071232157577],
    
    [0.8056716012394011 0.03788170752125487
     0.03788170752125487 0.9423010776566965])
    Trait9Trait10
    (Σa[i,j],Σe[i,j]) = (
    [0.24729383232785143 -0.0023083632331894533
     -0.0023083632331894533 0.09982638783372684],
    
    [0.7510505981833204 0.07407294365773788
     0.07407294365773788 0.9000613327603086])
    Trait9Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.24782344372317563 0.031823502985820915
     0.031823502985820915 0.1648904641109765],
    
    [0.7505321390736757 0.15228537870002187
     0.15228537870002187 0.83377795534806])
    Trait9Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.25033469514724843 0.08457136153430893
     0.08457136153430893 0.08875872340836785],
    
    [0.7480909649333196 0.10775632151728669
     0.10775632151728669 0.910108091343637])
    Trait9Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.24944189418598844 0.09348451303745715
     0.09348451303745715 0.05793201493993036],
    
    [0.7489745640206875 0.09821909822628513
     0.09821909822628513 0.9413348855487351])
    Trait10Trait11
    (Σa[i,j],Σe[i,j]) = (
    [0.09313966827485977 0.10003371877328632
     0.10003371877328632 0.16495492788178234],
    
    [0.906703444773408 0.4744266593662351
     0.4744266593662351 0.8337151519529007])
    Trait10Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.09672471643798064 0.05640434659756219
     0.05640434659756219 0.07945282707191781],
    
    [0.9031497841220121 0.08532319187443138
     0.08532319187443138 0.9193343434315169])
    Trait10Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.10098492171842248 -0.027991587712823545
     -0.027991587712823545 0.0578369411328611],
    
    [0.8989368215861225 0.16605077488864645
     0.16605077488864645 0.9413901799271158])
    Trait11Trait12
    (Σa[i,j],Σe[i,j]) = (
    [0.16384057742532637 0.05703178017185482
     0.05703178017185482 0.07921436807488584],
    
    [0.8348140972542764 0.14559650974208901
     0.14559650974208901 0.9195521322078624])
    Trait11Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.1648829333720086 -0.0015841372105068166
     -0.0015841372105068166 0.05749684374690435],
    
    [0.8337979488921528 0.20061222835020495
     0.20061222835020495 0.9417152321030534])
    Trait12Trait13
    (Σa[i,j],Σe[i,j]) = (
    [0.08459459620614283 0.06850521854233418
     0.06850521854233418 0.05547594264883066],
    
    [0.9142140604502765 0.573152408940862
     0.573152408940862 0.9437349886346289])
     98.060194 seconds (2.33 G allocations: 35.661 GB, 7.59% gc time)


## 3-trait analysis

Researchers want to jointly analyze traits 5-7. Our strategy is to try both Fisher scoring and MM algorithm with different starting point, and choose the best local optimum. We first form the data set and run Fisher scoring, which yields a final objective value -1.4700991+04.


```julia
traitidx = 5:7
# form data set
trait57_data = TwoVarCompVariateRotate(cg10kdata_rotated.Yrot[:, traitidx], cg10kdata_rotated.Xrot, 
    cg10kdata_rotated.eigval, cg10kdata_rotated.logdetV2)
# initialize model parameters
trait57_model = VarianceComponentModel(trait57_data)
# estimate variance components
@time mle_fs!(trait57_model, trait57_data; solver=:Ipopt, verbose=true)
trait57_model
```

    This is Ipopt version 3.12.4, running with linear solver mumps.
    NOTE: Other linear solvers might be more efficient (see Ipopt documentation).
    
    Number of nonzeros in equality constraint Jacobian...:        0
    Number of nonzeros in inequality constraint Jacobian.:        0
    Number of nonzeros in Lagrangian Hessian.............:       78
    
    Total number of variables............................:       12
                         variables with only lower bounds:        0
                    variables with lower and upper bounds:        0
                         variables with only upper bounds:        0
    Total number of equality constraints.................:        0
    Total number of inequality constraints...............:        0
            inequality constraints with only lower bounds:        0
       inequality constraints with lower and upper bounds:        0
            inequality constraints with only upper bounds:        0
    
    iter    objective    inf_pr   inf_du lg(mu)  ||d||  lg(rg) alpha_du alpha_pr  ls
       0  3.0247512e+04 0.00e+00 1.00e+02   0.0 0.00e+00    -  0.00e+00 0.00e+00   0 
       5  1.6834796e+04 0.00e+00 4.07e+02 -11.0 3.66e-01    -  1.00e+00 1.00e+00f  1 MaxS
      10  1.4744497e+04 0.00e+00 1.12e+02 -11.0 2.45e-01    -  1.00e+00 1.00e+00f  1 MaxS
      15  1.4701497e+04 0.00e+00 1.30e+01 -11.0 1.15e-01  -4.5 1.00e+00 1.00e+00f  1 MaxS
      20  1.4700992e+04 0.00e+00 6.65e-01 -11.0 1.74e-04  -6.9 1.00e+00 1.00e+00f  1 MaxS
      25  1.4700991e+04 0.00e+00 2.77e-02 -11.0 7.36e-06  -9.2 1.00e+00 1.00e+00f  1 MaxS
      30  1.4700991e+04 0.00e+00 1.15e-03 -11.0 3.06e-07 -11.6 1.00e+00 1.00e+00f  1 MaxS
      35  1.4700991e+04 0.00e+00 4.76e-05 -11.0 1.27e-08 -14.0 1.00e+00 1.00e+00h  1 MaxS
      40  1.4700991e+04 0.00e+00 1.97e-06 -11.0 5.26e-10 -16.4 1.00e+00 1.00e+00h  1 MaxSA
      45  1.4700991e+04 0.00e+00 8.17e-08 -11.0 2.18e-11 -18.8 1.00e+00 1.00e+00h  1 MaxSA
    iter    objective    inf_pr   inf_du lg(mu)  ||d||  lg(rg) alpha_du alpha_pr  ls
    
    Number of Iterations....: 49
    
                                       (scaled)                 (unscaled)
    Objective...............:   4.4724330090668150e+02    1.4700991028593420e+04
    Dual infeasibility......:   6.4551211049368979e-09    2.1218132783605816e-07
    Constraint violation....:   0.0000000000000000e+00    0.0000000000000000e+00
    Complementarity.........:   0.0000000000000000e+00    0.0000000000000000e+00
    Overall NLP error.......:   6.4551211049368979e-09    2.1218132783605816e-07
    
    
    Number of objective function evaluations             = 50
    Number of objective gradient evaluations             = 50
    Number of equality constraint evaluations            = 0
    Number of inequality constraint evaluations          = 0
    Number of equality constraint Jacobian evaluations   = 0
    Number of inequality constraint Jacobian evaluations = 0
    Number of Lagrangian Hessian evaluations             = 49
    Total CPU secs in IPOPT (w/o function evaluations)   =      0.022
    Total CPU secs in NLP function evaluations           =      3.613
    
    EXIT: Optimal Solution Found.
      3.714206 seconds (93.11 M allocations: 1.410 GB, 7.39% gc time)





    VarianceComponentModels.VarianceComponentModel{Float64,2,Array{Float64,2},Array{Float64,2}}(0x3 Array{Float64,2},(
    3x3 Array{Float64,2}:
     0.281163  0.280014  0.232384
     0.280014  0.284899  0.220285
     0.232384  0.220285  0.212687,
    
    3x3 Array{Float64,2}:
     0.716875  0.66125   0.674025
     0.66125   0.714602  0.581433
     0.674025  0.581433  0.784324),0x0 Array{Float64,2},Char[],Float64[],-Inf,Inf)



We then run the MM algorithm, starting from the Fisher scoring answer. MM finds an improved solution with objective value 8.955397e+03.


```julia
# trait59_model contains the fitted model by Fisher scoring now
@time mle_mm!(trait57_model, trait57_data; verbose=true)
trait57_model
```

    
         MM Algorithm
      Iter      Objective  
    --------  -------------
           0  -1.470099e+04
           1  -1.470099e+04
    
      0.083887 seconds (2.06 M allocations: 35.243 MB, 5.32% gc time)





    VarianceComponentModels.VarianceComponentModel{Float64,2,Array{Float64,2},Array{Float64,2}}(0x3 Array{Float64,2},(
    3x3 Array{Float64,2}:
     0.281163  0.280014  0.232384
     0.280014  0.284899  0.220285
     0.232384  0.220285  0.212687,
    
    3x3 Array{Float64,2}:
     0.716875  0.66125   0.674025
     0.66125   0.714602  0.581433
     0.674025  0.581433  0.784324),0x0 Array{Float64,2},Char[],Float64[],-Inf,Inf)



Do another run of MM algorithm from default starting point. It leads to a slightly better local optimum -1.470104e+04, slighly worse than the Fisher scoring result. Follow up anlaysis should use the Fisher scoring result.


```julia
# default starting point
trait57_model = VarianceComponentModel(trait57_data)
@time _, _, _, Σcov, = mle_mm!(trait57_model, trait57_data; verbose=true)
trait57_model
```

    
         MM Algorithm
      Iter      Objective  
    --------  -------------
           0  -3.024751e+04
           1  -2.040338e+04
           2  -1.656127e+04
           3  -1.528591e+04
           4  -1.491049e+04
           5  -1.480699e+04
           6  -1.477870e+04
           7  -1.477026e+04
           8  -1.476696e+04
           9  -1.476499e+04
          10  -1.476339e+04
          20  -1.475040e+04
          30  -1.474042e+04
          40  -1.473272e+04
          50  -1.472677e+04
          60  -1.472215e+04
          70  -1.471852e+04
          80  -1.471565e+04
          90  -1.471336e+04
         100  -1.471152e+04
         110  -1.471002e+04
         120  -1.470879e+04
         130  -1.470778e+04
         140  -1.470694e+04
         150  -1.470623e+04
         160  -1.470563e+04
         170  -1.470513e+04
         180  -1.470469e+04
         190  -1.470432e+04
         200  -1.470400e+04
         210  -1.470372e+04
         220  -1.470347e+04
         230  -1.470326e+04
         240  -1.470307e+04
         250  -1.470290e+04
         260  -1.470275e+04
         270  -1.470262e+04
         280  -1.470250e+04
         290  -1.470239e+04
         300  -1.470229e+04
         310  -1.470220e+04
         320  -1.470213e+04
         330  -1.470205e+04
         340  -1.470199e+04
         350  -1.470193e+04
         360  -1.470187e+04
         370  -1.470182e+04
         380  -1.470177e+04
         390  -1.470173e+04
         400  -1.470169e+04
         410  -1.470165e+04
         420  -1.470162e+04
         430  -1.470159e+04
         440  -1.470156e+04
         450  -1.470153e+04
         460  -1.470150e+04
         470  -1.470148e+04
         480  -1.470146e+04
         490  -1.470143e+04
         500  -1.470141e+04
         510  -1.470140e+04
         520  -1.470138e+04
         530  -1.470136e+04
         540  -1.470134e+04
         550  -1.470133e+04
         560  -1.470132e+04
         570  -1.470130e+04
         580  -1.470129e+04
         590  -1.470128e+04
         600  -1.470127e+04
         610  -1.470125e+04
         620  -1.470124e+04
         630  -1.470123e+04
         640  -1.470122e+04
         650  -1.470122e+04
         660  -1.470121e+04
         670  -1.470120e+04
         680  -1.470119e+04
         690  -1.470118e+04
         700  -1.470118e+04
         710  -1.470117e+04
         720  -1.470116e+04
         730  -1.470116e+04
         740  -1.470115e+04
         750  -1.470114e+04
         760  -1.470114e+04
         770  -1.470113e+04
         780  -1.470113e+04
         790  -1.470112e+04
         800  -1.470112e+04
         810  -1.470111e+04
         820  -1.470111e+04
         830  -1.470111e+04
         840  -1.470110e+04
         850  -1.470110e+04
         860  -1.470109e+04
         870  -1.470109e+04
         880  -1.470109e+04
         890  -1.470108e+04
         900  -1.470108e+04
         910  -1.470108e+04
         920  -1.470108e+04
         930  -1.470107e+04
         940  -1.470107e+04
         950  -1.470107e+04
         960  -1.470106e+04
         970  -1.470106e+04
         980  -1.470106e+04
         990  -1.470106e+04
        1000  -1.470106e+04
        1010  -1.470105e+04
        1020  -1.470105e+04
        1030  -1.470105e+04
        1040  -1.470105e+04
        1050  -1.470105e+04
        1060  -1.470104e+04
        1070  -1.470104e+04
        1080  -1.470104e+04
        1090  -1.470104e+04
        1100  -1.470104e+04
        1110  -1.470104e+04
    
      9.687002 seconds (224.30 M allocations: 5.566 GB, 13.29% gc time)





    VarianceComponentModels.VarianceComponentModel{Float64,2,Array{Float64,2},Array{Float64,2}}(0x3 Array{Float64,2},(
    3x3 Array{Float64,2}:
     0.281188  0.280032  0.232439
     0.280032  0.284979  0.220432
     0.232439  0.220432  0.212922,
    
    3x3 Array{Float64,2}:
     0.71685   0.661232  0.67397 
     0.661232  0.71452   0.581287
     0.67397   0.581287  0.784092),0x0 Array{Float64,2},Char[],Float64[],-Inf,Inf)



Heritability from 3-variate estimate and their standard errors.


```julia
h, hse = heritability(trait57_model.Σ, Σcov)
[h'; hse']
```




    2x3 Array{Float64,2}:
     0.281741   0.285122   0.21356  
     0.0778033  0.0773313  0.0841103



## Save analysis results


```julia
#using JLD
#@save "copd.jld"
#whos()
```