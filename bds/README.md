Pipelines in Kundaje lab based on BigDataScript (BDS)
===============================================

Taking advandatge of the powerful pipeline language BigDataScript (http://pcingola.github.io/BigDataScript/index.html), Pipelines in Kundaje lab have the following features:

```
1) One command line (or one configuration file) to run the whole pipeline.
2) Resuming from the point of failure without re-doing finished stages.
3) All stages in the pipeline are parallelized in the most efficient way (using UNIX threads or Sun Grid Engine).
4) One species file to define all species specific parameters. 
5) Realtime HTML Progress reports to monitor pipeline jobs.
```


### Cloning/pulling this repo

When cloning, don't forget to FULLY clone including submodules.
```
$ git clone https://github.com/kundajelab/pipelines --recursive
```

When pulling,
```
$ git pull && git submodule update --init --recursive
```



### Installation instruction for BDS

Get BigDataScript (v0.9999 is stable and doesn't require high java version).
```
$ git clone https://github.com/pcingola/BigDataScript
$ cd BigDataScript
$ git checkout tags/v0.9999
$ cp distro/bds_Linux.tgz $HOME
$ cd $HOME
$ tar zxvf bds_Linux.tgz
```

(Recommended if you don't know what these are) Add the following lines to your $HOME/.bashrc or $HOME/.bash_profile:
```
export _JAVA_OPTIONS="-Xms256M -Xmx512M -XX:ParallelGCThreads=1"
export MAX_JAVA_MEM="8G"
export MALLOC_ARENA_MAX=4
export PATH=$PATH:$HOME/.bds
```

Find bds.config anywhere (pipelines/bds/ or pipelines/bds/chipseq/) in the pipeline repo and copy it to $HOME/.bds/.
```
$ mkdir -p $HOME/.bds
$ cp bds.config $HOME/.bds/
```



### Installation instruction for BDS on SCG3 clusters (scg3, carmack and crick)

Follow the instruction in the above section.



### Installation instruction for BDS on Kundaje lab clusters (mitra, nandi, vayu, kali, durga, wotan and amold)

For Kundaje lab members, BDS and all dependencies have already been installed on lab servers. Do not run install_dependencies.sh on Kundaje lab servers. Find bds.config anywhere (pipelines/bds/ or pipelines/bds/chipseq/) in the pipeline repo and copy it to $HOME/.bds/.
```
$ mkdir -p $HOME/.bds
$ cp bds.config $HOME/.bds/
```


### Running pipelines on SCG3 ###

You always need to submit pipeline jobs to Sun Grid Engine on SCG3 by using:
```
$ bds -s sge pipeline.bds ...
```

You need to carefully define resources settings (parameters like -wt, -memory, -wt_* -mem_*). -wt and -memory defines general walltime/max. memory for all jobs. On SCG3, if it's over 6 hours, jobs will be queued in a longer queue which makes you wait longer to get jobs actually executed. By default walltime is 11 hours. Change it to shorter than 6h if you want to get your pipeline faster.
```
-wt 5:59:59
```

You can also change resource settings of specific tasks (e.g. bwa aln, bowtie2, mac2 ...). For example,
```
-wt_macs2 100h
```



### Running pipelines on Kundaje lab cluster ###

You have two options `$ bds -s sge pipeline.bds ...` (running on SGE) or `$ bds pipeline.bds ...` (running with UNIX thread). For both, there is no limit for walltime and max. memory.




### How to deal with BDS pipeline errors?

```
1) Take a look at HTML report (which contains all STDERR/STDOUT for all jobs in the pipeline). It tells you everything about all jobs. Find out which step is errorneous. Carefully look at messages (STDERR and STDOUT).

2) Correct errors.
   2-1) Lack of memory: increase memory for all jobs (e.g. add -memory 20G) or a specific problematic job (e.g. add -mem_macs2 20G).
   2-2) Timeout: increase walltime for all jobs (e.g. add -wt 24h) or a specific long job (e.g. add -wt_macs2 200h).
                 (Warning! Most clusters have limit for walltime. Make it as shortest as you can to get your jobs queued quickly.)
   2-3) Wrong input: check all input files are available.
   2-4) Software error: use recommended software versions

3) Resume pipeline with the same command line that you used to start the pipeline with. Previous steps will be automatically skipped.
```


### How to resume pipeline from the point of failure?

Pipelines automatically determine if each stage has finished or not (comparing timestamps of input/output files for each stage). To run the pipeline from the point of failure, take a look at BDS HTML reports check stdoutcorrect error first and then just run the pipeline with the same command that you started the pipeline with. There is no additional parameter for restarting the pipeline.



### How to get help for all parameters?

Run the pipeline without additional command line argument.
```
$ bds [PIPELINE_BDS]
```



### How to run pipelines with Sun Grid Engine?

Run the pipeline without additional command line argument.
```
$ bds -s sge [PIPELINE_BDS]
```

When you run pipelines with SGE, default walltime and max. memory settings will be used if walltime and max. memory settings are not explicted defined.

If you SGE allows infinite walltime and max. memory, add the following to your command line.
```
-skip_wt_memory
```

Otherwise, carefully define walltime and max. memory for each jobs.

For example, if you want to generally use 6hours and 10G for all jobs and 20 hours and 20G specifically for MACS2 peak calling
```
-memory 10G -wt 6h -mem_macs2 30G -wt_macs2 20h
```



### How to define parameters?

There are two ways to define parameters for pipelines. Default values are already given for most of parameters. Take a look at example commands and configuration files (./examples). Both methods share the same key names.

1) From command line arguments 
```
$ bds [PIPELINE_BDS] [OPTS]
```
Example for chipseq pipeline:
```
$ bds chipseq.bds \
-fastq1 /DATA/ENCFF000YLW.fastq.gz \
-fastq2 /DATA/ENCFF000YLY.fastq.gz \
-ctl_fastq1 /DATA/Ctl/ENCFF000YRB.fastq.gz \
-bwa_idx /INDEX/encodeHg19Male_bwa-0.7.3.fa
...
```

2) From a configuration file
```
$ bds [PIPELINE_BDS] [CONF_FILE]
```
Example configuriation file for chipseq pipeline:
```
$ cat [CONF_FILE]

fastq1= /DATA/ENCFF000YLW.fastq.gz
fastq2= /DATA/ENCFF000YLY.fastq.gz
ctl_fastq1= /DATA/Ctl/ENCFF000YRB.fastq.gz
bwa_idx= /INDEX/encodeHg19Male_bwa-0.7.3.fa
...
```



### Using species file on SCG3 and Kundaje lab clusters (IMPORTANT)

Add the following to the command line and that's it.
```
-species [SPECIES; hg19, mm9, ...]
```

hg19 and mm9 are available for both SCG3 and Kundaje lab clusters. If you are interested in other species, modify pipelines/bds/chipseq/species/species*.conf and share with lab members or create your own species file.



### Using species file

There are many species specific parameters like indices (bwa, bowtie, ...), chromosome sizes and sequence files (chr*.fa). If you have multiple pipelines, it's hard to individually define all parameters in a command line argument (or in a configruation file) for each pipeline run. However, if you have a species file with all species specific parameters defined, then you define less parameters and share the species file with all other pipelines.

Add the following to the command line to specify species and species file.
```
-species [SPECIES; hg19, mm9, ...] -species_file [PATH_FOR_SPECIES_FILE]
```

You can override any parameters defined in the species file by adding them to command line argument or configuration file. For example, if you want to override parameters for BWA index and umap:
```
-species hg19 -species_file /.../species.conf -bwa_idx [YOUR_OWN_BWA_IDX] -chrsz [YOUR_OWN_CHR_SIZES_FILE]
```

Example species file looks like the following. You can define your own species.
```
[hg19]
chrsz   = /mnt/data/annotations/by_release/hg19.GRCh37/hg19.chrom.sizes // chrome sizes
seq     = /mnt/data/ENCODE/sequence/encodeHg19Male 			// genome reference sequence
gensz   = hs // genome size: hs for humna, mm for mouse
umap    = /mnt/data/ENCODE/umap/encodeHg19Male/globalmap_k1tok1000 	// uniq. mappability tracks
umap_hic= /mnt/data/ENCODE/umap/encodeHg19Male/globalmap_k20tok54 	// uniq. mappability tracks
bwa_idx = /mnt/data/annotations/indexes/bwa_indexes/encodeHg19Male/v0.7.10/encodeHg19Male_bwa-0.7.10.fa
bwt_idx = /mnt/data/annotations/indexes/bowtie1_indexes/encodeHg19Male/encodeHg19Male
bwt2_idx = /mnt/data/annotations/indexes/bowtie2_indexes/bowtie2/ENCODEHg19_male
vplot_idx = /mnt/data/annotations/indexes/vplot_indexes/hg19/parsed_hg19_RefSeq.merged.ANS.bed

# your own definition for species
[hg19_custom]
chrsz   = ...
seq     = ...
...

[mm9]
...

[mm10]
...
```

Description for parameters in a species file.
```
chrsz               : Chrome sizes file path (use fetchChromSizes from UCSC tools).
seq                 : Reference genome sequence directory path (where chr*.fa exist).
gensz               : Genome size; hs for human, mm for mouse (default: hs).
umap                : Unique mappability tracks directory path (https://sites.google.com/site/anshulkundaje/projects/mappability).
umap_hic            : Unique mappability tracks directory path (for HiC, DO NOT USE all-mappable umap track starting from 1bp)
bwa_idx             : BWA index (full path prefix of [].bwt file) .
bwt_idx             : Bowtie index (full path prefix of [].1.ebwt file).
bwt2_idx            : Bowtie2 index (full path prefix of [].1.bt2 file).
vplot_idx           : V plot index (full path for bed file).
```


### Useful HTML report for debugging

BigDataScript HTML report is located at the working folder with name [PIPELINE_NAME]_[TIMESTAMP]_report.html. This report is automatically generated by BigDataScript and is very useful for debugging since it shows summary, timeline, Stdout and Stderr for all jobs in the pipeline.



### Debugging pipelines (for advanced users)

Make BDS verbose.
```
$ bds -v [PIPELINE_BDS] ...
```

Display debugging information.
```
$ bds -d [PIPELINE_BDS] ...
```

Test-run (this actually does nothing) to check next stages and input/output file names for it.
```
$ bds -dryRun [PIPELINE_BDS] ...
```



### How to set shell environments (What are MOD, shcmd and addpath?)

Ignore this section if you are working on Kundaje lab clusters (just add a flag '-kundaje_lab' to the command line).

It is important to define enviroment variables (like $PATH) to make bioinformatics softwares in the pipeline work properly. MOD, shcmd and addpath are three convenient ways to define environment variables. Environment variables defined with MOD, shcmd and addpath are preloaded for all tasks on the pipeline. For example, if you define environment variables for bwa/0.7.3 with MOD. bwa of version 0.7.3 will be used throughout the whole pipeline (including bwa aln, bwa same and bwa sampe).

1) mod

