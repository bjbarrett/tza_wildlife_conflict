---
title: "tza_dags_section"
author: "Brendan Barrett"
date: "5/28/2021"
output: pdf_document
---



# Causal Inference and DAGs

A primary aim of the scientific enterprise is to infer causal effects of predictors on outcome variables of inference, to increase our understanding of how systems function. 
This also can help folks working in applied contexts  such as mitigating human-wildlife conflict mmake informed interventions. 
Well-designed experiments are one typical approach to understand causality, but in many cases, like the study presented in this paper, experiments would be not feasible or unethical. 

Many common approaches in statistical inference, such as multivariate regression, do not make any claims about causality, and statistical information flows directly between outcome variable and predictors. 
Researchers are often concerned about the effect of predictor, X, on an outcome variable, Y.  
However, X may be correlated with another covariate(s) of interest, Z, which can confound the relationship between X and Y.
To infer the relationship between X and Y, researchers will often add covariates like Z (and often times many others) to control for potential covariates. A common term in ecology papers is to ''control for seasonality'' or ''control for environmental effects.''

Confounding factors are a real, and valid concern, but whether or not to include, or exclude, a variable in a multivariate regression depends on the causal relationships between measureable variables of interest, and any potential unobserved variables. In some cases, including  covariate Z can reduce the precision of an estimate of the effect of X on Y or render it entirely unreliable if Z is a collider (where X and Y both cause Z).

## What is a DAG
DAGs (directed acyclical graphs) and are a common tool in causal inference, a topic seperate from, but related to statistical inference. Generalized linear models do not imply the direction of causality as information in both directions between variables of interst.
DAGs imply the direction of causality. 
DAGs are common in field like epidemiology, but are increasingly common in the social and biological sciences (CITE). 
By proposing a DAG about the causal relationships between predictors of importance and outcomes in our study systems DAGs can help us understand:

1. which confounding variables to include in a regression when we wish to make a claim about the causal relationship between X → Y. In causal inference, this is known as closing the *backdoor path*.
2. which covariates to exclude from our analysis, as including them will introduce a confound. A common example of this is *collider bias*.
3. whether or not reliable inferences about the causal relationship between X and Y is even possible.

Other advantages of DAGs are that they force researchers to be explicit about causal relationships and think carefully about their study system.
Does X directly cause Y? 
Or, does X also cause Z which causes Y? 
Perhaps X causes Z, which is also caused by Y?
The answer to these questions informs us what to include or exclude in our statistical model.
Our experience is that researchers often will say X causes Y, when in reality there is a middle step that is implied or ignored.
Researchers can use their knowledge of their study systems to propose a DAG or DAGs, and they should justify the thinking bejind each direct causal arrow.
Assuming a DAG is true, we can use it to inform which regressions we run to make the most reliable inferences about the effect of X on Y.
A critic of research may also make propose a different DAG, which might suggest that a different analysis should be run, or that the question may not be reliably answered at all. 

# Drawing a DAG

To draw a DAG, we first consider all of the variable of interest in the system. We typically want to known the effect of a treatment/predictor/exposure on an outcome variable. If we think X, our predictor, directly causes Y, we draw an arrow from X to Y

This arrow implies a direct causal relationship between X and Y. Something has a causal relationship if the natural process determining Y is *directly influenced* by the status of X. However, an arrow X → Y only represents the part of the causal effect that is not mediated by any of the other variables in the DAG. If one is sure X does not directly mediate Y, an arrow can be excluded. One must also ensure that causes come before effects, and X preceeds Y. In instances where this is not the case, and there are bidirectional arrows between X and Y we violate this assumption and need an experiment or time series of treatments on outcomes.

## Crop Wildlife Conflict DAG

Below is a DAG for understanding what causes crop conflict in the Greater Grumeti Ecosystem Area.


