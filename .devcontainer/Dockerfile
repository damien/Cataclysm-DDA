# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.191.1/containers/cpp/.devcontainer/base.Dockerfile

# [Choice] Debian / Ubuntu version: debian-11, debian-10, debian-9, ubuntu-20.04, ubuntu-18.04
ARG VARIANT="buster"
FROM mcr.microsoft.com/vscode/devcontainers/cpp:0-${VARIANT}

# * See https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/COMPILING/COMPILING.md
#   for details on compilation dependencies
# * See https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/COMPILING/COMPILER_SUPPORT.md
#   for details on supported compilers and compiler tooling
# * See https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/DEVELOPER_TOOLING.md
#   for details on tools commonly used and configured for this project
# * Note: libintl is included as part of the `libc6-dev` on Debian systems
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
    astyle \
    build-essential \
    gettext \
    libfreetype6-dev \
    libncurses5-dev \
    libncursesw5-dev \
    libsdl2-dev \
    libsdl2-image-dev \
    libsdl2-mixer-dev \
    libsdl2-ttf-dev
