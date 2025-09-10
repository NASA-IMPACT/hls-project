## hls-project
This document provides an overview of code and artifacts used by the Harmonized Landsat Sentinel (HLS) project.  For more detailed information about the HLS product specification and distribution consult the LPDAAC [product landing page](https://lpdaac.usgs.gov/products/hlss30v002/)

The initial development goal of the HLS project was to expand existing, experimental HLS scientific algorithm code to a full scale global production pipeline running on scalable AWS infrastructure.  Due to the nature of the project and the potential for a large number of components, an early decision was made to use individual repositories for code management rather than a monorepo.  This provides the advantage of clear traceability and narrative of a component's development over time by reviewing the repository's commit history.  The disadvantage of this approach is the large number of repositories with no clear map of how they are interelated.  This document provides this map of how components are interelated and how they interoperate.


### Containers
The core of the HLS processing pipeline is algorithmic C code packaged as Docker containers.  Because different scientific libraries utilized in these containers share common dependencies, we use a simple hierarchical image dependency graph.

![Alt text](/docs/docker.png)

- [espa-dockerfiles](https://github.com/NASA-IMPACT/espa-dockerfiles)
Provides the core library dependencies used by all of our project images.  Specifically, we utilize `centos.external` as our primary base image.

- [hls-base](https://github.com/NASA-IMPACT/hls-base) - Provides the externally developed C/Python/Matlab libraries used for atmospheric correction and cloud masking in both our HLS S30 and L30 pipelines.  These include
    - [espa-product-formatter](https://github.com/NASA-IMPACT/espa-product-formatter) - An ESPA metadata and format conversion utility.
    - [espa-surface-reflectance](https://github.com/NASA-IMPACT/espa-surface-reflectance) - The C implementation of the LaSRC surface reflectance algorithm.
    - [espa-python-library](https://github.com/NASA-IMPACT/espa-python-library) - A Python library for manipulating and validating ESPA metadata.
    - [Fmask](https://github.com/GERSL/Fmask) - The Matlab implementation of the Fmask cloud masking algorithm.

- [hls-sentinel](https://github.com/NASA-IMPACT/hls-sentinel) - Uses internally developed C libraries and utilities for generating HLS S30 products from Sentinel 2 inputs.
- [hls-landsat](https://github.com/NASA-IMPACT/hls-landsat) - Uses internally developed C libraries and utilities for generating intermediate surface reflectance proudcts from Landsat inputs.
- [hls-landat-tile](https://github.com/NASA-IMPACT/hls-landsat-tile) - Provides the internally devloped C libraries and utilities for generating tiled HLS L30 proudcts from intermediate surface reflectance proudcts.
- [hls-laads](https://github.com/NASA-IMPACT/hls-laads) - Uses C utilities from [espa-surface-reflectance](https://github.com/NASA-IMPACT/espa-surface-reflectance) to download and synchronize required auxilary data from the LAADS DAAC.
- [hls-vi-historical] - Computes HLS-VI product from existing, historical HLS granules using tools from [hls-vi].

### Utilities
Generating HLS proudcts requires a suite of additional metadata and secondary files for ingestion into external systems such as [CMR](https://earthdata.nasa.gov/eosdis/science-system-description/eosdis-components/cmr), [Cumulus](https://nasa.github.io/cumulus/docs/cumulus-docs-readme) and [GIBs](https://earthdata.nasa.gov/eosdis/science-system-description/eosdis-components/gibs).  These Python CLI utilities are installed and used from within the containers.

- [hls-metadata](https://github.com/NASA-IMPACT/hls-metadata) - Generate CMR metadata for HLS products.

- [hls-cmr_stac](https://github.com/NASA-IMPACT/hls-cmr_stac) - Convert CMR metadata to STAC metadata for HLS products.

- [hls-utilities](https://github.com/NASA-IMPACT/hls-utilities) - A suite of utilities used by HLS processing to read and manipulate Sentinel and Landsat product specific file formats.

- [hls-browse_imagery](https://github.com/NASA-IMPACT/hls-browse_imagery) - Create GIBs browse imagery for HLS products.

- [hls-hdf_to_cog](https://github.com/NASA-IMPACT/hls-hdf_to_cog) - Convert internal HLS hdf product formats to [COGs](https://www.cogeo.org/) for distribution.

- [hls-manifest](https://github.com/NASA-IMPACT/hls-manifest) - Generate Cumulus CNM messages to be used for LPDAAC ingestion.

- [hls-thumbnails](https://github.com/NASA-IMPACT/hls-thumbnails) - Generate reduced resolution, true color thumbnails for HLS products.

- [hls-testing_data](https://github.com/NASA-IMPACT/hls-testing_data) - Not an actual utility but a suite of sample HLS products used for integration testing utilities which must read directly from the file format.

- [hls-vi] - Scripts and functions for computing HLS-VI vegetation index products from HLS bands.

### Static lookup files
The HLS pipeline relies on several static lookup files generated by the scientific team.  To support full process reproducibility, the code used to generate these files is openly maintained.

- [hls-L8S2overlap](https://github.com/NASA-IMPACT/hls-L8S2overlap) - Generates a lookup file of MGRS Landsat Path Row intersections clipped to the HLS data processing boundaries.

- [hls-land_tiles](https://github.com/NASA-IMPACT/hls-land_tiles) - Generates a lookup file of valid MGRS land tiles used to trim the MGRS Landsat Path Row overlap file.

### Infrastructure and Orchestration
These repositories define the infrastructure as code and AWS components which manage the flow of data through the HLS procesing pipelines.

- [hls-orchestration](https://github.com/NASA-IMPACT/hls-orchestration) - The core HLS processing infrastructure which receives notifcations for new Sentinel 2 and Landsat data to process and generates HLS S30 and L30 products.

- [hls-sentinel2-downloader-serverless](https://github.com/NASA-IMPACT/hls-sentinel2-downloader-serverless) - Monitors new publications and continually downloads Sentinel 2 data from the [ESA International Access Hub](https://inthub.copernicus.eu/) in near real time.

- [hls-landsat-historic](https://github.com/NASA-IMPACT/hls-landsat-historic) - Sends date range listings of historical Landsat data from the USGS S3 archive to [hls-orchestration](https://github.com/NASA-IMPACT/hls-orchestration) for incremental archival processing.

- [hls-lpdaac](https://github.com/NASA-IMPACT/hls-lpdaac) - Sends HLS product CNM messages to LPDAAC's Cumulus queue to trigger ingest.

- [hls-vi-historical-orchestration] - Orchestrates running [hls-vi-historical] container to backfill the HLS-VI product for historical time periods.


[hls-vi]: https://github.com/NASA-IMPACT/hls-vi
[hls-vi-historical]: https://github.com/NASA-IMPACT/hls-vi-historical
[hls-vi-historical-orchestration]: https://github.com/NASA-IMPACT/hls-vi-historical-orchestration
