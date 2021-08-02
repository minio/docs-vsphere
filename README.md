# MinIO Documentation for VMware Cloud Foundation

This is the source repository for the MinIO documentation on the 
VMware Cloud Foundation 4.2 MinIO plugin at https://docs.min.io/minio/vsphere.

This project uses Sphinx and a Gulp.js scss compilation pipeline to build a
static site for hosting on http:/

Contributions are welcome. 

# Build Instructions

MinIO uses [Sphinx](https://www.sphinx-doc.org/en/master/index.html) to generate
static HTML pages using ReSTructured Text.

## Prerequisites

- Python 3.6.0 or later. MinIO uses the latest stable version of Python for regular writing and development work.

- NodeJS 14.5.0 or later.

- `git` or a git-compatible client.

- Access to https://github.com/minio/docs-vsphere

All instructions below are intended for Linux systems. Windows builds may work
using the following instructions as general guidance.

## Build Process

1. Run `git checkout https://github.com/minio/docs-vsphere` and `cd docs-vsphere` to move into
   the working directory.

2. Create a new virtual environment `python -3 m venv venv`. Activate it using
   `source venv/bin/activate`.

3. Run `pip install -r requirements.txt` to setup the Python environment.

4. Run `make html`

5. Run `python -m http.server --directory build/BRANCH/html`. Open your
   browser to the output URL to view the staged output.

This project is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/legalcode).

See [CONTRIBUTIONS](https://github.com/minio/docs/tree/master/CONTRIBUTIONS.md) for more information on contributing content to the MinIO Documentation project.
