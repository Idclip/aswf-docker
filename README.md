# Docker Images for the Academy Software Foundation
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black) ![pytest coverage](https://img.shields.io/azure-devops/coverage/academysoftwarefoundation/Academy%20Software%20Foundation/2/master)

| Docker Images | Build Status |
| ---:         |     :---      |
| `aswf/ci-*`        | [![Build Status - aswf](https://dev.azure.com/academysoftwarefoundation/Academy%20Software%20Foundation/_apis/build/status/AZP%20aswf-docker?branchName=master)](https://dev.azure.com/academysoftwarefoundation/Academy%20Software%20Foundation/_build/latest?definitionId=2&branchName=master)
| `aswftesting/ci-*` | [![Build Status - aswf](https://dev.azure.com/academysoftwarefoundation/Academy%20Software%20Foundation/_apis/build/status/AZP%20aswf-docker?branchName=testing)](https://dev.azure.com/academysoftwarefoundation/Academy%20Software%20Foundation/_build/latest?definitionId=2&branchName=testing)


More information:
* [VFXPlatform](https://vfxplatform.com)
* [ASWF](https://aswf.io)

Changes are documented [here](CHANGELOG.md)

## CI Images

These images are for Continuous Integration testing of various project managed by the ASWF.
Each image (apart from `ci-common`) is available for multiple VFX Platform Years.

* `aswf/ci-common:1`: A base CentOS-7 image with devtoolset-6 (GCC-6.3.1), clang-7 and cuda.
* `aswf/ci-common:2`: A base CentOS-7 image with devtoolset-9.1 (GCC-9.3.1), clang-10 and cuda.
* `aswf/ci-base:20XX`: Based on `aswf/ci-common` with most most VFX Platform requirements pre-installed.
* `aswf/ci-openexr:20XX`: Based on `aswf/ci-common`, comes with all OpenEXR upstream dependencies pre-installed.
* `aswf/ci-ocio:20XX`: Based on `aswf/ci-common`, comes with all OpenColorIO upstream dependencies pre-installed.
* `aswf/ci-opencue:20XX`: Based on `aswf/ci-common`, comes with all OpenCue upstream dependencies pre-installed.
* `aswf/ci-openvdb:20XX`: Based on `aswf/ci-common`, comes with all OpenVDB upstream dependencies pre-installed.
* `aswf/ci-usd:20XX`: Based on `aswf/ci-common`, comes with all USD upstream dependencies pre-installed.
* `aswf/ci-osl:20XX`: Based on `aswf/ci-common`, comes with all OpenShadingLanguage upstream dependencies pre-installed.
* `aswf/ci-vfxall:20XX`: Based on `aswf/ci-common`, comes with most VFX packages pre-installed.

### Versions

The `VFXPLATFORM_VERSION` is the calendar year mentioned in the VFX Platform, e.g. `2019`.

The `ASWF_VERSION` is a semantic version made of the `VFXPLATFORM_VERSION` as the major version number, and a minor version number to indicate minor changes in the Docker Image that still point to the same calendar year version, e.g. `2019.0` would be followed if necessary by a `2019.1` version.
The minor version here does *not* point to a calendar month or quarter, it is solely to express that the image has changed internally. We could also have a patch version.

### Image Tags

The most precise version tag is the `ASWF_VERSION` of the image, e.g. `aswf/ci-base:2019.0`, but it is recommended to use the `VFXPLATFORM_VERSION` as the tag to use in CI pipelines, e.g. `aswf/ci-openexr:2019`.

The `latest` tag is pointing to the current VFX Platorm year images, e.g. `aswf/ci-openexr:latest` points to `aswf/ci-openexr:2019.0` but will be updated to point to `aswf/ci-openexr:2020.0` in the calendar year 2020.


## Testing Images

There is another dockerhub organisation with copies of the `aswf` docker images called `aswftesting`, images published there are for general testing and experimentations. Images can be pushed by any fork of the official repo as long as the branch is called `testing`. Images in this org will change without notice and could be broken in many unexpected ways!

To get write access to the `aswftesting` dockerhub organisation you can open a jira issue [there](jira.aswf.io).

### Status
As of June 2020 there are full 2018, 2019 and 2020 [VFX Platform](https://vfxplatform.com) compliant images. And a preview of 2021
with devtoolset-9.1 (GCC-9.3.1) and clang-10, N.B. that the built-in CUDA-10.2 is currently incompatible with GCC-9, hoping to update to
CUDA-11 as soon as possible to restore CUDA support in 2021 images.


## CI Packages

In order to decouple the building of packages (which can take a lot of time, such as clang, Qt and USD) from the management of the CI Images, the packages are built and stored into "scratch" docker images that can be "copied" into the CI images at image build time by Docker.
Storing these CI packages into docker images has the additional benefit of being completely free to store on the docker hub repository.
The main negative point about this way of storing build artifacts is that tarballs are not available directly to download. It is very trivial to generate one and the provided `download-package.sh` script can be used to generate a local tarball from any package.

Also, CI packages are built using experimental docker syntax that allows cache folders to be mounted at build time, and is built with `docker buildx`. The new Docker BuildKit system allows the building of many packages in parallel in an efficient way with support for [ccache](https://ccache.dev/).

## Python Utilities

Check [aswfdocker](python/README.md) for python utility usage.

## Manual Builds
To build packages and images locally follow the instructions to install the [aswfdocker](python/README.md) python utility.

### Packages
Packages require a recent Docker version with [buildx](https://docs.docker.com/buildx/working-with-buildx/) installed and enabled.

To build all packages:
```bash
aswfdocker --verbose build -t PACKAGE --group-name common,base,vfx --group-version 2018,2019,2020
```

To build a single package, e.g. USD:
```bash
# First list the available CI packages to know which package belong to which "group":
aswfdocker packages
# Then run the build
aswfdocker --verbose build -t PACKAGE --group-name vfx --group-version 2019 --target usd
```

### Images
Images can be build with recent Docker versions but do not require [buildx](https://docs.docker.com/buildx/working-with-buildx/) 
but it is recommended to speed up large builds.

To build all images:
```bash
aswfdocker --verbose build -t IMAGE --group-name base,vfx1,vfx2,vfxall --group-version 2018,2019,2020
```

To build a single image:
```bash
# First list the available CI images to know which package belong to which "group":
aswfdocker images
# Then run the build
aswfdocker --verbose build -t IMAGE --group-name vfx1 --group-version 2019 --target openexr
```
