#######################################################################
# Intro
#######################################################################

The following folder contains the reproducible data and software code for the study entitled "Weather conditions are systematically associated with nonroutine long-range movements in a large scavenger". The folder is divided in a series of datasets and software codes.

#######################################################################
# Datasets
#######################################################################

The dataset "Dataset_GAMM_20250807.Rdata" contains the processed data, ready to be used in the final GAMM aimed to link weather conditions to the probability of short, medium and long-range movement. Rows (observational units) correspond to GPS fix, acquired by GPS tags. 
The dataset contains the following columns:
- id -> The identifier of each one of the 20 individuals
- timestamp -> The timestamp of each GPS location
- lon.utm -> Longitude of each GPS location in epsg: 32632
- lat.utm -> Latitude of each GPS location in epsg: 32632
- difftime -> Time elapsed between each location and the previous one
- id.ehrm -> A label indicating to which extra-home-range-movement each GPS location belongs to
- rank.ehrm -> A label indicating the order of each GPS location in a certain extra-home range movement
- cluster -> A label, from PAM cluster analysis, indicating the group to which each extra-home range movement trajectory belonged to. The variable has the following values:
	-> 0 for GPS fix within the home range (short-range movements)
	-> 1, 2, 3 for GPS locations belonging to short movements outside the home range (medium-range movements)
	-> 4 for GPS locations belonging to long movements outside the home range (long-range movements)

