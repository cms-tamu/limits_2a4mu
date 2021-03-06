# Notes:
. -m HybridNew :  Searching for a signal where a small number of events are expected (<10)
                  Because asymptotic profile likelihood test-statistic distribution is no longer a good approximation.
                  but can be very CPU / time intensive 
. STD command  -> combine -m 125 -M HybridNew --rule CLs --testStat LHC datacard_H2A4Mu_mA_0.2200_GeV.txt -t 100000 -s -1 (--fork 10)

## Installing the framework
```
cmsrel CMSSW_7_4_7 #(Release needed for the Hggs combine tool)   
cd CMSSW_7_4_7/src/   
cmsenv    
git clone https://github.com/cms-analysis/HiggsAnalysis-CombinedLimit.git HiggsAnalysis/CombinedLimit
cd HiggsAnalysis/CombinedLimit
source env_standalone.sh 
make -j 8; make # second make fixes compilation error of first
cd ../../
git clone https://github.com/cms-tamu/MuJetAnalysis_LimitSetting.git    
scram b -j 6   
cd MuJetAnalysis_LimitSetting    
```
# Running the model independent limits
1. Copy here "ws_FINAL.root" from bbBar estimation    
   -> RooStat file which has background model inside it.   

2. root -b -q .x CreateDatacards.C+   
   -> The first time you need to use option "true", so you will create "CreateROOTfiles.sh", a file that uses makeWorkSpace_H2A4Mu.C to make the RooStat files with S and B needed by CMS official limit calculator.   
   -> NB: makeWorkSpace_H2A4Mu.C has hardcoded inside the TH2 range and binning, plus the Signal events. So for unblinding, add here the events you see.   

3. (1st time only) cd macros; source CreateROOTfiles.sh; cd ..;    
   -> Make RooStat files that should be supplied to CMS official limit calculator.    

3. Send jobs to run on all datacards and save outputs in outPut.txt. The options with the least toys is
```
source macros/RunOnDataCard_std.sh
```
If you want more toys, please use one of the following below
```
source macros/RunOnDataCard_T10000.sh #10000 toys per job
source macros/RunOnDataCard_T30000.sh #30000 toys per job
source macros/RunOnDataCard_T50000.sh #50000 toys per job

```
-> RunOnDataCard_T50000 runs more toys and it is better, but take more time. Choose the one you want.    

4. cd macros; python PrintOutLimits.py; cd ..;   
   -> This macro will print the lines you have to copy inside scripts/CmsLimitVsM.py (that will be used by Plots.py). 
   -> You have to copy the lined from "That contain N limits: \n XXX" until teh line before "Now remove the worse items". Remove also the first number each line, that represent the number of the jobs used to produce such limit   

5. In case you run combine several times (or more poeple followed the steps until here), you may want to average all the results, and place in scripts/CmsLimitVsM.py the final one
   -> Imagine 2 people followed this instruction, and you have two output from "PrintOutLimits.py". You copy the lines after "That contain N limits: \n XXX" until the line before "Now remove the worse items" in tow txt files.
   -> Then you run: python MergeLimit.py (where inside you specified the txt files locations and names)
   -> It will print out the lines to place in "scripts/CmsLimitVsM.py"

6. vim scripts/CmsLimitVsM.py + copy the lines you just produced after "Limits_HybridNew = ["    
   -> If you change method from HybridNew, you can copy the line into another list and specify the correct method in CmsLimitVsM.py.    

7. python Plots_RunMe.py    
   -> To make final limit plots. Here you can specify which function of Plots.py do you want to use.    

# Some notes   
1. NMSSM plots are done assuming an efficiency for the H decay taken from 2012 analysis
