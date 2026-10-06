# BRAIN-TO MRI Protocols

Optimised MRI acquisition protocols for Siemens 3 T and 7 T systems running
XA60.

> 🔴 **Active development:** This repository may change without notice. Consider
> it complete only when this banner has been removed.

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
