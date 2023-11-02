# [Legacy] DQMGUI Deployment

This repository contains all the requirements for deploying the [Legacy] [DQMGUI](https://github.com/cms-DQM/dqmgui_prod) on a Red Hat Enterprise Linux 8 machine, with Python3.6 for the needs of moving DQM production to new machines.

It aims to replace the existing one (`Deploy`), found in [`dmwm/deployment`](https://github.com/dmwm/deployment/tags), which only targets OS up to SLC7. The main difference is that the `Deploy` script relies on pre-built libraries and executables, found on `cmsrep.cern.ch`, while this script downloads and builds all requirements from source (hence takes longer to deploy).

## Requirements

- RHEL8 (Tested with version 8.8)
- `sudo` permissions:
  - To install the system-wide packages (listed below).
  - To create a soft link in `/usr/lib64`.
- Python 3.6
- Git
- Access to PyPI for downloading python packages.
- Access to GitHub for cloning several external dependencies.
- Several system packages, installed via `yum`:
  <details>
  <summary>Package list</summary>  

  - bzip2
  - libglvnd-opengl
  - libX11-devel
  - libXext-devel
  - libXft-devel
  - libXpm-devel
  - mesa-libGLU
  - mesa-libGLU-devel
  - perl-Env
  - perl-Switch
  - perl-Thread-Queue
  - glibc-headers
  - libidn
  - libXcursor
  - libXi
  - libXinerama
  - libXrandr
  - perl
  - perl-Digest-MD5
  - tcsh
  - zsh
  - root
  - python3-root
  - epel-release
  - python3-pip
  - libcurl-devel
  - python36-devel
  - boost-python3-devel
  - protobuf-devel
  - jemalloc-devel
  - pcre-devel
  - boost-devel
  - lzo-devel
  - cmake
  - xz-devel
  - python3-sphinx
  - openssl-devel
  - libjpeg-turbo-devel
  - libpng-devel
  </details>

## Deploying [Legacy] DQMGUI

This procedure has been tested on a RHEL8 Openstack VM.


1. Install the system packages:

  ```bash
  sudo yum install -y tmux git bzip2 libglvnd-opengl libX11-devel libXext-devel libXft-devel libXpm-devel mesa-libGLU mesa-libGLU-devel perl-Env perl-Switch perl-Thread-Queue glibc-headers libidn libXcursor libXi libXinerama libXrandr perl perl-Digest-MD5 tcsh zsh root python3-root epel-release python3-pip libcurl-devel python36-devel boost-python3-devel protobuf-devel jemalloc-devel pcre-devel boost-devel lzo-devel cmake xz-devel python3-sphinx openssl-devel libpng-devel lijpeg-turbo-devel 
  ```

1. Create a link to `libDQMGUI.so` which we will be compiling shortly:
  
  ```bash
  ln -s /data/srv/<DMWM tag>/sw/el8_amd64_gcc11/cms/dqmgui/<DQMGUI tag>/128/lib/libDQMGUI.so /usr/lib64/libDQMGUI.so
  ```

  Replace:

  - `<DMQM tag>` with the [`dmwm/deployment`](https://github.com/dmwm/deployment/tags) tag you want to use (contains the layouts).
  - `<DQMGUI tag>` with the [DQMGUI](https://github.com/cms-DQM/dqmgui_prod/tags) tag you want to use (contains the underlying GUI code).

  e.g.:

  ```bash
  ln -s /data/srv/HG2903c/sw/el8_amd64_gcc11/cms/dqmgui/10.0.0/128/lib/libDQMGUI.so /usr/lib64/libDQMGUI.so
  ```

1. Add a non-privileged user, create and give access to necessary directories and switch to it:
  
  ```bash
  adduser dqm
  sudo mkdir -p /data/srv
  sudo chown -R dqm /data/srv
  sudo mkdir -p /dqmdata
  sudo chown -R dqm /dqmdata
  sudo su dqm
  ```

1. Start the deployment (`dev` flavor):
  
  ```bash
  cd ~
  # Clone this script and all necessary files, like patches
  git clone https://github.com/cms-DQM/dqm_gui_prod_deployment
  
  # Start the deployment script, it will take some time to finish
  bash /home/dqm/dqm_gui_prod_deployment /deploy_dqm_online_el8.sh

  # Start all the services
  /data/srv/current/config/dqmgui/manage -f dev start "I did read documentation"
  ```


## [Debug] Selectively run parts of the installation script

The script is split into steps (see: `installation_steps`), which can all be toggled off or on by arguments when running the script.

The flags are named by concatenating `do_` with the name of the step, so, for example, `do_check_dependencies` or `do_install_rotoglup`.

The are all set to `1` by default, and you can override them when running the script as follows:

```bash
bash deploy_dqm_online_el8.sh do_preliminary_checks=0 do_check_dependencies=0
```

A useful combination that can be used when you've already downloaded all external dependencies once, and you want just to re-compile the DQMGUI part for testing:

```bash
bash deploy_dqm_online_el8.sh do_preliminary_checks=0 do_check_dependencies=0 do_preliminary_checks=0 do_check_dependencies=0 do_create_directories=1 do_install_boost_gil=0 do_install_gil_numeric=0 do_install_rotoglup=0 do_install_classlib=0 do_install_dmwm=0 do_install_dqmgui=0 do_compile_dqmgui=1 do_install_yui=0 do_install_extjs=0 do_install_d3=0 do_install_jsroot=0
```