```r
crop_damage_dag <- 
  dagitty('dag {
  c2070 -> crop_damage
  c70 -> crop_damage
  river -> c70
  river -> crop_damage
  months_planted -> crop_damage
  farm_size -> crop_damage
  farm_size -> num_protect
  num_protect -> crop_damage
  crop_damage -> num_protect
  hh_size -> num_protect
  hh_size -> farm_size
  hh_size -> crop_damage
  see_field -> crop_damage
  road <-> bd
  bd -> crop_damage
  bd -> c2070
  bd -> c70
  sd -> bd
  sd -> crop_damage
  cd -> c70
  cd -> c2070
  cd -> crop_damage
  slope -> bd 
  slope -> crop_damage
  slope -> c2070 
  slope -> river 
  slope -> road
  slope -> cd
  }')

plot(crop_damage_dag)
```

```
## Plot coordinates for graph not supplied! Generating coordinates, see ?coordinates for how to set your own.
```

![plot of chunk crop damage dag](figure/crop damage dag-1.png)

### Reasoning for direct causal effects
1. c2070 -> crop_damage: c2070 is habitat refuge for crop raiders. More habitat could mean there are more places to hide, or less habitat could mean that they are forced to raid crops more. 
2. c70 -> crop_damage: c70 is habitat refuge for crop raiders. More habitat could mean there are more places to hide, or less habitat could mean that they are forced to raid crops more. 
3. river -> c70: the presence of water in rivers creates conditions where c70 classified lands grow.
4. river -> crop_damage: Animals dwell near rivers, and are likely to cause conflict at places near them as a consequence.
5. months_planted -> crop_damage : The more time there are crops in the field, the more likely conflict will be observed,
6. farm_size -> crop_damage: That larger the farm, the more available crops are, and the more likely they will get damaged.
7. farm_size -> num_protect #farm size influences the type of protection, which influences the number of strategies used. this is really indirect.
8. num_protect -> crop_damage: Using a range of strategies may reduce crop raiding.
9. crop_damage -> num_protect: desperate farmers with crop damage may try lots of new crop strategies out of desperation.
10. hh_size -> num_protect: larger households engage more effort in protection 
11. hh_size -> farm_size:more available people to work fields, makes it possible to have a larger farm
12. hh_size -> crop_damage: Animals avoid field with more human activity
13. farm_size -> num_protect: larger farms employ more protection strategies, particularly things like fences etc. that do not require folks standing there 
14. see_field -> crop_damage: farmers that see their field can react quickly and minimize damage
15. road -> bd: People will build settlements along roads due to access. It is less certain that building density also causes roads, but possible tertiary roads and smaller roads get built to connect dense places. However, the layer we used to estimate raod densitymeasure if primary roads
16. bd -> crop_damage: building density attracts and deters different crop raiders (i.e. vervets vs. elephants)
17. bd -> c2070: construction of buildings causes loss in c2070 and changes classification probability
18. bd -> c70: construction of buildings causes loss in c70 and changes classification probability
19. sd -> bd: Cities expand toward protected areas, less dense at edges. 500m buffer & preservation mean that density is lower right next to PA.
20. sd -> crop_damage: Different animals have different risk tolerances, some venture far from PA.
21. cd -> c70: increased crop density and land conversion means there is less likely to be c70.
22. cd -> c2070: increased crop density and land conversion means there is less likely to be c2070.
23. cd -> crop_damage: more beneficial to raid areas with a higher density of crops.
24. slope -> bd: More houses are built on less hilly land for ease of construction and material transport.
25. slope -> crop_damage: elephants don't like traveling on hills, so less likely to damage farms on slopes.
26. slope -> c2070: 2070 is more likely on hillsides either due to the difficulty require in cutting it down, or ecological conditions.
27. slope -> river: water flows down hills and is likely to be in places with decreasing slopes. 
28. slope -> road:  slope influences where roads are built, the are preferentially built in easier, less hilly places and lower mountain passes.
29. slope -> cd: crops are more densley planted in flat areas (less runoff, easier to plant things close together)

Using the `dagitty` pacakge in `R` we can use the `adjustmentSets` function to help us understand what are the minimal number of covariates we need to include in a model to reliably estimate the effect of a predictor on crop raiding.

Now we can look at all of the direct arrows to estimate the effect of X on Y, and determine which covariates to include in the models relevant to the predictor of interest.

For c2070 the minimal model `mc_c2070_min` includes: 