There are different versions of bioinformatics softwares (eg. samtools, bedtools and bwa) and <a href="http://modules.sourceforge.net/">Enviroment Modules</a> is the best way to manage environemt variables for them. For example, if you want to add environment variables for bwa 0.7.3 by using Environment Modules. You can simply type the following:

```
$ module add bwa/0.7.3;
```

The equivalent setting in the pipeline configuration file should look like:
```
mod= bwa/0.7.3;
```

You can have multiple lines for MOD since any suffix is allowed. Use ; as a delimiter.
```
mod_bio= bwa/0.7.3; bedtools/2.x.x; samtools/1.2
mod_lang= r/2.15.1; java/latest
```

2) shcmd

If you have softwares locally installed on your home, you may need to add to them environment variables like $PATH, $LD_LIBRARY_PATH and so on. <b>IMPORTANT!</b> Note that any pre-defined enviroment variables (like $PATH) should be referred in a curly bracket like ${PATH}. This is because BDS distinguishes environment variables from BDS variables by a curly bracket ${}.
```
shcmd= export PATH=${PATH}:path_to_your_program
```

You can have multiple lines for shcmd since any suffix is allowed. Use ; as a delimiter. 
```
shcmr_R= export PATH=${PATH}:/home/userid/R-2.15.1;
shcmd_lib= export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:${HOME}/R-2.15.1/lib
```

