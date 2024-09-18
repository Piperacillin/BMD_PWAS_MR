# BMD_PWAS_MR

Reference:
1. A Mendelian randomization analysis for the druggable genome in Parkinson's disease.(https://www.nature.com/articles/s41467-021-26280-1)
2. A Mendelian randomization analysis for the druggable genome in AA. (https://www.thelancet.com/journals/ebiom/article/PIIS2352-3964(22)00381-4/fulltext)
3. Introduction to forestploter. (https://cran.r-project.org/web/packages/forestploter/vignettes/forestploter-intro.html)

20240918 Update: The manuscript is scheduled for submission to the journal for internal and peer review. The formal code release will occur upon the manuscript's official publication.

## Pipeline Overview

1. Select exposure data and outcome data.
```bash
echo "cis_deCODE
cis_UKBBB
cis_islander
full_UKBBB
full_deCODE
full_islander" > exposure_data.txt

echo "BMD outcome names" > outcomes.txt

echo "" > discovery_outcomes.txt
```
2. Prepare the data for the Mendelian randomization analysis 
```bash
bash ./mr_druggable_genome_BMD/shell/06data_prep.sh 
```
3. Generate scripts that can be run in parallel

```bash
while read EXPOSURE_DATA; do
    while read OUTCOME; do

            export EXPOSURE_DATA=${EXPOSURE_DATA}
            export OUTCOME=${OUTCOME}
            mkdir ${EXPOSURE_DATA}_${OUTCOME}

            while read DISCOVERY_OUTCOME; do
                    export DISCOVERY_OUTCOME=${DISCOVERY_OUTCOME}
                    bash ./mr_druggable_genome_BMD/shell/07generate_parallel_scripts.sh
            done < discovery_outcomes.txt

    done < outcomes.txt
done < exposure_data.txt
```

4. Run Mendelian randomization for the druggable genome using all the exposure data and outcome data you have specified.

bash batch_bash_liberal_all.sh
```

```bash 
5. Remove genes that throw errors. Note: you may need to rerun this step a few times until all genes that cause an error are removed.
```bash
nohup bash ./mr_druggable_genome_BMD/shell/run_liberal_scripts_failed_nohup.sh &> ./mr_druggable_genome_BMD/shell/nohup_run_liberal_scripts_failed.log & 

```
6. For each exposure-data-outcome combination, put all the results into one results file. #FDR0.05
```bash
while read EXPOSURE_DATA; do
    while read OUTCOME; do
        export EXPOSURE_DATA=${EXPOSURE_DATA}
        export OUTCOME=${OUTCOME}
        cd ./${EXPOSURE_DATA}_${OUTCOME}/results
        nohup Rscript ../../mr_druggable_genome_BMD/R/combine_results_liberal_r2_0.2.R &> ../../${EXPOSURE_DATA}_${OUTCOME}/nohup_combine_results_liberal_r2_0.2_${EXPOSURE_DATA}_${OUTCOME}.log &
        cd ../..
    done < outcomes.txt
done < exposure_data.txt
```

9. Rerun the analysis for all significant genes using the clumping threshold r2 = 0.001.
```bash
while read EXPOSURE_DATA; do
    while read OUTCOME; do
        (( outcome_count++ ))
        
        export EXPOSURE_DATA=${EXPOSURE_DATA}
        export OUTCOME=${OUTCOME}
        nohup bash ./${EXPOSURE_DATA}_${OUTCOME}/script_conservative_r2_0.001_${EXPOSURE_DATA}_${OUTCOME}.sh &> ./${EXPOSURE_DATA}_${OUTCOME}/nohup_script_conservative_r2_0.001_${EXPOSURE_DATA}_${OUTCOME}.log &
        
        if [ $outcome_count -eq 10 ]; then
            wait
            outcome_count=0
        fi
        
    done < outcomes.txt
done < exposure_data.txt

wait
```
5. Remove genes that throw errors. Note: you may need to rerun this step a few times until all genes that cause an error are removed.
```bash
nohup bash ./mr_druggable_genome_BMD/shell/run_liberal_scripts_failed_nohup_conservative.sh &> ./mr_druggable_genome_BMD/shell/run_liberal_scripts_failed_nohup_conservative.log & 
```


10. Some data formatting steps.
```bash

mkdir full_results

bash ./mr_druggable_genome_BMD/shell/final_results_report.sh

Rscript ./mr_druggable_genome_BMD/R/format_supplement.R
```
11. Generate forest plots. This script is not generic for any exposure/outcome data.
```bash
mkdir figures

Rscript ./mr_druggable_genome_BMD/R/make_forest_plot.R
```
12. Colocalization analysis.

```bash
mkdir coloc

Rscript ./mr_druggable_genome_BMD/R/out_rdata_to_txt.R

Rscript ./mr_druggable_genome_BMD/R/data_prep_coloc.R

Rscript ./mr_druggable_genome_BMD/R/run_coloc.R

```