```r
 adjustmentSets( crop_damage_dag , exposure="c2070" , outcome="crop_damage" ) 
```

```
## { bd, cd, slope }
```
while the canonical model, `mc_c70_c2070_can` includes:

```r
 adjustmentSets( crop_damage_dag , exposure="c2070" , outcome="crop_damage" ) 
```

```
## { bd, cd, slope }
```

For c70 the minimal model `mc_c70_min` includes:

```r
adjustmentSets( crop_damage_dag , exposure="c70" , outcome="crop_damage" ) 
```

```
## { bd, cd, river }
```

For c70 the canonical model `mc_c70_c2070_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="c70" , outcome="crop_damage" ) 
```

```
## { bd, cd, river }
```


For cd the minimal model `mc_cd_min` includes:

```r
adjustmentSets( crop_damage_dag , exposure="cd" , outcome="crop_damage" ) 
```

```
## { slope }
```

For cd the canonical model `mc_cd_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="cd" , outcome="crop_damage" ) 
```

```
## { slope }
```

For river the minimal model `mc_riv_min` includes:

```r
adjustmentSets( crop_damage_dag , exposure="river" , outcome="crop_damage" )
```

```
## { slope }
```

For river the minimal model `mc_riv_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="river" , outcome="crop_damage" )
```

```
## { slope }
```

For settlement distance the minimal model `mc_sd_min` includes:

```r
adjustmentSets( crop_damage_dag , exposure="sd" , outcome="crop_damage" )
```

```
##  {}
```
It requires no other covariates.

For settlement distance the canonical model `mc_sd_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="sd" , outcome="crop_damage" )
```

```
##  {}
```

For building density the minimal model `mc_bd_min` includes:

```r
adjustmentSets( crop_damage_dag , exposure="bd" , outcome="crop_damage" )
```

```
## { sd, slope }
```

For building density the canonical model `mc_sd_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="bd" , outcome="crop_damage" )
```

```
## { sd, slope }
```

For months planted the minimal model `mc_mp_min` includes:

```r
adjustmentSets( crop_damage_dag , exposure="months_planted" , outcome="crop_damage" )
```

```
##  {}
```

For months planted the canonical model `mc_fs_mp_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="months_planted" , outcome="crop_damage" )
```

```
##  {}
```

For see field the minimal model `mc_see_min` includes:

```r
adjustmentSets( crop_damage_dag , exposure="see_field" , outcome="crop_damage" )
```

```
##  {}
```

For see field the canonical model `mc_see_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="see_field" , outcome="crop_damage" )
```

```
##  {}
```

For number of protection strategies, we cannot reliably make an inference conditional on this DAG being true:

```r
adjustmentSets( crop_damage_dag , exposure="num_protect" , outcome="crop_damage" )
```

For number of protection strategies the canonical model `m_np_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="num_protect" , outcome="crop_damage" )
```

For household size the minimal model `m_hhs_min` includes:

```r
adjustmentSets( crop_damage_dag , exposure="hh_size" , outcome="crop_damage" )
```

```
##  {}
```

For household size the canonical model `m_hhs_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="hh_size" , outcome="crop_damage" )
```

```
##  {}
```

For farm size the minimal model `m_fs_min` includes:

```r
adjustmentSets( crop_damage_dag , exposure="farm_size" , outcome="crop_damage" )
```

```
## { hh_size }
```

For farm size the canonical model `m_fs_mp_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="farm_size" , outcome="crop_damage" )
```

```
## { hh_size }
```

For slope the minimal model `m_slope_min` includes:

```r
adjustmentSets( crop_damage_dag , exposure="slope" , outcome="crop_damage" )
```

```
##  {}
```

For slope the canonical model `m_slope_can` includes:

```r
adjustmentSets( crop_damage_dag , exposure="slope" , outcome="crop_damage")
```

```
##  {}
```

Note that c2070 and c70 have the same canonical model. 
Months planted and farm size have the same canonical model. 
Importantly, assuming the DAG is true, we cannot estimate the effectiveness of the number of protection strategies on crop damage given our current data. 
We need a time series or an experimental intervention to measure conflict rates before and after an intervention

## Livestock Carnivore Conflict DAGs

