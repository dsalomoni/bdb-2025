# bdb-2025 <!-- omit in toc -->

This repository contains files used in the course "Biomedical Data Bases" (BDB) Module 2
at the University of Bologna, Academic Year 2024-2025, taught by prof. Davide Salomoni.

For details, see the course slides.

For more information on the course, see [here](https://www.unibo.it/en/study/phd-professional-masters-specialisation-schools-and-other-programmes/course-unit-catalogue/course-unit/2024/504329).

**Table of Contents**
- [1. Install Docker](#1-install-docker)
- [2. Create a docker network](#2-create-a-docker-network)
- [3. Start the Jupyter notebook](#3-start-the-jupyter-notebook)
  - [3.1. Making a local directory visible to Jupyter](#31-making-a-local-directory-visible-to-jupyter)
    - [3.1.1. On Windows 10/11](#311-on-windows-1011)
    - [3.1.2. On Linux/MacOS](#312-on-linuxmacos)
- [4. Start the redis server](#4-start-the-redis-server)
  - [4.1. Without persistence:](#41-without-persistence)
  - [4.2. With persistence:](#42-with-persistence)
- [5. Possible issues with Docker and how to fix them:](#5-possible-issues-with-docker-and-how-to-fix-them)
  - [5.1. `docker run` returns `docker: Error response from daemon: Conflict. The container name "/my_jupyter" is already in use ...`](#51-docker-runreturns-docker-error-response-from-daemon-conflict-the-container-name-my_jupyter-is-already-in-use-)
  - [5.2. On Windows, you get the message `"Error response ... : file exists."`](#52-on-windows-you-get-the-message-error-response---file-exists)
  - [5.3. On Windows, after you install Docker Desktop you cannot run VirtualBox VMs anymore](#53-on-windows-after-you-install-docker-desktop-you-cannot-run-virtualbox-vms-anymore)
  - [5.4. On Windows, you get the message `"docker_engine: Access is denied"`](#54-on-windows-you-get-the-message-docker_engine-access-is-denied)
  - [5.5. On Windows, you get the message `"Docker failed to initialize"` when running Docker Desktop](#55-on-windows-you-get-the-message-docker-failed-to-initialize-when-running-docker-desktop)
  - [5.6. On Windows, a message says that you need to enable virtualization](#56-on-windows-a-message-says-that-you-need-to-enable-virtualization)
  - [5.7. Other errors](#57-other-errors)

## 1. Install Docker

Refer to the course slides for instructions on how to do that.

## 2. Create a docker network

This is a Docker virtual network used by the containers that you will instantiate in this course.

`docker network create bdb-net`

## 3. Start the Jupyter notebook

Remember to **change the password** to access Jupyter (parameter `JUPYTER_TOKEN` in the command below). Once the Docker container is started,
connect to http://127.0.0.1:8888. Note anyway that with the command below access is only permitted from the PC where Docker is running.

```
docker run -d --rm --name my_jupyter --mount src=bdb_data,dst=/home/jovyan -p 127.0.0.1:8888:8888 --network bdb-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdb_password" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" jupyter/datascience-notebook
```

### 3.1. Making a local directory visible to Jupyter

If you want to make a directory on your system visible to Jupyter, modify the `docker run` command listed above as per the following instructions, which you should edit according to your needs: here a local directory called `bdb` will be mapped to the `work` directory on Jupyter.

#### 3.1.1. On Windows 10/11
- Open the Windows PowerShell as Administrator.
- Type the command `mkdir C:\bdb`
- If a `my_jupyter` container is running (check with `docker ps`), stop it with `docker stop my_jupyter`.
- Run the following command to start Jupyter:
```
docker run -d --rm --name my_jupyter --mount src=bdb_data,dst=/home/jovyan -p 127.0.0.1:8888:8888 --network bdb-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdb_password" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" --mount src=C:\bdb,dst=/home/jovyan/work,type=bind jupyter/datascience-notebook
```

#### 3.1.2. On Linux/MacOS
- Open a terminal window.
- Type the command `mkdir $HOME/bdb`
- If a `my_jupyter` container is running (check with `docker ps`), stop it with `docker stop my_jupyter`.
- Run the following command to start Jupyter:
```
docker run -d --rm --name my_jupyter --mount src=bdb_data,dst=/home/jovyan -p 127.0.0.1:8888:8888 --network bdb-net -e JUPYTER_ENABLE_LAB=yes -e JUPYTER_TOKEN="bdb_password" --user root -e CHOWN_HOME=yes -e CHOWN_HOME_OPTS="-R" --mount src=$HOME/bdb/,dst=/home/jovyan/work,type=bind jupyter/datascience-notebook
```

## 4. Start the redis server

### 4.1. Without persistence:
```
docker run -d --rm --name my_redis --mount src=bdb_data,dst=/data --network bdb-net --user 1000 redis redis-server --maxmemory 32mb --maxmemory-policy allkeys-lru
```

### 4.2. With persistence:
```
docker run -d --rm --name my_redis --mount src=bdb_data,dst=/data --network bdb-net --user 1000 redis redis-server --maxmemory 32mb --save 180 1 --dbfilename my_database.rdb
```

## 5. Possible issues with Docker and how to fix them:

### 5.1. `docker run` returns `docker: Error response from daemon: Conflict. The container name "/my_jupyter" is already in use ...`

- Type `docker stop my_jupyter`
- Type `docker rm my_jupyter`
- Type the `docker run` command again.

### 5.2. On Windows, you get the message `"Error response ... : file exists."`
 
 If **on Windows** you get an error like the following when typing a `docker run` command:
> Error response from daemon: error while creating mount source path ... file exists.

you should restart the Docker daemon. Open the "Docker Desktop" application from the Windows main menu. If any updates to Docker are mentioned, apply them. After these updates (if any), make sure you select the option to restart the Docker daemon. Then issue the `docker run` command again.

### 5.3. On Windows, after you install Docker Desktop you cannot run VirtualBox VMs anymore

This is a problem occurring only on Windows, where anyway it does not always happen. If you do not have VirtualBox installed, or if VirtualBox and Docker both work on your system, skip this part. 

If you have this issue, there are a couple of options. 

1. if you use VirtualBox just to get a Ubuntu system, have a look at the <a href="https://github.com/dsalomoni/my-ubuntu">my ubuntu</a> repository, which explains how to use Docker to run and customize a full Ubuntu system, complete with a GUI. This is typically _much faster_ than a VirtualBox VM.

2. if you do need to have VirtualBox VMs _and_ you experience that after having installed Docker Desktop you cannot run VirtualBox VMs anymore, first of all **update your VirtualBox software** to the latest available version. This is because in recent versions VirtualBox should have fixed their incompatibility with Docker.

   If you still have the problem even after having upgraded VirtualBox to the latest version, there is unfortunately only a workaround that seems to work, and that is to enable **either** Docker Desktop **or** VirtualBox. You can have both installed on your system, but only one at a time can be enabled.

   Enabling one or the other is done via this procedure:

   1. open the Windows Control Panel (in Italian: Impostazioni)
   2. search for the string "Windows features" and select the item "Turn Windows features on or off". Those with an Italian version of Windows may search for the string "Attiva o disattiva funzionalità di Windows".

   Now, in the windows that opens up:

   - **if you want to use VirtualBox**, _deselect_ the items called "Virtual machine platform" and "Windows Hypervisor Platform" and then click "OK". 
   - **if you want to use Docker**, _select_ the items called "Virtual machine platform" and "Windows Hypervisor Platform" and then click "OK". 

   Those with an Italian version of Windows will see the two items mentioned above translated as "Piattaforma macchina virtuale" and "Piattaforma Windows Hypervisor".

   In either case, you will have to reboot your system. When Windows boots up again, you will be able to run either VirtualBox or Docker, depending on whether you deselected or selected the items above. If you find a different way to handle this issue, please let prof. Salomoni know.

### 5.4. On Windows, you get the message `"docker_engine: Access is denied"`

This error may be due to several reasons:

- make sure that Docker Desktop is properly installed and that when you open it it says "Docker is running". If Docker cannot start, make sure you have applied all suggested updates, including (if you are prompted about that) the "Windows Subsystem for Linux 2", or WSL 2. If all updates have been applied, deinstall and reinstall the Docker Desktop application, rebooting when prompted to do so.
- make sure you run the Windows terminal as "administrator". 

### 5.5. On Windows, you get the message `"Docker failed to initialize"` when running Docker Desktop

This error may be due to several reasons. A possible workaround is described at https://github.com/docker/for-win/issues/3088, where it is suggested to delete the directory `C:\Users\xxxxxxx\AppData\Roaming\Docker` (replace `xxxxxxx` with your username).

### 5.6. On Windows, a message says that you need to enable virtualization

If you see an error like this when starting Docker Desktop for Windows:

    “Hardware assisted virtualization and data execution protection must be enabled in the BIOS. See Logs and troubleshooting | Docker Documentation”

This might mean first of all that hardware virtualization is not enabled for your laptop. Recent laptops normally have it enabled by default, as this is a feature that _must_ be turned on for Docker (or for VirtuaBox, for that matter) to work.
You can check and possibly enable hardware virtualization when the laptop starts, accessing its "BIOS configuration". How to access this configuration varies from laptop to laptop, so you should look up how to do it in your laptop manual.

If you know that hardware virtualization is enabled but you are still getting this error, it is possible that Windows "believes" that it is still not enabled. To fix this, open PowerShell __with administrator privileges__ and issue the following command:

    bcdedit /set hypervisorlaunchtype auto

and then reboot your laptop. Make also sure you have followed the advices in the section describing what to do if VirtualBox works and Docker doesn't; see in particular the section explaining what to do with the "Windows features".

### 5.7. Other errors

If you encounter other errors, contact prof. Salomoni through the usual University channels.