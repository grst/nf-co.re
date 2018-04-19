# Linting Errors

This page contains detailed descriptions of the tests done by the [nf-core/tools](https://github.com/nf-core/tools) package. Linting errors should show URLs next to any failures that link to the relevant heading below.

## <a name="1"></a>Error #1 - File not found
nf-core pipelines should adhere to a common file structure for consistency. The lint test looks for the following required files:

* `nextflow.config`
    * The main nextflow config file
* `Dockerfile`
    * A docker build script to generate a docker image with the required software
* `.travis.yml` or `circle.yml`
    * A config file for automated continuous testing with either [Travis CI](https://travis-ci.org/) or [Circle CI](https://circleci.com/)
* `LICENSE`, `LICENSE.md`, `LICENCE.md` or `LICENCE.md`
    * The MIT licence. Copy from [here](https://raw.githubusercontent.com/nf-core/tools/master/LICENSE).
* `README.md`
    * A well written readme file in markdown format
* `CHANGELOG.md`
    * A markdown file listing the changes for each pipeline release
* `docs/README.md`, `docs/output.md` and `docs/usage.md`
    * A `docs` directory with an index `README.md`, usage and output documentation

The following files are suggested but not a hard requirement. If they are missing they trigger a warning:

* `main.nf`
    * It's recommended that the main workflow script is called `main.nf`
* `conf/base.config`
    * A `conf` directory with at least one config called `base.config`
* `tests/run_test.sh`
    * A bash script to run the pipeline test run


## <a name="2"></a>Error #2 - Docker file check failed
Pipelines should have a file called `Dockerfile` in their root directory.
This is used for automated docker image builds. This test checks that the file
exists and contains at least the string `FROM `.

## <a name="3"></a>Error #3 - Licence check failed
nf-core pipelines must ship with an open source [MIT licence](https://choosealicense.com/licenses/mit/).

This test fails if the following conditions are not met:

* No licence file found
    * `LICENSE`, `LICENSE.md`, `LICENCE.md` or `LICENCE.md`
* Licence file contains fewer than 4 lines of text
* File does not contain the string `without restriction`
* Licence contains template placeholders
    * `[year]`, `[fullname]`, `<YEAR>`, `<COPYRIGHT HOLDER>`, `<year>` or `<copyright holders>`

## <a name="4"></a>Error #4 - Nextflow config check failed
nf-core pipelines are required to be configured with a minimal set of variable
names. This test fails or throws warnings if required variables are not set.

> **Note:** These config variables must be set in `nextflow.config` or another config
> file imported from there. Any variables set in nextflow script files (eg. `main.nf`)
> are not checked and will be assumed to be missing.

The following variables fail the test if missing:

* `params.version`
    * The version of this pipeline. This should correspond to a [GitHub release](https://help.github.com/articles/creating-releases/).
* `params.nf_required_version`
    * The minimum version of Nextflow required to run the pipeline.
    * This should correspond to the `NXF_VER` version tested by Travis.
* `params.outdir`
    * A directory in which all pipeline results should be saved
* `manifest.description`
    * A description of the pipeline
* `manifest.homePage`
    * The homepage for the pipeline. Should be the nf-core GitHub repository URL,
      so beginning with `https://github.com/nf-core/`
* `timeline.enabled`, `trace.enabled`, `report.enabled`, `dag.enabled`
    * The nextflow timeline, trace, report and DAG should be enabled by default
* `process.cpus`, `process.memory`, `process.time`
    * Default CPUs, memory and time limits for tasks

The following variables throw warnings if missing:

* `manifest.mainScript`
    * The filename of the main pipeline script (recommended to be `main.nf`)
* `timeline.file`, `trace.file`, `report.file`, `dag.file`
    * Default filenames for the timeline, trace and report
    * Should be set to a results folder, eg: `${params.outdir}/pipeline_info/trace.[workflowname].txt"``
    * The DAG file path should end with `.svg`
        * If Graphviz is not installed, Nextflow will generate a `.dot` file instead
* `process.container`
    * A single default container for use by all processes
* `params.reads`
    * Input parameter to specify input data (typically FastQ files / pairs)
* `params.singleEnd`
    * Specify to work with single-end sequence data instead of default paired-end
    * Used with Nextflow: `.fromFilePairs( params.reads, size: params.singleEnd ? 1 : 2 )`

## <a name="5"></a>Error #5 - Continuous Integration configuration
nf-core pipelines must have CI testing with Travis or Circle CI.

This test fails if the following happens:

* `.travis.yml` does not contain the string `nf-core lint ${TRAVIS_BUILD_DIR}` under `script`
* `.travis.yml` does not test the Nextflow version specified in the pipeline as `nf_required_version`
    * This is expected in the `env` section of the config, eg:
    ```yaml
    env:
      - NXF_VER=0.27.0
      - NXF_VER=''
    ```
    * At least one of these `NXF_VER` variables must match the `params.nf_required_version` version specified in the pipeline config
    * Other variables can be specified on these lines as long as they are space separated.

## <a name="6"></a>Error #6 - Repository `README.md` tests
The `README.md` files for a project are very important and must meet some requirements:

* Nextflow badge
    * If no Nextflow badge is found, a warning is given
    * If a badge is found but the version doesn't match the minimum version in the config file, the test fails
    * Example badge code:
    ```markdown
    [![Nextflow](https://img.shields.io/badge/nextflow-%E2%89%A50.27.6-brightgreen.svg)](https://www.nextflow.io/)
    ```
* Bioconda badge
    * If your pipeline contains a file called `environment.yml`, a bioconda badge is required
    * Required badge code:
    ```markdown
    [![install with bioconda](https://img.shields.io/badge/install%20with-bioconda-brightgreen.svg)](http://bioconda.github.io/)
    ```

## <a name="7"></a>Error #7 - Pipeline and container version numbers

> This test only runs when `--release` is set or `$TRAVIS_BRANCH` is equal to `master`

These tests look at `params.container`, `process.container` and `$TRAVIS_TAG`, only
if they are set.

* Container name must have a tag specified (eg. `nfcore/pipeline:version`)
* Container tag / `$TRAVIS_TAG` must contain only numbers and dots
* Tags and `$TRAVIS_TAG` must all match one another


## <a name="8"></a>Error #8 - Conda environment tests

> These tests only run when your pipeline has a root file called `environment.yml`

* The environment `name` must match the pipeline name and version
    * The pipeline name is found from the Nextflow config `manifest.homePage`,
      which assumes that the URL is in the format `github.com/nf-core/[pipeline-name]`
    * Example: For `github.com/nf-core/test` version 1.4, the conda environment name should be `nfcore-test-1.4`

Each dependency is checked using the [Anaconda API service](https://api.anaconda.org/docs).
Dependency sublists are ignored with the exception of `- pip`: these packages are also checked
for pinned version numbers and checked using the [PyPI JSON API](https://wiki.python.org/moin/PyPIJSON).

Note that conda dependencies with pinned channels (eg. `conda-forge::openjdk`) are fine
and should be handled by the linting properly.

Each dependency can have the following lint failures and warnings:

* (Test failure) Dependency does not have a pinned version number, eg. `toolname=1.6.8`
* (Test failure) The package cannot be found on any of the listed conda channels (or PyPI if `pip`)
* (Test warning) A newer version of the package is available

## <a name="9"></a>Error #9 - Dockerfile for use with Conda environments

> This test only runs if there is both `environment.yml`
> and `Dockerfile` present in the workflow.

If a workflow has a conda `environment.yml` file (see above), the `Dockerfile` should use this
to create the container. Such `Dockerfile`s can usually be very short, eg:

```Dockerfile
FROM nfcore/base
LABEL authors="your@email.com" \
      description="Docker image containing all requirements for nf-core/EXAMPLE pipeline"

COPY environment.yml /
RUN conda env create -f /environment.yml && conda clean -a
ENV PATH /opt/conda/envs/nfcore-EXAMPLE/bin:$PATH
```

To enforce this minimal `Dockerfile` and check for common copy+paste errors, we require
that the above template is used.
Failures are generated if the `FROM`, `COPY`, `RUN` and `ENV` statements above are not present.
These lines must be an exact copy of the above example, with the exception that
the `ENV PATH` must reference the name of your pipeline instead of `nfcore-EXAMPLE`.

Additional lines and different metadata can be added without causing the test to fail.