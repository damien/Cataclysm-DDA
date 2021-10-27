# syntax = docker/dockerfile:1.3
# Note: The above line is a flag that enables Docker's BuildKit build engine.
# This Docker image depends on BuildKit, so ensure it is enabled when running 
# docker build` or this image may fail to build.
# See: https://hub.docker.com/r/docker/dockerfile

# [Choice] Ubuntu or Debian version. Full list of valid tags listed on Dockerhub's buildpack-deps page.
# See: https://hub.docker.com/_/buildpack-deps/
ARG VARIANT="hirsute"

# Optional: Configure an Apt cache or proxy.
#
# This greatly speeds up rebuilds of this container and is recommended for
# anyone working on changes or improvements to this Dockerfile.

# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.202.5/containers/ubuntu/.devcontainer/base.Dockerfile
FROM mcr.microsoft.com/vscode/devcontainers/base:0-${VARIANT}


# pyenv system dependencies
# See: https://github.com/pyenv/pyenv/wiki#suggested-build-environment
ARG PYENV_DEPENDENCIES="make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev"
ENV PYENV_DEPENDENCIES=${PYENV_DEPENDENCIES}

# Version of Python to default to within this image.
# Change this when we want to do most development against a different version of Python.
#
# Developers using this image may also use `pyenv` to install
# and use whatever version of Python they wish to use.
ARG DEFAULT_PYTHON_VERSION="3.10.0"
ENV DEFAULT_PYTHON_VERSION=${DEFAULT_PYTHON_VERSION}


# Cataclysm: Dark Days Ahead build dependencies
# See: https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/COMPILING/COMPILING.md
# See: https://github.com/damien/Cataclysm-DDA/blob/master/.github/workflows/release.yml
ARG CDDA_DEPENDENCIES_BUILD="build-essential make astyle ccache gettext parallel"
ENV CDDA_DEPENDENCIES_BUILD=${CDDA_DEPENDENCIES_BUILD}

ARG CDDA_DEPENDENCIES_NCURSES="libncurses5-dev libncursesw5-dev"
ENV CDDA_DEPENDENCIES_NCURSES=${CDDA_DEPENDENCIES_NCURSES}

ARG CDDA_DEPENDENCIES_SDL="libsdl2-dev libsdl2-ttf-dev libsdl2-image-dev libsdl2-mixer-dev libfreetype6-dev"
ENV CDDA_DEPENDENCIES_SDL=${CDDA_DEPENDENCIES_SDL}


# The base Ubuntu Docker image configures Apt to remove packages after installation to keep image sizes small.
# We're using Docker BuildKit's build mounts feature to cache Apt package downloads and speed up subsequent
# builds of this development environment, so we'd rather keep cache downloaded Apt packages between builds.
#
# See: https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/syntax.md#run---mounttypecache
RUN rm -f /etc/apt/apt.conf.d/docker-clean; echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache

# Install any system dependencies required by this project.
RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt \
    apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends ${PYENV_DEPENDENCIES}

RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt \
    apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends ${CDDA_DEPENDENCIES_BUILD}

RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt \
    apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends ${CDDA_DEPENDENCIES_NCURSES}

RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt \
    apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends ${CDDA_DEPENDENCIES_SDL}


# Use Pyenv and Poetry to manage Python versions and dependencies
#
# Note: Previous commands are run as `root` while Pyenv and Poetry
# may only be installed by a non-root user (`vscode`)
USER vscode
ENV HOME /home/vscode

## Pyenv
RUN curl -sSL https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash

# Directory pyenv installs to
ENV PYENV_ROOT ${HOME}/.pyenv

# Ensure Pyenv installed binaries are discoverable on user path
ENV PATH ${PYENV_ROOT}/shims:${PYENV_ROOT}/bin:${PATH}

# Specify a directory for pyenv's python-install script to store downloaded packages in.
# This is required to avoid re-downloading Python versions between rebuilds of this Docker image.
# See: https://github.com/pyenv/pyenv/tree/master/plugins/python-build#special-environment-variables
# See: https://github.com/pyenv/pyenv/tree/master/plugins/python-build#package-download-caching
ENV PYTHON_BUILD_CACHE_PATH ${PYENV_ROOT}/cache

# Set up directories we want Pyenv to use
RUN mkdir -p ${PYENV_ROOT}

# Install our desired default version of Python
# Note: 1000 is the UID of user "vscode", though this may change
# if other users are created before the vscode user is.
RUN --mount=type=cache,target=${PYTHON_BUILD_CACHE_PATH},uid=1000 \
    pyenv install $DEFAULT_PYTHON_VERSION
RUN pyenv global $DEFAULT_PYTHON_VERSION

## Poetry
ENV POETRY_HOME ${HOME}/.poetry
ENV PATH ${POETRY_HOME}/bin:${PATH}

RUN curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/install-poetry.py | python -

# This is a single project Docker image, so we have no need for Poetry to use
# Python virtualenvs to isolate dependencies
RUN poetry config virtualenvs.create false

# Running the container directly will drop the user into a fully configured shell environment
CMD ["bash"]