shcmd is not just for adding environemt variables. It can execute any bash shell commands prior to any jobs on the pipeline. For example, to give all jobs peaceful 10 seconds before running.
```
shcmd_SLEEP_TEN_SECS_FOR_ALL_JOBS= echo "I am sleeping..."; sleep 10
```

3) addpath

If you just want to add something to your $PATH, use addpath instead of shcmd. It's much simpler. Use : or ; as a delimiter.

```
addpath= ${HOME}/program1/bin:${HOME}/program1/bin:${HOME}/program2/bin:/usr/bin/test
```


### What are -mod, -shcmd and -addpath?

They are command line argument versions of MOD, shcmd and addpath. For example,

```
$ bds [PIPELINE_BDS] -mod 'bwa/0.7.3; samtools/1.2' -shcmd 'export PATH=${PATH}:/home/userid/R-2.15.1' -addpath '${HOME}/program1/bin'
```



### Troubleshooting

If see the following error when you submit jobs to Sun Grid Enginee,
```
/bin/bash: module: line 1: syntax error: unexpected end of file
```

Check your $HOME/.bashrc if it has any errorneous lines.

Remove the following line in you module initialization scripts ($MODULESHOME/init/bash or /etc/profile.d/modules.sh).
```
export -f module
```




### Contributors

* Jin wook Lee - PhD Student, Mechanical Engineering Dept., Stanford University
* Anshul Kundaje - Assistant Professor, Dept. of Genetics, Stanford University