```r
ls_conf_yes_guard <- 
  dagitty('dag {
  c2070 -> conflict
  bd -> conflict 
  bd <-> road 
  c70 -> conflict 
  hh_size -> guards 
  hh_size -> lsh 
  lsh -> conflict 
  river -> c70 
  river -> conflict 
  sd -> bd 
  sd -> conflict
  bd -> c70
  bd -> c2070
  guards <-> conflict 
  slope -> bd 
  slope -> conflict
  slope -> c2070 
  slope -> river 
  slope -> road 
}')

plot(ls_conf_yes_guard)
```

```
## Plot coordinates for graph not supplied! Generating coordinates, see ?coordinates for how to set your own.
```

![plot of chunk livestock dag yes double](figure/livestock dag yes double-1.png)

Our direct causal relationships are justified as follows:
1. c2070 -> conflict: c2070 is refuge habitat for carnivores who live there more and use c2070 as refuge to avoid detection and forage at night.
2. bd -> conflict: building density signifies human presence, and carnivores avoid dense areas. 
3. bd <-> road: People will build settlements along roads due to access.
4. c70 -> conflict: c70 is refuge habitat for predators, i.e. riparian zone and dense thickets. Loss of c70, causes loss of habitat and can increase conflict.
5. hh_size -> lsh: Larger households are often multi-generational, which means they have more capital which is often invested in cattle
6. hh_size -> guards: The more people in the house, the more there are available to act as guards.
7. lsh -> conflict: The greater number of cattle there are to eat, the more likely they will be predated. 
8. river -> c70: Rivers are habitat for vegetation communities classified as c2070.
9. river -> conflict: Animals need water to live, may raid more when less water available.
10. sd -> bd: laws in place to prevent people from buiding within a 500m buffer of reserve, so they build elsewhere. People prefer to build away from PA, and really only do so out of necessity
11. sd -> conflict: some species prefer to be in PA, and are only likely raid farms close to boundaries. Others venture out to raid farms.
12. bd -> c70: construction of buildings causes loss in c70 and changes classification probability
13. bd -> c2070: construction of buildings causes loss in c2070 and changes classification probability
14. guards <-> conflict: Guards in theory reduce conflict if effective. That is their point. However, due to conflict, farmers may be more inclined to hire guards. To break this bidirectional arrow, one could randomly apply numbers of guards to people's herds, prevent them from changing it, and measure conflict. However, this is unethical. Instead, one would need to measure conflict levels, or amount of crop loss, as a function of the number of guards introduced, or used at each timestep.
15. slope -> bd: More houses are built on less hilly land for ease of construction and material transport.
16. slope -> conflict: Some predators travel in hillier areas
18. slope -> rivers: water flows down hills and is likely to be in places with decreasing slopes. 
19. slope -> road:  slope influences where roads are built, the are preferentially built in easier, less hilly places and lower mountain passes.

Now we can run the adjustment sets. 

For c2070 the minimal model `ml_c2070_min` includes:

```r
adjustmentSets( ls_conf_yes_guard , exposure="c2070" , outcome="conflict" , type="minimal")
```

```
## { bd, slope }
```
For c70 the minimal model `ml_c70_min` includes:

```r
adjustmentSets( ls_conf_yes_guard , exposure="c70" , outcome="conflict" , type="minimal")
```

```
## { bd, river }
```
For number of livestock head the minimal model `ml_lsh_min` includes:

```r
adjustmentSets( ls_conf_yes_guard , exposure="lsh" , outcome="conflict" , type="minimal")
```

```
##  {}
```
For river density the minimal model `ml_riv_min` includes:

```r
adjustmentSets( ls_conf_yes_guard , exposure="river" , outcome="conflict" , type="minimal")
```

```
## { slope }
```
For distance from settlement edge the minimal model `ml_sd_min` includes:

```r
adjustmentSets( ls_conf_yes_guard , exposure="sd" , outcome="conflict" , type="minimal")
```

```
##  {}
```
For building density the minimal model `ml_bd_min` includes:

```r
adjustmentSets( ls_conf_yes_guard , exposure="bd" , outcome="conflict" , type="minimal")
```

