# MinIO Documentation for VMware Cloud Foundation

This is the source repository for the MinIO documentation on the 
VMware Cloud Foundation 4.2 MinIO plugin.

This project uses Sphinx and a Gulp.js scss compilation pipeline to build a
static site for hosting on docs.min.io. 

Contributions are welcome. 

## Requirements

- python3 (any version should be fine, latest is ideal)
- suggest creating a virtual environment to keep things clean and simple:
- start by cloning the repository. `cd` into the cloned repo. You might need to `git fetch` to refresh the repo.
- once in the repository root, run the following.

```shell
python3 -m venv venv
```

```shell
source venv/bin/activate
```

```shell
pip3 install -r requirements.txt
```

To make the build, do:

```shell
make html
```

Artifacts output to the `build/` directory as HTML. Just open `index.html` to get started poking around.

If you modify things, I suggest doing clean builds to make sure all artifacts are fresh:

```shell
rm -rf build/ && make html
```

The easiest way to view the docs is to use `python -m http.server`:

```shell
python -m http.server --directory build/<branch>/html
```

If you're modifying any SCSS file, run `gulp watch` in a terminal *before*
saving the SCSS to re-compile all css.

Still need to work out deployment steps, but this should work for testing locally.

The `source` directory contains all of the source files, including css and js.

The `sphinxext` directory contains some python stuff related to the custom directive/roles, and its a rats nest right now.