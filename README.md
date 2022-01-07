# Banich Lab fMRI Preprocessing Pipeline
Custom preprocessing pipeline built using FMRIB Software Library (FSL) v6.0 for functional neuroimaging. The pipeline is built using fsl best practices and conforms to BIDS format.  

Contents:
  - [Workflow Summary](#workfolw-summary)
  - [Installation](#installation)
  - [Bids Format](#bids-format)
  - [Command-Line Arguments](#command-line-arguments)
  - [Examples](#docker-examples)
    - [Docker](#docker)
    - [Singularity](#singularity)
- [Known Issues](#known-issues)

# Workflow Summary
_fmripreprpoc_ is a single command line tool used to run a standard fsl based preprocessing scheme for functional neuroimaging data. Key components of the pipeline include: skull stripping and tissue segmentation, fielmap estimation and distortion correction, motion correction, outlier detection, trimming, normalization to standard space, and AROMA denoising. 

[image]!fmripreproc workflow diagram

"short methods section..."


```bash
    fMRI Preprocessing Pipeline
        Usage:  --in=<bids-inputs> --out=<outputs> --participant-label=<id> [OPTIONS]
        OPTIONS
          --help                      show this usage information and exit
          --participant-label=        participant name for processing (pass only 1)
          --work-dir=                 (Default: <outputs>/scratch/particiant-label) directory 
                                        path for working directory
          --clean-work-dir=           (Default: TRUE) clean working directory 
          --trim-vols=                (Default: 10) trim inital volumes from all bold scans
          --dummy-scans=              (Default: 10) add dummy scan indicator variables in confounds
                                        file. DO NOT use with "trim-vols"
          --outliers-fd=              (Default: 0.9mm) generate indicator variables for framewise
                                        displacement outliers above given threshold
          --outliers-dvars=           (Default: XX) generate indicator variables for dvars outliers
                                        above given threshold
          --run-qc                    add flag to run automated quality 
                                        control for preprocessing
          --run-aroma                 add flag to run aroma noise removal on 
                                        preprocessed images
          --run-fix (?)               add flag to run fsl-fix noise removal on 
                                        preprocessed images
    ** OpenMP used for parellelized execution of XXX. Multiple cores (CPUs) 
       are recommended (XX cpus for each fmri scan).
       
    ** see github repository for more information and to report issues: 
       https://github.com/intermountainneuroimaging/fmri-preproc.git
```


# Installation

Use the _fsl-fdt_ Docker image (recommended):

```shell
docker run --rm amhe4269/fsl-fdt:0.0.1 --help
```

Use the _fsl-fdt_ Singularity image (recommended):

```shell
singularity pull fsl-fdt docker://amhe4269/fsl-fdt:0.0.1
singularity run amhe4269/fsl-fdt:0.0.1 --help
```
# BIDS format
The fsl-fdt workflow takes advantage of the BIDS naming convention and supporting metadata. The input data must be in a valid BIDS format, and include at least one dwi image with accompanying bval and bvec files. Metadata including readoutime must be including in a json sidecar file for each dwi image. See [dcm2niix](https://github.com/rordenlab/dcm2niix) and [BIDS Validator](https://bids-standard.github.io/bids-validator/) for more details. 

# Command-Line Arguments

```
usage: fsl-fdt [-h] [-i INPUT_PATH] [-o OUTPUT_PATH]
               [--participant-label= ID]
               [--work-dir= SCRATCH_PATH]
               [--clean-work-dir= {TRUE,FALSE}]
               [--concat-preproc= {TRUE,FALSE}]
               [--run-qc= {TRUE,FALSE}]
               [--studyname= ACCOUNT]

optional arguments:
  -h, --help                          show this help message and exit
  -i INPUT_PATH, --in= INPUT_PATH     (required) BIDS input directory path
  -o OUTPUT_PATH, --out= OUTPUT_PATH  (required) BIDS output / derivatives directory 
  --participant-label= ID             (required) participant label for processing
  --work-dir= SCARTCH_PATH            select working directory for analysis (DEFAULT: /scratch)
  --clean-work-dir= {TRUE,FALSE}      flag used to define if working directory should be cleared after execution (DEFAULT: TRUE)
  --concat-preproc= {TRUE,FALSE}      flag used to select if all dwi images should be concatinated before correction (DEFAULT: FALSE)
  --run-qc= {TRUE,FALSE}              flag set to include EDDY_QC (DEFAULT: TRUE)
  --run-tensor-fit                    add flag to run tensor-fit processing on preprocessed images
  --run-bedpostx                      add flag to run bedpostx tractography processing on preprocessed images (default settings used for analysis)
  
** OpenMP used for parellelized execution of eddy. Multiple cores (CPUs) are recommended (4 cpus for each dwi scan).

Docker entrypoint: FDTPipeline.py
```
> ### Important
> If running multiple instances of fsl-fdt, you _MUST_ create a unique working directory for each instance to avoid loop contamination.

# Running _fsl-fdt_ using Docker Engine
This pipeline is built with the intented to be used with docker or singularity engines. Compiled in the docker image includes all python packages and FSL version (6.0.3) for the pipeline.
```shell
$ docker run -ti --rm \
    -v path/to/data:/data:ro \
    -v path/to/output:/out \
    amhe4269/fsl-fdt:<latest-version> /data /out --participant-label=[ID] [OPTIONS] 
```

## Docker examples

The canonical examples install ANTs version 2.3.1 on Debian 9 (Stretch).

_Note_: Do not use the `-t/--tty` flag with `docker run` or non-printable characters will be a part of the output (see [moby/moby#8513 (comment)](https://github.com/moby/moby/issues/8513#issuecomment-216191236)).

### Docker
Run the Diffusion Toolbox pipleine using docker. We first mount appropriate volumes (external directories) and assign relevant arguments including participant-label.
```shell
$ docker run -ti --rm \
    -v $studydir/BIDS:/data:ro \
    -v $studydir/ANALYSIS:/out \
    -v $studydir/tmp/ds005-workdir:/work \
    amhe4269/fsl-fdt:<latest-version> \
    /data /out/ --participant-label=0001 --work-dir /work --clean-work-dir=FALSE
```

### Singularity
Run the Diffusion Toolbox pipleine using singularity. We first mount appropriate volumes (external directories) and assign relevant arguments including participant-label.
```shell
$ singularity run 
    -bind $studydir/BIDS:/data:ro \
    -bind $studydir/ANALYSIS:/out \
    -bind $studydir/tmp/ds005-workdir:/work \
    amhe4269/fsl-fdt:<latest-version> \
    /data /out/ --participant-label=0001 --work-dir /work --clean-work-dir=FALSE
```

# Known Issues
Working directory must be explicitly defined (in sperate locations) if running multiple instances of fsl-fdt pipeline on the same computational resources.
