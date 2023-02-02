# docker-nvidia-ansible
An Ansible playbook to install docker and nvidia drivers on Ubuntu Linux x86_64 machines.

## Included software

- [Docker Engine](https://docs.docker.com/engine/install/ubuntu/) with bash completion.
- [NVIDIA driver and Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).

## System requirements
1. [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html?extIdCarryOver=true&sc_cid=701f2000001OH7YAAW)
on your controller machine (e.g. laptop). This should be different from the machines you
are installing the packages to. The easiest way to install ansible (assuming python is installed)
is to use `pip`:
    ```bash
    pip install --user ansible
    ```
    Verify that ansible is installed by running:
    ```bash
    ansible --version
    ```
    You also need a couple of additional Ansible collections, install using the command:
    ```bash
    ansible-galaxy install nvidia.nvidia_driver nvidia.nvidia_docker
    ```
1. One or more SSH connections to the target machine(s) with public key authentication configured. You can do that by editing your `.ssh/config`. For example:
    ```bash
    # ~/.ssh/config
    Host awsgpu1
        HostName 100.100.100.100
        User ubuntu
        IdentityFile ~/.ssh/id_rsa
        IdentitiesOnly yes
    ```

1. Edit `inventory.ini` to include the target machines:
    ```ini
    # inventory.ini
    [all]
    awsgpu1 ansible_ssh_user=ubuntu
    awsgpu2 ansible_ssh_user=ubuntu
    ...
    ```

## Installation
To launch the installation process that will configure all target
machines at once, run the following:
```bash
git clone https://github.com/hammady/docker-nvidia-ansible.git
cd docker-nvidia-ansible
ansible-playbook docker-nvidia.yaml --inventory ./inventory.ini
```

After a couple of minutes, you should get a report similar to the below if everything goes well:
```
...
PLAY RECAP ****************************************************************************************************************************************************************************************************************************************
awsgpu1                    : ok=34   changed=14   unreachable=0    failed=0    skipped=6    rescued=0    ignored=0   
```

## Testing

To test that the installation was successful, restart then ssh to the machine(s) and run the following:
```bash
docker run --rm --gpus all nvidia/cuda:11.6.2-base-ubuntu20.04 nvidia-smi
```

You should get a report similar to the below:
```
Thu Feb  2 01:25:42 2023
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 515.86.01    Driver Version: 515.86.01    CUDA Version: 11.7     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
| N/A   29C    P8    14W /  70W |      2MiB / 15360MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
