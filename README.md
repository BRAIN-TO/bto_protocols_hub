# BRAIN-TO MRI Protocols

Optimised MRI acquisition protocols for Siemens 3 T and 7 T systems running
XA60.

> 🔴 **Active development:** This repository may change without notice. Consider
> it complete only when this banner has been removed.

## Table of contents

- [Protocol collections](#protocol-collections)
- [Motivation](#motivation)
- [Design principles](#design-principles)
- [What this repository contains](#what-this-repository-contains)
- [Recommended workflow after data acquisition](#recommended-workflow-after-data-acquisition)
- [bto-docker](#bto-docker)
- [Help and issue reporting](#help-and-issue-reporting)
- [Use in research](#use-in-research)
- [Relevant publications](#relevant-publications)
- [Contributions](#contributions)
- [Previous version](#previous-version)
- [Contact](#contact)

## Protocol collections

- [3 T XA60 protocols](3T/README.md)
- [7 T XA60 protocols](7T/README.md)

Each page describes its collection and will link to the matching Zenodo package.
The 3 T collection includes anatomy, fMRI, diffusion, and ASL. The 7 T
collection includes anatomy, fMRI, and diffusion.

The 7 T collection does not include a product ASL protocol. To discuss 7 T ASL
options, reach out via email (see end of page).

## Motivation

Protocol fragmentation is a quiet crisis in neuroimaging research.
Standardisation efforts are not keeping pace with the need, especially in
clinical research. Scanner time and research funding are also under increasing
pressure, while data are being reused more often. A control group acquired for
one study may support another study, and datasets collected at different sites
may be combined.

That depends on acquisition parameters being shared and understood. In
practice, protocols often vary between studies at the same site. Small changes
in resolution, timing, acceleration, reconstruction, or coil configuration can
make data harder to compare and can affect later processing. The problem is
larger in multi-site studies, where local scanner setups and established
practices already differ.

Access is another constraint. Hospitals and research groups may not be able to
obtain or maintain specialised research sequences. BRAIN-TO uses Siemens
product sequences so that collaborating sites can implement the protocols
without Work-in-Progress packages or C2Ps.

BRAIN-TO provides a stable starting point for researchers, MR physicists, and
technologists who need dependable protocols without designing every sequence
from scratch. The aim is consistent performance and practical reuse, rather
than pushing individual scanner settings to their limit.

## Design principles

- Siemens product sequences make the protocol sets practical to share across
  sites.
- Isotropic voxels support consistent spatial sampling and reduce
  partial-voluming concerns.
- Most scans target 5 to 6 minutes at 3 T and 6 to 8 minutes at 7 T (exceptions
  exist).
- Coil-specific variants use the acceleration available from the head coil.
- Data acquired with BRAIN-TO protocols have been evaluated with
  community-standard MRI processing software.

## What this repository contains

- [Protocol index and compatibility reference](PROTOCOLS.md)
- [3 T helper scripts](3T/scripts/README.md)
- [7 T helper scripts](7T/scripts/README.md)
- GitHub issue forms for protocol, acquisition, and processing support

Zenodo is the single source for downloadable protocol packages and their full
acquisition parameters. The repository provides an index, practical guidance,
and links to related tools.

## Recommended workflow after data acquisition

1. Use [dichotomise](https://github.com/srikash/dichotomise) to check, sort,
   rename, de-identify, and archive DICOM exports from Siemens XA60+ systems.
2. Convert the organised data to BIDS with
   [BIDScoin](https://github.com/Donders-Institute/bidscoin).
3. Use [bto-docker](https://github.com/srikash/bto-docker), a general-purpose
   BTO protocols processing container with FSL, oxasl, FreeSurfer, and
   supporting analysis scripts for ASL perfusion and structural MRI.

Each repository documents its supported inputs, versions, and release
locations. Check its README before using it with a BRAIN-TO dataset.

## bto-docker

General-purpose processing container for BTO protocols.

### Included tools

| Tool | Version |
|---|---|
| FSL | 6.0.7.22 |
| FreeSurfer | 8.2.0 |
| oxasl | custom |
| ANTs | 2.6.5 |
| AFNI | latest |
| MRtrix3 | latest |

### Quick start

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

### CLI commands

| Command | Purpose |
|---|---|
| `bto-asl` | ASL perfusion pipeline |
| `bto-anat fsl` | FSL structural preprocessing |

#### bto-asl — ASL perfusion pipeline

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

##### Key flags

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

#### bto-anat — structural preprocessing

##### FSL

```bash
docker run --rm -v /path/to/data:/data \
    brainto/bto-docker bto-anat fsl \
    --t1 /data/sub-01/anat/T1w.nii.gz
```

Runs bias correction, brain extraction (`mri_synthstrip`), linear and nonlinear MNI registration, FAST tissue segmentation, and subcortical segmentation. Output defaults to `<t1_parent>/derivatives/anat/<t1_stem>.anat/`.

### Notes

- FreeSurfer license is never baked into the image — mount it at runtime to `/opt/fs-license/license.txt`
- Base image: `ubuntu:24.04`
- All scripts use FSL's conda Python (`/usr/local/fsl/bin/python3`) as the runtime interpreter
- Default Docker runs as root; pass `--user $(id -u):$(id -g)` to write output as the current user

### Dev

[github.com/srikash/bto-docker](https://github.com/srikash/bto-docker)

## Help and issue reporting

Use the issue form that matches the problem:

- [Protocol question or problem](../../issues/new?template=protocol.yml)
- [Acquisition problem](../../issues/new?template=acquisition.yml)
- [Processing question or problem](../../issues/new?template=processing.yml)

## Use in research

Before using a protocol, confirm that it is appropriate for the local scanner
configuration, coil, participant group, safety procedures, ethics approval,
and study question. Local MR physics and technologist review remains essential.

## Relevant publications

![](misc/fig/MAGNETOM_Flash_Figure_2.png)

- Kashyap S, Uludağ K. *Advancing Clinical and Neuroscientific Research
  Through Accessible and Optimized Protocol Design at 3T.* Siemens MAGNETOM
  Flash, RSNA Edition, 2023.
  [Read the article](https://marketing.webassets.siemens-healthineers.com/ed15b22a01ec5497/ef408bcafa80/siemens-healthineers-magnetom-world-Kashyap_Uludag_BRAIN-TO_protocols.pdf)
- Kashyap, S. (2026). *Advancing clinical neuroscience research through
  accessible, optimised protocol design at 3 and 7 T.* Zenodo. Annual Meeting
  of the Organization for Human Brain Mapping (OHBM), Marseille, France.
  [https://doi.org/10.5281/zenodo.22769256](https://doi.org/10.5281/zenodo.22769256)

## Contributions

Contributions are welcome. Useful contributions include protocol feedback,
implementation experience at another site, field-specific scripts, processing
pipelines, and documentation improvements. Please open an issue before
proposing a substantial change.

## Previous version

The complete XA30 protocol set is available on the
[`xa30_archive`](../../tree/xa30_archive) branch.

## Contact

For questions about protocol selection, implementation, or 7 T ASL,
email [sriranga.kashyap@utoronto.ca](mailto:sriranga.kashyap@utoronto.ca).