- rel.type -> A label indicating whether individuals were subjected to short ("SA") or long acclimatization ("LA") before being released. For further details please see https://doi.org/10.1016/j.isci.2023.106699
- date -> The date of each GPS locations
- date.born -> The date when each individuals was deemed to be born. For further details please see https://doi.org/10.1016/j.isci.2023.106699
- days.age -> The age of each animal, when each GPS locations was collected, obtained by differencing "date" and "date.born"
- days.age.std -> The age of each animal, converted to a Z-score (https://en.wikipedia.org/wiki/Standard_score)
- yday -> The day of each location, on each year. From 1 to 366.
- yday.std -> The day of each location, converted to a Z-score
- behav -> the type of movement, obtained from the "cluster" variable. As an integer, to be used with the ocat family in mgcv (see https://stat.ethz.ch/R-manual/R-devel/library/mgcv/html/ocat.html for a complete description)
	-> 1 corresponds to short-range movements (cluster == 1)
	-> 2 corresponds to medium-range movements (cluster %in% c(2, 3, 4))
	-> 3 corresponds to long-range movements (cluster == 4)
- wind.str -> Wind strength, in m/s. Wind strength was calculated from the u and v component of the wind at an elevation of 100 m, downloaded from ERA5 (https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=overview). With the formula sqrt(u^2 + v^2)
- wind.str.std -> Wind strength, converted to a Z-score.
- wind.dir -> Wind direction, expressed in degrees from North (from 0 to 360). Wind direction was calculated from the u and v component of the wind at an elevation of 100 m, downloaded from ERA5 (https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=overview). With the formula (atan2(u/d$wind.str, v/d$wind.str)*(180/pi)) + 180
- wind.dir.std -> Wind direction, converted to a Z-score
- sol.rad -> Solar radiation, expressed as the mean surface downward short-wave radiation flux, downloaded from ERA5 (https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=overview).
- sol.rad.std <- Solar radiation, converted to a Z-score

The dataset "eHRm_20250807.Rdata", containing information about single extra-home range movements (EHRMs), identified by overlapping trajectories over home-ranges. Each row in this dataset corresponds to a track. The dataset eHRm_20250807.Rdata contains the following columns:
- id.ehrm -> A label indicating to which EHRM each GPS location belongs to
- id -> A label indicating the individual
- "start" -> the timestamp when the EHRM started, with the animal exiting the home range
- "end" -> the timestamp corresponding to the moment when an animal stopped the EHRM and came back to the HR
- "postrel.days" -> the number of days elapsed between each EHRM and the release
- "n.fix" -> The number of GPS locations inside of each EHRM
- "mean.difftime" -> The median time elapsed between GPS locations in a EHRM
- "tot.len" -> The total traveled distance in each EHRM, obtained by summing steps
- "avg.northing" -> The mean direction of steps in each EHRM
- "max.dist.hr" -> The maximum distance attained by an individual in a certain EHRM
- "median.step.lgt" -> The median value of step lengths in a certain EHRM
- "cluster" -> The label from cluster analysis, categorizing EHRM in 3 categories, corresponding to two different movements

The file "FittedHRs_20250814.zip" contains all the home-ranges, estimated through continuous time movement models and AKDE/wKADE. Due to the size of each file, this folder is compressed. Single R files should be accessed from the unzipped file.

The file "Sardinia_EPSG32632.shp" contains the shapefile of Sardinia, the study area. Please note that the .cpg, .dbf, .prj, .qmd, .shp and .shx files should remain the same folder.

#######################################################################
# Codes
#######################################################################

The folder also contains 5 main software codes for replicating the analyses in R (https://www.r-project.org/):
    - The file "Script_HomeRange_20250814.R" computes HR for each individual
    - The file "Script_ExtraHRMovements_20250814.R" combines tracks and HRs to identify extra home-range movements
    - The file "Script_ClusterAnalysis_20250814.R" carries out cluster analysis to classify extra home-range movements
    - The file "Script_GAMM_20250806.R" fits models to predict whether griffon vultures engages in short-range, medium-range or long-range movements, according to different environmental conditions.
    - The file "Script_GLMM_20250814.R" contains a code for model the effect of solar radiation of the duration of long-range extra-home range movements (LRMs).
    
#Home range calculation
#######################################################################
The script "Script_HomeRange_20250814.R" computes HR for each individual, through different movement models and AKDE/wAKDE in the ctmm package (https://ctmm-initiative.github.io/ctmm/). The script requires the GPS locations from "Dataset_GAMM_20250807.Rdata".

Due to the time required to fit ctmms, fitted AKDE/wAKDEs are stored, in the form of objects of type "UD", in the zipped file "FittedHRs_20250814.zip". This has to be uncompressed and files uploaded in the scripts, whenever needed.

#Calculation of extra-home range movement trajectories
#######################################################################

The script "Script_ExtraHRMovements_20250814.R" computes extra-home range trajectories for each individual and for each trajectory calculates:
their duration in hours, until when the animal came back in its home range
- their tortuosity, quantified throught the straightness index (see https://doi.org/10.1016/j.jtbi.2004.03.016)
- their mean orientation of segments, quantified as the arithmetic mean of their degrees north
- the maximum distance, expressed in km from the border of the home range
- their median step length, expressed in km

#Identification of different types of extra home range movement trajectories through cluster analysis
#######################################################################

The script "Script_ClusterAnalysis_20250814.R" carries out a Partitioning around Medoid (PAM) Cluster Analysis to distinguish between different types of extra-home range movement trajectories.Trajectories are clusterized according to
- their duration in hours, until when the animal came back in its home range
- their tortuosity, quantified throught the straightness index (see https://doi.org/10.1016/j.jtbi.2004.03.016)
- their mean orientation of segments, quantified as the arithmetic mean of their degrees north
- the maximum distance, expressed in km from the border of the home range
- their median step length, expressed in km

PAM clustering is robust against non-normality and we used different criteria to determine the number of clusters. We recommend reading Kassambara et al., #2017 "Practical guide to cluster analysis in R : unsupervised machine learning" for further details.

The script requires two datasets:
- "eHRm_20250807.Rdata", containing information about single extra-home range movements (EHRMs). Each row in this dataset corresponds to a track. This dataset is using to replicate PAM clustering
- "Dataset_GAMM_20250807.Rdata", containing the annotated GPS locations from tags. Each row in this dataset corresponds to a GPS location. This dataset is using to plot short, medium and long-range movements (Fig. 1 of the manuscript)


#Modelingthe effect of weather on different types of movement
#######################################################################

The script "Script_GAMM_20250806.R" works with "Dataset_GAMM_20250807.Rdata" and fits Generalized Additive Models to predict whether griffon vultures engages in short-range, medium-range or long-range movements, according to different environmental conditions. Generalized Additive Mixed Models are fitted with the mgcv package in R.

The dataset "Dataset_GAMM_20250807.Rdata" contains the processed data, ready to be used in the final GAMM aimed to link weather conditions to the probability of short, medium and long-range movement. Rows (observational units) correspond to GPS fix, acquired by GPS tags. 
The dataset contains the following columns:
- id -> The identifier of each one of the 20 individuals
- timestamp -> The timestamp of each GPS location
- lon.utm -> Longitude of each GPS location in epsg: 32632
- lat.utm -> Latitude of each GPS location in epsg: 32632
- difftime -> Time elapsed between each location and the previous one
- id.ehrm -> A label indicating to which extra-home-range-movement each GPS location belongs to
- rank.ehrm -> A label indicating the order of each GPS location in a certain extra-home range movement
- cluster -> A label, from PAM cluster analysis, indicating the group to which each extra-home range movement trajectory belonged to. The variable has the following values:
	-> 0 for GPS fix within the home range (short-range movements)
	-> 1, 2, 3 for GPS locations belonging to short movements outside the home range (medium-range movements)
	-> 4 for GPS locations belonging to long movements outside the home range (long-range movements)

- rel.type -> A label indicating whether individuals were subjected to short ("SA") or long acclimatization ("LA") before being released. For further details please see https://doi.org/10.1016/j.isci.2023.106699
- date -> The date of each GPS locations
- date.born -> The date when each individuals was deemed to be born. For further details please see https://doi.org/10.1016/j.isci.2023.106699
- days.age -> The age of each animal, when each GPS locations was collected, obtained by differencing "date" and "date.born"
- days.age.std -> The age of each animal, converted to a Z-score (https://en.wikipedia.org/wiki/Standard_score)
- yday -> The day of each location, on each year. From 1 to 366.
- yday.std -> The day of each location, converted to a Z-score
- behav -> the type of movement, obtained from the "cluster" variable. As an integer, to be used with the ocat family in mgcv (see https://stat.ethz.ch/R-manual/R-devel/library/mgcv/html/ocat.html for a complete description)
	-> 1 corresponds to short-range movements (cluster == 1)
	-> 2 corresponds to medium-range movements (cluster %in% c(2, 3, 4))
	-> 3 corresponds to long-range movements (cluster == 4)
- wind.str -> Wind strength, in m/s. Wind strength was calculated from the u and v component of the wind at an elevation of 100 m, downloaded from ERA5 (https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=overview). With the formula sqrt(u^2 + v^2)
- wind.str.std -> Wind strength, converted to a Z-score.
- wind.dir -> Wind direction, expressed in degrees from North (from 0 to 360). Wind direction was calculated from the u and v component of the wind at an elevation of 100 m, downloaded from ERA5 (https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=overview). With the formula (atan2(u/d$wind.str, v/d$wind.str)*(180/pi)) + 180
- wind.dir.std -> Wind direction, converted to a Z-score
- sol.rad -> Solar radiation, expressed as the mean surface downward short-wave radiation flux, downloaded from ERA5 (https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=overview).
- sol.rad.std <- Solar radiation, converted to a Z-score

#Assessing the effect of solar radiation over the duration of long-range movements
#######################################################################

The file "Script_GLMM_20250814.R" contains a code for model the effect of solar radiation of the duration of long-range extra-home range movements (LRMs).

To replicate the analyses, two datasets are needed:
- "Dataset_GAMM_20250807.Rdata", containing the annotated GPS locations from tags. Each row in this dataset corresponds to a GPS location.
- "eHRm_20250807.Rdata", containing information about single extra-home range movements (EHRMs). Each row in this dataset corresponds to a track.
	
The dataset eHRm_20250807.Rdata contains the following columns:
- id.ehrm -> A label indicating to which EHRM each GPS location belongs to
- id -> A label indicating the individual
- "start" -> the timestamp when the EHRM started, with the animal exiting the home range
- "end" -> the timestamp corresponding to the moment when an animal stopped the EHRM and came back to the HR
- "postrel.days" -> the number of days elapsed between each EHRM and the release
- "n.fix" -> The number of GPS locations inside of each EHRM
- "mean.difftime" -> The median time elapsed between GPS locations in a EHRM
- "tot.len" -> The total traveled distance in each EHRM, obtained by summing steps
- "avg.northing" -> The mean direction of steps in each EHRM
- "max.dist.hr" -> The maximum distance attained by an individual in a certain EHRM
- "median.step.lgt" -> The median value of step lengths in a certain EHRM
- "cluster" -> The label from cluster analysis, categorizing EHRM in 3 categories, corresponding to two different movements

An overview of the GLMM is available in "Appendix4.html"

#######################################################################
#Do you want to know more
#######################################################################
For further questions, please contacts dr. Jacopo Cerri (j.cerri@ibs.bialowieza.pl)
