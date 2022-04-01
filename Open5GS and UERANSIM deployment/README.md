# Open5GS and UERANSIM deployment

- [Requirements](#requirements)
- [Configuration steps](#configuration-steps)

- [Project Main Page](https://github.com/J1mmy99/6184_5G_ansible_deployment)


### Requirements
The installation can be done directly over the host operating system (OS) or inside a virtual machine (VM).   

**System requirements**

Open5GS(5G Core network) deployment
- CPU type: x86-64 (specific model and number of cores only affect performance)
- RAM: 4 GB
- Disk space: 20 GB
- Ubuntu 18.04 LTS

UERANSIM(5G UE & 5G RAN) deployment
- CPU type: x86-64 (specific model and number of cores only affect performance)
- RAM: 4 GB
- Disk space: 20 GB
- Ubuntu 18.04 LTS

### Configuration steps
The following steps are required to complete setup.

**On server1:**

**1. Install git:**      
    
``` sudo apt git ``` 

**2. Install ansible:**      
    
``` sudo apt ansible ```

**3. Clone this repository:**      
    
``` git clone https://github.com/J1mmy99/6184_5G_ansible_deployment.git ```

**4. Go to the folder:**      
    
``` cd Open5GS and UERANSIM deployment ```
``` cd 6184_5G_ansible_deployment/ ```

**5. Run ansible playbook:**

``` ansible-playbook ansible_core_network.yaml ```