```
## { sd, slope }
```
For slope the minimal model `ml_sl_min` includes:

```r
adjustmentSets( ls_conf_yes_guard , exposure="slope" , outcome="conflict" )
```

```
##  {}
```
For number of guards, the minimal model `ml_guards_min` includes:

```r
adjustmentSets( ls_conf_yes_guard , exposure="slope" , outcome="conflict" )
```

```
##  {}
```

Note the last adjustment set. There is no output. That is because we can't make any reliable inference about guards affecting conflict given our DAG. We need to design data collection or a study where this is a single arrow, or a different dag is implied. Double arrows typically mean we need to break apart the timescale. Guards cause conflict in that they in theory reduce it. Conflict causes guards because people may get more guards if they have conflict. We need data that separates these two.

### Alternative DAGs without Bidirectional Arrows.

```r
ls_conf_yes_guard2 <- 
  dagitty('dag {
  c2070 -> conflict
  bd -> conflict 
  bd <-> road 
  c70 -> conflict 
  hh_size -> guards 
  hh_size -> lsh 
  lsh -> conflict 
  river -> c70 
  river -> conflict 
  sd -> bd 
  sd -> conflict
  bd -> c70
  bd -> c2070
  guards -> conflict 
  slope -> bd 
  slope -> conflict
  slope -> c2070 
  slope -> river 
  slope -> road 
}')

plot(ls_conf_yes_guard2)
```

```
## Plot coordinates for graph not supplied! Generating coordinates, see ?coordinates for how to set your own.
```

![plot of chunk livestock dag no double](figure/livestock dag no double-1.png)
For number of guards, the minimal model `ml_guards_min` would include:

```r
adjustmentSets( ls_conf_yes_guard2 , exposure="guards" , outcome="conflict" )
```

```
## { lsh }
## { hh_size }
```

For number of guards, the minimal model `ml_lsh_min` would include:

```r
adjustmentSets( ls_conf_yes_guard2 , exposure="lsh" , outcome="conflict" )
```

```
## { guards }
## { hh_size }
```

```r
crop_damage_dag2 <- 
  dagitty('dag {
  c2070 -> crop_damage
  c70 -> crop_damage
  river -> c70
  river -> crop_damage
  months_planted -> crop_damage
  farm_size -> crop_damage
  farm_size -> num_protect
  num_protect -> crop_damage
  hh_size -> num_protect
  hh_size -> farm_size
  hh_size -> crop_damage
  see_field -> crop_damage
  road <-> bd
  bd -> crop_damage
  bd -> c2070
  bd -> c70
  sd -> bd
  sd -> crop_damage
  cd -> c70
  cd -> c2070
  cd -> crop_damage
  slope -> bd 
  slope -> crop_damage
  slope -> c2070 
  slope -> river 
  slope -> road
  slope -> cd
  farm_size -> crop_prot_music -> crop_damage        
  farm_size -> crop_prot_w_fence -> crop_damage            
  farm_size -> crop_prot_sisal -> crop_damage              
  farm_size -> crop_prot_shout -> crop_damage             
  farm_size -> crop_prot_fire -> crop_damage              
  hh_size -> crop_prot_chase -> crop_damage             
  hh_size -> crop_prot_guard -> crop_damage       
  farm_size -> crop_prot_guard -> crop_damage     
  farm_size -> crop_prot_chase -> crop_damage             
  }')

plot(crop_damage_dag2)
```

```
## Plot coordinates for graph not supplied! Generating coordinates, see ?coordinates for how to set your own.
```

![plot of chunk crop damage dag 2](figure/crop damage dag 2-1.png)
For number of guards, the minimal model `ml_np_min` would include:

```r
adjustmentSets( crop_damage_dag2 , exposure="num_protect" , outcome="crop_damage" )
```

```
## { farm_size, hh_size }
```
For used of wire fences, the minimal model `ml_cpwf_min` would include:

```r
adjustmentSets( crop_damage_dag2 , exposure="crop_prot_w_fence" , outcome="crop_damage" )
```

```
## { farm_size }
```

rmarkdown::render("tza_dags_section.Rmd", output_format = latex_document())
