# BRAIN-To MRI Protocols Repository
This is a repository of wiki, code and containers for processing data acquired using the BRAIN-TO imaging protocols on Siemens MAGNETOM Prisma 3 T (XA30).

Please refer the article titled, [Advancing Clinical and Neuroscientific Research Through Accessible and Optimized Protocol Design at 3T](https://marketing.webassets.siemens-healthineers.com/ed15b22a01ec5497/ef408bcafa80/siemens-healthineers-magnetom-world-Kashyap_Uludag_BRAIN-TO_protocols.pdf), published in the RSNA Edition of Siemens MAGNETOM Flash (2023) for an overview.

![](misc/fig/MAGNETOM_Flash_Figure_2.png)

#### Download Links for BRAIN-TO Protocols
1. Complete Set <br /> [https://zenodo.org/records/10685481](https://zenodo.org/records/10685481)

2. Anatomy Set <br /> [https://zenodo.org/records/10685475](https://zenodo.org/records/10685475)

3. fMRI Set <br /> [https://zenodo.org/records/10685458](https://zenodo.org/records/10685458)

4. Diffusion Set <br /> [https://zenodo.org/records/10685449](https://zenodo.org/records/10685449)

5. ASL Set <br /> [https://zenodo.org/records/10685308](https://zenodo.org/records/10685308)

#### This repository is being updated steadily and include the following 
1. ~~Zenodo links for XA30 exar1 files~~
2. ~~Zenodo links for Protocol PDFs (for translating to other Prisma software versions)~~
3. Wiki with recommended pipelines and tools
4. Adding scripts and example code for processing data

#### Please feel free to contribute to our efforts. 
For urgent requests, please [e-mail](mailto:sriranga.kashyap@uhn.ca).

#### Contributors
Sriranga Kashyap, BRAIN-To Lab, Krembil Brain Institute, UHN<br />
Yuexin Xi, PhD Student, Dept. of Medical Biophysics, University of Toronto

---
# bto-docker

General-purpose processing container for BTO protocols.


---

## Included tools

| Tool | Version |
|---|---|
| FSL | 6.0.7.22 |
| FreeSurfer | 8.2.0 |
| oxasl | custom |
| ANTs | 2.6.5 |
| AFNI | latest |
| MRtrix3 | latest |

---

## Quick start

```bash
docker pull brainto/bto-docker:latest
```

Mount your data directory to `/data`:

```bash
docker run --rm \
    --user $(id -u):$(id -g) \
    --cpuset-cpus "0-9" \
    -v /path/to/data:/data \
    brainto/bto-docker \
    bto-asl --help
```

FreeSurfer requires a valid license file mounted at runtime:

```bash
docker run --rm \
    -v /path/to/data:/data \
    -v /path/to/license.txt:/opt/fs-license/license.txt \
    brainto/bto-docker \
    bto-anat freesurfer --help
```

All file paths must be absolute paths as seen from inside the container (i.e. under `/data`).

---

## CLI commands

| Command | Purpose |
|---|---|
| `bto-asl` | ASL perfusion pipeline |
| `bto-anat fsl` | FSL structural preprocessing |

## bto-asl — ASL perfusion pipeline

End-to-end pipeline.

```bash
docker run --rm -it \
    --user $(id -u):$(id -g) \
    --cpuset-cpus "0-9" \
    -v $PWD:/data \
    brainto/bto-docker bto-asl \
    --subjid  sub-01_ses-rest \
    --asl     /data/sub-01/ses-rest/perf/sub-01_ses-rest_asl.nii.gz \
    --t1      /data/sub-01/ses-anat/anat/sub-01_T1w.nii.gz \
    --m0-fwd  /data/sub-01/ses-rest/fmap/sub-01_ses-rest_dir-PA_m0scan.nii.gz \
    --m0-rev  /data/sub-01/ses-rest/fmap/sub-01_ses-rest_dir-AP_m0scan.nii.gz
```

JSON sidecars (`.json` alongside each NIfTI) are auto-detected for acquisition parameters. Pass `--no-json` to skip sidecar detection and use assumed PE direction.

If structural preprocessing has already been run, pass `--anatdir` to reuse the existing `.anat` directory:

```bash
    --anatdir /data/sub-01/ses-anat/anat/sub-01_T1w.anat
```

### Key flags

| Flag | Default | Description |
|---|---|---|
| `--subjid` | required | Subject ID — names the output directory `bto-asl_<subjid>/` |
| `--asl` | required | 4D ASL NIfTI (repeat for multiple runs) |
| `--t1` | — | T1w structural NIfTI — triggers FSL structural preprocessing |
| `--anatdir` | — | Existing `.anat` directory — skips structural preprocessing |
| `--m0-fwd` | — | Forward-PE M0 calibration image |
| `--m0-rev` | — | Reverse-PE M0 for topup distortion correction |
| `--outdir` | alongside `--asl` | Base output directory |
| `--no-json` | off | Skip JSON sidecar requirement |
| `--pe-dir` | `j` | Forward PE direction when `--no-json` is active |

---

## bto-anat — structural preprocessing

### FSL

```bash
docker run --rm -v /path/to/data:/data \
    brainto/bto-docker bto-anat fsl \
    --t1 /data/sub-01/anat/T1w.nii.gz
```

Runs bias correction, brain extraction (`mri_synthstrip`), linear and nonlinear MNI registration, FAST tissue segmentation, and subcortical segmentation. Output defaults to `<t1_parent>/derivatives/anat/<t1_stem>.anat/`.

## Notes

- FreeSurfer license is never baked into the image — mount it at runtime to `/opt/fs-license/license.txt`
- Base image: `ubuntu:24.04`
- All scripts use FSL's conda Python (`/usr/local/fsl/bin/python3`) as the runtime interpreter
- Default Docker runs as root; pass `--user $(id -u):$(id -g)` to write output as the current user

---

## Dev

[github.com/srikash/bto-docker](https://github.com/srikash/bto-docker)
