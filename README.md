# Emma

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.996308.svg)](https://doi.org/10.5281/zenodo.996308)

Emma is a project to create a platform for development of application for Spark and DockerSwarm clusters. The platform runs on an infra-structure composed by virtual machines and Ansible playbooks are used to create a storage layer, processing layer and [JupyterHub](https://jupyter-notebook.readthedocs.io/en/latest/index.html) services. The storage layer offers two flavors of storage, file-base by [GlusterFS](https://www.gluster.org/) and [Hadoop Distributed File System (HDFS)](http://hadoop.apache.org/), and object-based using [Minio](https://www.minio.io). The processing layer has a [Apache Spark cluster](http://spark.apache.org/) and a [Docker Swarm](https://docs.docker.com/engine/swarm/) sharing the storage instances.

## Deployment
At the moment the deployment of clusters with **emma** was tested with two OSs environment, Linux and Windows.

### Linux as host OS
For Linux it was tested using Ubuntu 14.04 and Ubuntu 16.04, however, it is recommended to use the latter. For Linux no special environment setup is required.
Before deploying a cluster user needs to clone **emma** repository:
```
git clone https://github.com/nlesc-sherlock/emma
```

### Windows as host OS
For Windows, **emma** was only tested on Windows 10 using the embedded Ubuntu 16.04 environment. It setup is straight forward and it simple follows the steps listed in the [installation guide](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide).

Once installed it recommended to verify if the version 16.04 is installed. For that the user simply needs to type **bash** in the search windows at the left bottom of Windows 10 Desktop-environment and press enter.
Once the bash console is open the user should then type:
```
lsb_release -a
```

If not 16.04, then do the following:
```
#Powershell as administrator, and enter the command
lxrun /uninstall /full .

lxrun /install /y

#Verify again the version:
lsb_release -a
```

After the installation the Ubuntu environment is accessible through the bash command of Windows.
To add Windows executables to your Ubuntu *$PATH*, add the following to your **~/.bashrc** in the bash console:
```
vim ~/bashrc
export PATH=$PATH:/mnt/c/Windows/System32/
```

Note the *C* drive will be mounted with the files owned by *root* and file permissions set to *777*.
This means ssh keys will to open for Ansible. Hence, before you run ansible you need to call getHosts.sh.

For windows users need to change the line-ending setting for **git**, they should configure **git** to *Checkout as-is, commit as-is*:
```
git config --global core.autocrlf false
```

Before deploying a cluster user needs to clone **emma** repository:
```
git clone https://github.com/nlesc-sherlock/emma
```

The Windows Subsystem Linux will mount the C: drive as `/mnt/c`. 
By default the file permissions will be read/write/executable for all, this is bad because to use ssh-keys stricter permissions are required.
To solve this create the `/etc/wsl.conf` file with the following content and restart the shell.
```ini
[automount]
enable=true
root=/mnt
options="metadata"
```

### Setup environment

Independently of which OS the user is using the following steps need to be done after the repository is cloned.
```
cd emma

#create and edit env_linux.sh
cp env_linux.sh.template env_linux.sh
vim env_linux.sh

#Linux environments (also in the embedded Ubuntu environment in Windows).
#On each bash
. env_linux.sh

# Key used by root, do not set passphrase when asked
ssh-keygen -f ${CLUSTER_NAME}.key
# Key used by ${CLUSTER_NAME} user
ssh-keygen -f files/${CLUSTER_NAME}.key
```

Every time the user opens a bash console on Windows or Linux the environment is set through the following commands:
```
#Windows
cd <path_to_emma>/emma
. env_linux.sh
./env_windows.sh

#Linux
cd <path_to_emma>/emma
. env_linux.sh
```

### Infra-structure

With the environment set, the next step is to setup the infra-structure. The infra-structure, physical place where the platform runs, is composed by a set of virtual machines with the following characteristics:
1. Ubuntu 16.04 OS
2. Public network interface
3. OS disk, 200Mb for software + enough room in /tmp
4. Passwordless login as root with `${CLUSTER_NAME}.key` private key.
5. XFS Partition mounted at /data/local (used for swapfile, GlusterFS brick, Docker root)
6. Python2 to run Ansible tasks


The infrastructure is a collection of machines which must be reachable by ssh. The machines must be prepared/constructed by either [preparing cloud virtual machine](cloud.md) or [constructing using Vagrant boxes](vagrant.md).
Once the machines are prepared the servers are provisioned using [Ansible](https://www.ansible.com/), an automation tool for IT infra-structure. The roles defined for Ansible will create a platform with the following features:

* [GlusterFS](glusterfs.md)
* [Minio](minio.md)
* [Hadoop](hadoop.md)
* [Spark](spark.md) Standalone cluster
* [Docker Swarm](dockerswarm.md)
* [JupyterHub](jupyterhub.md)

Preceding the platform's installation, the user should click on each feature to understand the setup requirements for each of them. Once all the requirements have been fulfilled, the user should follow the platform's installation steps listed in **[ansible.md](ansible.md)**.
