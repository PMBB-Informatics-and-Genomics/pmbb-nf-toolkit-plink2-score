
Documentation for PLINK2 Score
==============================

# Module Overview


A workflow that computes individual level polygenic scores from variant level scores using PLINK2 Score

[Example Module Config File](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/tree/main/Example_Configs/plink_score.config)

[Example nextflow.config File](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/tree/main/Example_Configs/nextflow.config)
## Cloning Github Repository


* Command: `git clone https://github.com/PMBB-Informatics-and-Genomics/geno_pheno_workbench.git`

* Navigate to relevant workflow directory...
## Software Requirements


* [Nextflow version 23.04.1.5866](https://www.nextflow.io/docs/latest/cli.html)

* [Singularity 3.8.3](https://sylabs.io/docs/) OR [Docker 4.30.0](https://docs.docker.com/)
## Commands for Running the Workflow


* Singularity Command: `singularity build plink2_score.sif docker://pennbiobank/plink2-score:latest`

* Docker Command: `docker pull pennbiobank/plink2-score:latest`

* Pull from Google Container Registry: `docker pull gcr.io/verma-pmbb-codeworks-psom-bf87/plink2-score:latest`

* Run Command: `nextflow run /path/to/toolkit/module/plink2_score.nf`

* Common `nextflow run` flags:

    * `-resume` flag picks up workflow where it left off

    * `-stub` performs a dry run, checks channels without executing code

    * `-profile` selects the compute profiles in nextflow.config

    * `-profile standard` uses the Docker image to execute processes

    * `-profile cluster` uses the Singularity container and submits processes to a queue

    * `-profile all_of_us` uses the Docker image on All of Us Workbench

* More info: [Nextflow documentation](https://www.nextflow.io/docs/latest/cli.html)
# Input Files for PLINK2_Score


* PLINK2 executable

    * This is the /path/to/the/correct/plink executable with the right version. It usually comes from the docker/singularity container

    * Type: Executable

* Python executable

    * This is the /path/to/the/correct/python executable that has the right packages installed and right version. It usually comes from the docker/singularity container. If not, be sure to have most of the standard data packages like pandas, numpy, scipy, matplotlib, seaborn, etc

    * Type: Executable

* SNP Weight Files

    * Variant-level scores

    * Type: PGS SNP Weights

    * Format: txt

    * File Header:


    ```
    CHR     RSID    POS     A1      A2      PGS
    ```
# Output Files for PLINK2_Score


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

    * File Header:


    ```
    #FID    IID     ALLELE_CT       NAMED_ALLELE_DOSAGE_SUM    ANCESTRY.PHENOTYPE.SCORE_AVG
    ```

        * Parallel By: Validation Population

* PLINK Score Variant Lists

    * This is a list of all the variants used in each validation cohort to compute the score, for all chromosomes.

    * Type: Computed Risk Scores

    * Format: txt

    * File Header:


    ```
    Variant_ID
    ```

        * Parallel By: Validation Population

* PLINK Score Sums

    * This is the summed score of all the variants for each person, for all chromosomes.

    * Type: Computed Risk Scores

    * Format: txt

    * File Header:


    ```
    #FID    IID     ALLELE_CT       NAMED_ALLELE_DOSAGE_SUM    ANCESTRY.PHENOTYPE.SCORE_SUM
    ```

        * Parallel By: Validation Population
# Parameters for PLINK2_Score

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
## Pre-Processing


* `score_variant_id_format` (Type: String)

    * score_variant_id_format specifies the variant ID format in the score file. this assumes that all score files have the same variant ID. modeled this based on the plink format. examples: "RSID" = RSID (ex: rs6893237), "@:#:$r:$a" = chr:pos:ref:alt (ex: 1:100000:G:A), "chr@:#:$r:$a" = chr:pos:ref:alt (ex: chr1:100000:G:A), '@:#' = chr:pos (ex: 1:100000), 'chr@:#' = chr:pos (ex: chr1:100000)

* `score_file_colnames` (Type: Map (Dictionary))

    * dictionary that specifies the column names in your PGS output files. need to specify chromosome column, position column, A1 column, A2 column, variant ID column, and score (PGS) column. this assumes that all PGS output files have the same column names. do not change the keys (pos_colname, a1_colname, etc), only change the values accordingly
## Workflow


* `validation_populations` (Type: Map (Dictionary))

    * dictionary that specifies validation cohort information. each key is a cohort nickname (ex: PMBB, UKBB, eMERGE). each key must have nested keys 'plink_prefix', 'plink_suffix', 'plink_file_flag', 'population_subset_file' and 'variant_id format' but change these values accordingly (make sure the values are in quotes). plink_prefix must have full file path (If you want to use a relative path, add "${launchDir}/" in front of the path). plink files must be chromosome separated. all cohorts must be in the same genome build as each other and as the PGS outputs

* `input_descriptor_table_colnames` (Type: Map (Dictionary))

    * dictionary including names of essential columns in input descriptor table. specifies cohort (nickname or dataset), genetic ancestry, phenotype, and PGS weights FULL FILE PATH AND FILENAME column names (If you want to use a relative path, add "${launchDir}/" in front of the path)

* `input_descriptor_table_filename (Compute PGS)` (Type: File Path)

    * file that specifies the cohort, ancestry, phenotype, and pgs weights filenames. must have four columns: cohort, ancestry, phenotype, pgs weights filename. MUST BE A CSV (comma delimited)

* `my_python` (Type: File Path)

    * Path to the python executable to be used for python scripts - often it comes from the docker/singularity container (/opt/conda/bin/python)
# Configuration and Advanced Workflow Files

## Current Dockerfile for Container/Image


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
## Current `nextflow.config` contents


```
includeConfig "${launchDir}/plink2_score.config"

// set up profile
// change these parameters as needed
profiles {
    non_docker_dev {
        process.executor = awsbatch-or-lsf-or-slurm-etc
    }

    standard {
        process.executor = awsbatch-or-lsf-or-slurm-etc
        process.container = 'pennbiobank/plink2-score:latest'
        docker.enabled = true
    }
    cluster {
        process.executor = awsbatch-or-lsf-or-slurm-etc
        process.queue = 'epistasis_normal'
        process.memory = '30GB'
        process.container = 'plink2_score.sif'
        singularity.enabled = true
        singularity.runOptions = '-B /root/,/directory/,/names/'
    }

    all_of_us {
       // CHANGE EVERY TIME! These are specific for each user, see docs
        google.lifeSciences.serviceAccountEmail = service@email.gservicaaccount.com
        workDir = /path/to/workdir/ // can be gs://
        google.project = terra project id

        // These should not be changed unless you are an advanced user
        process.container = 'gcr.io/verma-pmbb-codeworks-psom-bf87/plink2-score:latest' // GCR SAIGE docker container (static)

        // these are AoU, GCR parameters that should NOT be changed
        process.memory = '15GB' // minimum memory per process (static)
        process.executor = awsbatch-or-lsf-or-slurm-etc
        google.zone = "us-central1-a" // AoU uses central time zone (static)
        google.location = "us-central1"
        google.lifeSciences.debug = true 
        google.lifeSciences.network = "network"
        google.lifeSciences.subnetwork = "subnetwork"
        google.lifeSciences.usePrivateAddress = false
        google.lifeSciences.copyImage = "gcr.io/google.com/cloudsdktool/cloud-sdk:alpine"
        google.enableRequesterPaysBuckets = true
        // google.lifeSciences.bootDiskSize = "20.GB" // probably don't need this
    }
}

```
# Detailed Pipeline Steps


from pathlib import Path

detailed_steps_file = Path("Markdowns/Pipeline_Detailed_Steps.md")

# Write the detailed steps content to a separate file
detailed_steps_file

# Detailed Steps for Runnning One of our Pipelines

Note: test data were obtained from the [SAIGE github repo](https://github.com/saigegit/SAIGE).

## Part I: Setup
1. Start your own tools directory and go there. You may do this in your project analysis directory, but it often makes sense to clone into a general `tools` location

```sh
# Make a directory to clone the pipeline into
TOOLS_DIR="/path/to/tools/directory"
mkdir $TOOLS_DIR
cd $TOOLS_DIR
```

2. Download the source code by cloning from git

```sh
git clone https://github.com/PMBB-Informatics-and-Genomics/pmbb-nf-toolkit-saige-family.git
cd ${TOOLS_DIR}/pmbb-nf-toolkit-saige-family/
```

3. Build the `saige.sif` singularity image
- you may call the image whatever you like, and store it wherever you like. Just make sure you specify the name in `nextflow.conf`
- this does NOT have to be done for every saige-based analysis, but it is good practice to re-build every so often as we update regularly. 

```sh
cd ${TOOLS_DIR}/pmbb-nf-toolkit-saige-family/
singularity build saige.sif docker://pennbiobank/saige:latest
```

## Part II: Configure your run

1. Make a separate analysis/run/working directory.
   - The quickest way to get started, is to run the analysis in the folder the pipeline is run. However, subsequent analyses will over-write results from previous analyses. 
   - ❗This step is optional, but We Highly recommend making a  `tools` directory separate from your `run` directory. The only items that need to be in the run directory are the `nextflow.conf` file and the `${workflow}.conf` file.

```sh
WDIR="/path/to/analysis/run1"
mkdir -p 
cd $WDIR
```

2. Fill out the `nextflow.config` file for your system.
    - See [Nextflow configuration documentation](https://www.nextflow.io/docs/latest/config.html) for information on how to configure this file. An example can be found on our GitHub: [Nextflow Config](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/Example_Configs/nextflow.config).
    - ❗IMPORTANTLY, you must configure a user-defined profile for your run environments (local, docker, saige, cluster, etc.). If multiple profiles are specified, run with a specific profile using `nextflow run -profile ${MY_PROFILE}`.
    - For singularity, The profile's attribute `process.container` should be set to `'/path/to/saige.sif'` (replace `/path/to` with the location where you built the image above). See [Nextflow Executor Information](https://www.nextflow.io/docs/latest/executor.html) for more details.
    - ⚠️As this file remains mostly unchanged for your system, We recommend storing this file in the `tools/pipeline` directory and symlinking it to your run directory.

3. Create a pipeline-specific `.config` file specifying your run parameters and input files. See Below for workflow-specific parameters and what they mean.
   - Everything in here can be configured in `nextflow.config`, however we find it easier to separate the system-level profiles from the individual run parameters. 
   - Examples can be found in our Pipeline-Specific [Example Config Files](https://github.com/PMBB-Informatics-and-Genomics/pmbb-geno-pheno-toolkit/Example_Configs/).
   - you can compartamentalize your config file as much as you like by passing 
   - There are 2 ways to specify the config file during a run:
      - with the `-c` option on the command line: `nextflow run -c /path/to/workflow.conf`
      - in the `nextflow.conf`: at the top of the file add: `includeConfig '/path/to/workflow.conf'` 

## Part III: Run your analysis

- ❗We HIGHLY recommend doing a STUB run to test the analysis using the `-stub` flag. This is a dry run to make sure your environment, parameters, and input_files are specified and formatted correctly. 
- ❗We HIGHLY recommend doing a test run with the included test data in `${TOOLS_DIR}/pmbb-nf-toolkit-saige-family/test_data`
- in the `test_data/` directory for each pipeline, we have several pre-configured analyses runs with input data and fully-specified config files.

```sh
# run an exwas stub
nextflow run /path/to/pmbb-nf-toolkit-saige-family/workflows/saige_exwas.nf -profile cluster -c /path/to/run1/exwas.conf -stub
# run an exwas for real
nextflow run /path/to/pmbb-nf-toolkit-saige-family/workflows/saige_exwas.nf -profile cluster -c /path/to/run1/exwas.conf
# resume an exwas run if it was interrupted or ran into an error
nextflow run /path/to/pmbb-nf-toolkit-saige-family/workflows/saige_exwas.nf -profile cluster -c /path/to/run1/exwas.conf -resume
```
