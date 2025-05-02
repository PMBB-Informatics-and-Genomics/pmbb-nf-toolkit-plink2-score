
Documentation for PLINK2 Score
==============================

# Module Overview


A workflow that computes individual level polygenic scores from variant level scores using PLINK2 Score

[Example Module Config File](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/tree/main/Example_Configs/plink_score.config)

[Example nextflow.config File](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/tree/main/Example_Configs/nextflow.config)
## Cloning Github Repository:


* Command: `git clone https://github.com/PMBB-Informatics-and-Genomics/geno_pheno_workbench.git`

* Navigate to relevant workflow directory run commands (our pipelines assume all of the nextflow files/scripts are in the current working directory)
## Software Requirements:


* [Nextflow version 23.04.1.5866](https://www.nextflow.io/docs/latest/cli.html)

* [Singularity version 3.8.3](https://sylabs.io/docs/) OR [Docker version 4.30.0](https://docs.docker.com/)
## Commands for Running the Workflow


* Singularity Command: `singularity build plink2_score.sif docker://pennbiobank/plink2-score:latest`

* Docker Command: `docker pull pennbiobank/plink2-score:latest`

* Command to Pull from Google Container Registry: `docker pull gcr.io/verma-pmbb-codeworks-psom-bf87/plink2-score:latest`

* Run Command: `nextflow run /path/to/toolkit/module/plink2_score.nf`

* Common `nextflow run` flags:

    * `-resume` flag picks up the workflow where it left off, otherwise, the workflow will rerun from the beginning

    * `-stub` performs a sort of dry run of the whole workflow, checks channels without executing any code

    * `-profile` selects the compute profiles we set up in nextflow.config (see nextflow.config file below)

    * `-profile` selects the compute profiles we set up in nextflow.config (see nextflow.config file below)

    * `-profile standard` uses the docker image to executes the processes

    * `-profile cluster` uses the singularity container and submits processes to a queue- optimal for HPC or LPC computing systems

    * `-profile all_of_us` uses the docker image to execute pipelines on the All of Us Researcher Workbench

* for more information visit the [Nextflow documentation](https://www.nextflow.io/docs/latest/cli.html)
# Configuration Parameters and Input File Descriptions

## Workflow


* `validation_populations` (Type: Map (Dictionary))

    * dictionary that specifies validation cohort information. each key is a cohort nickname (ex: PMBB, UKBB, eMERGE). each key must have nested keys 'plink_prefix', 'plink_suffix', 'plink_file_flag', 'population_subset_file' and 'variant_id format' but change these values accordingly (make sure the values are in quotes). plink_prefix must have full file path (If you want to use a relative path, add "${launchDir}/" in front of the path). plink files must be chromosome separated. all cohorts must be in the same genome build as each other and as the PGS outputs

* `input_descriptor_table_colnames` (Type: Map (Dictionary))

    * dictionary including names of essential columns in input descriptor table. specifies cohort (nickname or dataset), genetic ancestry, phenotype, and PGS weights FULL FILE PATH AND FILENAME column names (If you want to use a relative path, add "${launchDir}/" in front of the path)

* `input_descriptor_table_filename (Compute PGS)` (Type: File Path)

    * file that specifies the cohort, ancestry, phenotype, and pgs weights filenames. must have four columns: cohort, ancestry, phenotype, pgs weights filename. MUST BE A CSV (comma delimited)

* `my_python` (Type: File Path)

    * Path to the python executable to be used for python scripts - often it comes from the docker/singularity container (/opt/conda/bin/python)
## Pre-Processing


* `score_variant_id_format` (Type: String)

    * score_variant_id_format specifies the variant ID format in the score file. this assumes that all score files have the same variant ID. modeled this based on the plink format. examples: "RSID" = RSID (ex: rs6893237), "@:#:$r:$a" = chr:pos:ref:alt (ex: 1:100000:G:A), "chr@:#:$r:$a" = chr:pos:ref:alt (ex: chr1:100000:G:A), '@:#' = chr:pos (ex: 1:100000), 'chr@:#' = chr:pos (ex: chr1:100000)

* `score_file_colnames` (Type: Map (Dictionary))

    * dictionary that specifies the column names in your PGS output files. need to specify chromosome column, position column, A1 column, A2 column, variant ID column, and score (PGS) column. this assumes that all PGS output files have the same column names. do not change the keys (pos_colname, a1_colname, etc), only change the values accordingly
## PLINK


* `independent_se` (Type: String)

    * The 'se' modifier causes the input coefficients to be treated as independent standard errors; in this case, standard errors for the score average/sum are reported, under a Gaussian approximation. (This will of course tend to underestimate standard errors when scored variants are in LD.) possible choices: '' (blank) = not using this modifier, 'se' = using this modifier

* `no_mean_imputation` (Type: String)

    * By default, copies of unnamed alleles contribute zero to score, while missing genotypes contribute an amount proportional to the loaded (via --read-freq) or imputed allele frequency. To throw out missing observations instead (decreasing the denominator in the final average when this happens), use the 'no-mean-imputation' modifier.  possible choices: '' (blank) = not using this modifier, 'no-mean-imputation' = using this modifier

* `read_freq` (Type: String)

    * the --read-freq loads in allele frequency estimates from a --freq (PLINK 1.x ok), --geno-counts, or PLINK 1.9 --freqx report, instead of imputing them from the immediate dataset. possible file types (by suffix): .afreq/.acount/.gcount/.freq/.frq/.frq.count/.frqx. possible choices: 'auto' = not specifying an allele frequency file, instead imputing them from the dataset being used, 'filename' = FULL file path and filename of an allele frequency file (If you want to use a relative path, add "${launchDir}/" in front of the path). 

* `xchr_model` (Type: String)

    * PLINK2 dosages are on a 0..2 scale on regular diploid chromosomes, and 0..1 on regular haploid chromosomes. However, chromosome X (chrX) doesn't fit neatly in either of those categories. --xchr-model lets you control its encoding in several contexts. Possible choices: '0' = Skip chrX, '1' = Male dosages are on a 0..1 scale on chrX, while females are 0..2. This was the PLINK 1.x default, '2' = Males and females are both on a 0..2 scale on chrX. This is the PLINK 2 default

* `dosage_transformation` (Type: String)

    * defines whether dosage information is transformed. possible choices: '' (blank) = basic allele dosages used (0..2 on diploid chromosomes, 0..1 on haploid, male chrX encoding controlled by --xchr-model), 'center' = translates all dosages to mean zero. (More precisely, they are translated based on allele frequencies, which you can control with --read-freq), 'variance-standardize' = linearly transforms each variant's dosage vector to have mean zero, variance 1 (cannot be used with chrX), 'dominant' = causes dosages greater than 1 to be treated as 1 (cannot be used with chrX), ‘recessive' = uses max(dosage - 1, 0) on diploid chromosomes (cannot be used with chrX)

* `my_plink2` (Type: File Path)

    * Path to the PLINK2 executable to be used for PLINK2 score - often it comes from the docker or singularity container (plink2) (is on the path in the container
# Output Files from PLINK2_Score


* Boxplots

    * Boxplot showing the distribution of individual-level polygenic scores per validation population and phenotype

    * Type: Distribution Plot

    * Format: png

    * Parallel By: Validation Population

* Density plots

    * Density plot showing the distribution of individual-level polygenic scores per validation population and phenotype

    * Type: Distribution Plot

    * Format: png

    * Parallel By: Validation Population

* PLINK Score Averages

    * This is the average score of all the variants for each person (Summed score/number of alleles), for all chromosomes.

    * Type: Computed Risk Scores

    * Format: txt

    * Parallel By: Validation Population

    * Output File Header:





    ```
    #FID    IID     ALLELE_CT       NAMED_ALLELE_DOSAGE_SUM    ANCESTRY.PHENOTYPE.SCORE_AVG
    ```

* PLINK Score Variant Lists

    * This is a list of all the variants used in each validation cohort to compute the score, for all chromosomes.

    * Type: Computed Risk Scores

    * Format: txt

    * Parallel By: Validation Population

    * Output File Header:





    ```
    Variant_ID
    ```

* PLINK Score Sums

    * This is the summed score of all the variants for each person, for all chromosomes.

    * Type: Computed Risk Scores

    * Format: txt

    * Parallel By: Validation Population

    * Output File Header:





    ```
    #FID    IID     ALLELE_CT       NAMED_ALLELE_DOSAGE_SUM    ANCESTRY.PHENOTYPE.SCORE_SUM
    ```
# Current Dockerfile for the Container/Image


```docker
FROM continuumio/miniconda3

WORKDIR /app

RUN apt-get update \
    # install tools needed to install plink
    && apt-get install -y --no-install-recommends wget unzip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    # install plink2
    && wget https://s3.amazonaws.com/plink2-assets/alpha5/plink2_linux_x86_64_20240105.zip \
    && unzip plink2_linux_x86_64_20240105.zip \
    && rm -rf plink2_linux_x86_64_20240105.zip \
    # move plink2 executable to $PATH
    && mv plink2 /usr/bin \
    # install R and python packages needed for pipeline
    && conda install -y -n base -c conda-forge pandas seaborn scipy matplotlib \
    && conda clean --all --yes

USER root

```