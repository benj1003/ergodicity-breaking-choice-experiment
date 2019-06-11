# ergodicity-breaking-choice-experiment
This is a repository for all code and data to reproduce an experiment exploring the effects of ergodicity breaking on economic choice behavior

# readMe

 Everything contained in this folder is necessary for reproducing/replicating the
 analyses of the experiment described in the paper:
 Ergodicity-breaking reveals time optimal economic behavior in humans
 by David Meder, Finn Rabe, Tobias Morville, Kristoffer H. Madsen, Magnus T. Koudahl, 
 Ray J. Dolan, Hartwig R. Siebner, and Oliver J. Hulme.

 This folder contains the data as well as code
 The readMe file has been structured loosely according to IEEE suggestions:
 https://journals.ieeeauthorcenter.ieee.org/create-your-ieee-article/prepare-supplementary-materials/creating-a-readme-file-for-datasets/


# Platform/Environment
 The original raw data files are in .txt format. A python script (Python 2.7) 
 reads in these individual data files (separate files per subject and 
 session), computes a number of basic variables and saves them in one
 combined .mat file.

 Most subsequent analyses have been performed with, or called from Matlab.
 The Hierarchical Bayesian modelling has been run with JAGS (via matjags, which allows
 running JAGS via matlab code.

 Separate statistical analyses were run on the statistical software JASP,
 https://jasp-stats.org/ in some cases this is takes the same raw data as input, in 
 other cases statistics are run on point estimates from the hiearchical bayesian modelling.
 The data that JASP runs on is in the .csv in the JASP folder.

# Folders summary 
 "data" contains all data files. For all details on files and the 
 variables they contain, please refer to _codebook.txt

 "figs" receives figures that are automatically generated by scripts.

 "fractals" contains the fractal images that participants were presented
 with. These images are a subset of the fractals available at
 https://github.com/smathot/materials_for_P0010.5/tree/master/stimuli/exp2/images

 "functions misc" contains all miscellaneous functions or toolboxes that
 are relied upon, e.g. the Variational Bayes toolbox.

 "JAGS" contains different versions of the "JAGS_..." scripts using
 JAGS code that are called by the matjags script in the folder "matjags"
 (see below)

 "JASP" contains the jasp analysis scripts and .csv data files on which they run. 
 you need to tell the jasp analysis scripts where the .csv is, and sync so that 
 the analysis scripts run.

 "jobs" contains the shell script jobs that run different versions of the
 script runHLM#.m which sets up different models. 

 "matjags" contains matjags scripts that are the means by which matlab runs 
 JAGS. There are multiple copies of this script, each calling a different
 copy of JAGS, allowing many jobs to be run at the same time. matjags1.m
 calls the first version of JAGS in "jagstmp1" and so forth.
 
 "samples&stats" contain the outputs of JAGS, including all relevant
 statistics and samples for further analysis by plotHLM.m

 The remaining files in the base directory are the main matlab scripts and JAGS models


# Major Component Description of Data Set
 The original .txt files contain a trial-wise (rows) list of basic
 variables such as time points of when different events occurred, which
 buttons were pressed at which time and what kind of stimuli were
 presented.
 The python script "readingData.py" reads these .txt, computes additional
 variables and saves them in one .mat file containing all data for all
 subjects
 For further details on the data, please refer to "Codebook.m"


# Detailed Setup Instructions - setting up and running JAGS

 Runnning a model starts with setting up the script runHLM#.m where #
 indicates the version number. There are multiple versions according to
 the aims of the model, which could vary by type of model, or by the type
 of data, varying between real data or synthetic data. To call a given
 runHLM# you need to run the job runHLM_job#.sh in the "jobs" directory.

 Each version of runHLM contains different specifications for running a 
 JAGS model, pertaining to the following key variables:
       which model - runModelNum
       which data to model - synthMode
       which independent copy of JAGS to run - whichJAGS
       which quality levels of the model to run in which order - whichQuals
       whether to generate plots - runPlots
 For further details, please refer to one of the different runHLM scripts
 (runHLM1.m - runHLM28.m) in the folder "jobs" and read the commenting

 runHLM#
 runHLM#.m (in folder 'jobs') calls setHLM.m (in main folder) which 
 contains preset information about what the different quality levels mean 
 in terms of burn-in, number of samples etc.

 To monitor the progress of the job, open the associated slurm file that
 was generated in the jobs directory. old slurms can be archived in
 "old_slurms" (in "jobs") to keep in order.
 
 setHLM
 setHLM then calls computeHLM which is the main script for computing the
 hieararchical bayesian models. This script processes the data, can plot
 key information, and principally calls matjags to run the model. This is
 done by calling the different versions of the "JAGS_..." scripts using
 JAGS code, to be found in the folder "JAGS"

 computeHLM
 computeHLM is a general script for running several types of hierarchical
 bayesian model via JAGS. It can run hiearchical latent mixture models in
 which different utility models can be compared via inference on the model
 indicator variables, and it can run without latent mixtures of models for
 instance in order to estimate parameters of a given utility model. 

 computeEtaBeta2TimeAv
 computeEtaBeta2TimeAv computes the time average growth over the gambles
 given an eta and beta value. This script is called by computeHLM
 
 computeCheckJAGSscripts
 computeCheckJAGSscripts is used for debugging JAGS scripts. JAGS scripts
 cannot be debugged in a sequential manner as in matlab, thus this script 
 does sanity check computations whether variables computed by the JAGS 
 script are meaningful

 plotHLM
 plotHLM plots and post-processes HLM results, generating histograms, 
 computes MAPs, and generally visualises data


# Scripts summary - generating synthetic data for parameter and model recovery

 The two main scripts are computeSyntheticData4ModelRecovery and 
 computeSyntheticData4ParameterRecovery.

 computeSyntheticData4ModelRecovery
 "presents" synthetic agents with the gambles presented to the subjects. 
 The agents operate with different utility functions with different 
 parameter values. The script generates choice probabilities based on 
 these functions, and then probabilistically realises left-right choices. 

 computeSyntheticData4ParameterRecovery 
 does the same as above, but only with the isoelastic utility function 
 with different parameter values. This script also calls: 

 computeEtaDistribution
 which draws values from normal distribution with given mean and SD, 
 restricting them to be within 3 standard deviations for sampling eta
 values around a certain mean

 Both computeSyntheticData4ModelRecovery and 
 computeSyntheticData4ParameterRecovery are used for setting up variables 
 which are then passed to:

 computeChoicesModelRecovery / computeChoicesParameterRecovery 
 which calculate the utility differences of the gambles and choice 
 probabilities given specific parameter values of the utility function.
 They call upon:

 computeExpectedUtility
 which calculates utilities based on prospect theory given the input exponents,
 loss aversion parameters (lambda), and probability values (probVals), via
 a weighted sum, as per cumulative prospect theory.

 computeIsoelasticUtility
 which computes isoelastic utility according given an eta value

 computeChoiceRealisation
 which probabilisitically realises gamble choices based on choice
 probabilities.


# Scripts summary - other scripts

 computeSyntheticWealthTrajectories
 This script computes wealth trajectories for synthetic agents playing the
 ergodicity game repeatedly over several timescales, hours to weeks to
 years.

 computeFactor2Increment
 computeFactor2Increment computes linear changes in wealth as a function 
 of current wealth and growth rate. This script is called by 
 computeSyntheticWealthTrajectories.

 computeIncrement2Factor
 computeIncrement2Factor computes growth factors from additive growth 
 increments and current wealth. This script is called by 
 computeSyntheticWealthTrajectories.

# Prefixes
 script prefixes denote the scripts primary function. 
 runX is a script whose main function is to initiate the running of a model
 setX is a script that simply helps set up a run
 computeX is a script or function that computes 
 JAGS_X is the text file that specifies the JAGS model

# Contact information
 In case of questions, please contact Oliver Hulme oliverh@drcmr.dk or
 David Meder davidm@drcmr.dk


