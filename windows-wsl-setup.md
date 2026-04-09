# Configure Windows WSL2

Windows has an integrated linux subsystem called WSL2 (Windows Subsystem for Linux). This allows you to run a
full linux environment on your Windows machine. This is useful for a number of reasons, but primarily because many
of the tools I use in development are linux based. This document will walk you through setting up WSL2 with
docker, docker-compose, IntelliJ, and various cloud CLIs.

!!! warning
    Create an Ubuntu 24.04 subsystem, otherwise you may well run into issues with some of the installs below.


## Install Windows Subsystem for Linux

1. Control Panel
1. Programs and Features
1. Windows Features
1. Enable Windows Subsystem for Linux
1. Enable Powershell if it isn’t already enabled
1. Go to the Windows Store
1. Install Ubuntu 24.04
1. Reboot
1. Make sure it works
1. Launch Powershell and run the following

## Check/Install Ubuntu 24.04

Verify if you have Ubuntu 24.04 installed and functional
```
> wsl --list

Windows Subsystem for Linux Distributions:
Ubuntu-24.04 (Default)
```

If you don’t see Ubuntu-24.04 then you can install it via the commandline.
You can see all the distros available to install by running:

```
> wsl --list --online

The following is a list of valid distributions that can be installed.
Install using 'wsl.exe --install <Distro>'.

NAME                            FRIENDLY NAME
AlmaLinux-8                     AlmaLinux OS 8
AlmaLinux-9                     AlmaLinux OS 9
AlmaLinux-Kitten-10             AlmaLinux OS Kitten 10
AlmaLinux-10                    AlmaLinux OS 10
Debian                          Debian GNU/Linux
FedoraLinux-42                  Fedora Linux 42
SUSE-Linux-Enterprise-15-SP6    SUSE Linux Enterprise 15 SP6
SUSE-Linux-Enterprise-15-SP7    SUSE Linux Enterprise 15 SP7
Ubuntu                          Ubuntu
Ubuntu-24.04                    Ubuntu 24.04 LTS
archlinux                       Arch Linux
kali-linux                      Kali Linux Rolling
openSUSE-Tumbleweed             openSUSE Tumbleweed
openSUSE-Leap-15.6              openSUSE Leap 15.6
Ubuntu-20.04                    Ubuntu 20.04 LTS
Ubuntu-22.04                    Ubuntu 22.04 LTS
OracleLinux_7_9                 Oracle Linux 7.9
OracleLinux_8_10                Oracle Linux 8.10
OracleLinux_9_5                 Oracle Linux 9.5
```

To install Ubuntu-24.04 run the following command in Powershell

```
> wsl --install Ubuntu-24.04
```

Now you should see it when you run `wsl --list` again.

```
> wsl -l -v
  NAME            STATE           VERSION
* Ubuntu-24.04    Running         2
```

## Install docker

1. Launch Ubuntu

1. Install dependencies

    ```
     sudo apt-get update
     sudo apt-get -y install apt-transport-https ca-certificates curl gnupg lsb-release
    ```

1. Add apt repository for docker

    ```
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

1. Install docker

    ```
    sudo apt-get update
    sudo apt-get -y install docker-ce docker-ce-cli containerd.io
    ```

1. Update user/group permissions
Most of the time the docker group should already exist but one time it didn’t get created for some reason.

    ```
    sudo groupadd docker
    sudo usermod -aG docker $USER
    ```

1. Start docker and validate

    ```
    sudo service docker start
    sudo service docker status
    docker run hello-world
    ```

1. Install docker-compose  - You should check and see if there’s a newer release of docker-compose than the one used in this command.

    ```
    sudo curl -L https://github.com/docker/compose/releases/download/v2.34.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    ```

1. Validate docker-compose

    ```
    docker-compose --version
    ```

1.  Validate things work in PowerShell
In PowerShell run:

    ```
    wsl docker ps
    wsl docker-compose --version
    ```

1. Create PowerShell aliases (optional)
   1.  Allow local (and remote signed) scripts to run
   In Powershell run

       ```
       Set-ExecutionPolicy -Scope CurrentUser
       ```

   1. It’ll prompt you for a value. Enter  

       ```
       RemoteSigned
       ```

   1. Find your PowerShell profile location  

       ```
       echo $PROFILE
       ```

   1. Create a file at that location with the following contents  

       ```
       Function Start-WslDocker {
           wsl docker $args
       }
       Function Start-WslDockerCompose {
           wsl docker-compose $args
       }
       Set-Alias -Name docker -Value Start-WslDocker
       Set-Alias -Name docker-compose -Value Start-WslDockerCompose
       ```
      
   1. Close PowerShell and reopen

   1. Validate aliases work

    ```
    docker ps
    docker-compose --version
    ```

## Install IntelliJ for linux

1. Download JDK 21 from [https://adoptium.net/temurin/releases/](https://adoptium.net/temurin/releases/)
1. Start Ubuntu and mv the downloaded JDK from /mnt/c/Users/<user>/Downloads to ~
1. Unzip the JDK and install it\
   Note: Obviously download the latest current version.

    ```
    tar zfx OpenJDK21U-jdk_x64_linux_hotspot_21.0.6_7.tar.gz
    sudo mv jdk-21.0.6+7 /opt
    sudo ln -s /opt/jdk-21.0.6+7 /opt/jdk-21
    ```

1. Add the following to your .bashrc

    ```
    export JAVA_HOME=/opt/jdk-21
    ```

1. Download the linux version of IntelliJ from
   [https://www.jetbrains.com/idea/download/?section=linux](https://www.jetbrains.com/idea/download/?section=linux)

1. In Ubuntu (obviously change the commands to the version you download)

    ```
    mv /mnt/c/Users/<user>/Downloads/ideaIU-2024.3.4.1.tar.gz ~
    ```

1. Install IntelliJ

    ```
    tar zfx ideaIU-2024.3.4.1.tar.gz
    sudo mv idea-IU-243.24978.46 /opt
    sudo ln -s /opt/idea-IU-243.24978.46 /opt/intellij
    ```

1. Add the following to your .bashrc (after the JAVA_HOME line)

    ```
    export PATH=$JAVA_HOME/bin:/opt/intellij/bin:$PATH
    ```

1. Create a menu item for IntelliJ (this will add it to your Start Menu in Windows) Your mileage may vary with this. It
   worked for me for a while but stopped. The menu item is there but the app never launches.

    Create a file /usr/share/applications/intellij.desktop with the following contents

    ```
    [Desktop Entry]
    Name=intellij
    TryExec=idea.sh
    Exec=/opt/intellij/bin/idea
    Type=Application
    Icon=/opt/intellij/bin/idea.png
    ```

1. Install Firefox so you can register IntelliJ.

    ```
    sudo apt-get install xdg-utils firefox
    ```

## Optional Steps

### Install Portainer (Web UI kind of like Docker Desktop)

Note: I personally don't think this is worthwhile unless you are very graphically oriented and want something that behaves kind of like the Docker Desktop UI.

1. Create Portainer data volume

    ```
    docker volume create portainer_data
    ```

1. Run Portainer

    NOTE: This step occasionally fails. If that happens delete the docker volume and recreate it.

    ```
    docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:lts
    ```

1. Configure Portainer
   1. Open up a web browser and go to https://localhost:9443
   1. Trust the self-signed cert
   1. Create an admin user
   1. Validate by making sure you can see the images pulled down earlier for testing

### Install AWS CLI on linux

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt-get install zip unzip bzip2
unzip awscliv2.zip
sudo aws/install
aws --version
```

### Install boto3 (AWS python module)

```
sudo apt-get install python3-full
sudo apt-get install python3-pip
sudo apt-get install python3-boto3
```

### Install Azure CLI on linux

```
curl -sLS https://packages.microsoft.com/keys/microsoft.asc |
  gpg --dearmor | sudo tee /etc/apt/keyrings/microsoft.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/microsoft.gpg
AZ_DIST=$(lsb_release -cs)
echo "Types: deb
URIs: https://packages.microsoft.com/repos/azure-cli/
Suites: ${AZ_DIST}
Components: main
Architectures: $(dpkg --print-architecture)
Signed-by: /etc/apt/keyrings/microsoft.gpg" | sudo tee /etc/apt/sources.list.d/azure-cli.sources
sudo apt-get update
sudo apt-get install azure-cli
```

#### Install Azure python modules in a virtual env

```
python3 -m venv ~/venv
~/venv/bin/pip install asyncio azure.identity msgraph-core msgraph-sdk ldap3
```

### Install Powershell on linux

Most people won’t need this but sadly I do for some shell script + Powershell automation for Entra ID.

```
 source /etc/os-release
 wget -q https://packages.microsoft.com/config/ubuntu/$VERSION_ID/packages-microsoft-prod.deb
 sudo dpkg -i packages-microsoft-prod.deb
 sudo apt-get update
 sudo apt-get install powershell
```

#### Install Powershell modules

Note: All of this is the same on Windows except you don’t need to `sudo pwsh`.

```
sudo pwsh
# Install Azure module (slow and no feedback for awhile)
Install-Module -Name Az -Repository PSGallery -Force

# Install Exchange module
Install-Module -Name ExchangeOnlineManagement

# Make sure it is up to date and pulls in child modules
Update-Module -Name Az -Force

# Login (powershell may need to be launched as Administrator)
Connect-AzAccount

# Get Subscription details
Get-AzSubscription

# If there is more than one pick the right one
Select-AzSubscription <name>
Get-AzSubscription -Current

# Log out
Disconnect-AzAccount
```

### Install Google Cloud CLI

1. Install
    ```
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg
    echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
    sudo apt-get update && sudo apt-get install google-cloud-cli
    gcloud init
    ```

1. Install Optional modules

   There are a bunch of other gcloud related packages. Install whichever ones you want/need via \`apt-get\`.

   - `google-cloud-cli-anthos-auth`
   - `google-cloud-cli-app-engine-go`
   - `google-cloud-cli-app-engine-grpc`
   - `google-cloud-cli-app-engine-java`
   - `google-cloud-cli-app-engine-python`
   - `google-cloud-cli-app-engine-python-extras`
   - `google-cloud-cli-bigtable-emulator`
   - `google-cloud-cli-cbt`
   - `google-cloud-cli-cloud-build-local`
   - `google-cloud-cli-cloud-run-proxy`
   - `google-cloud-cli-config-connector`
   - `google-cloud-cli-datastore-emulator`
   - `google-cloud-cli-firestore-emulator`
   - `google-cloud-cli-gke-gcloud-auth-plugin`
   - `google-cloud-cli-kpt`
   - `google-cloud-cli-kubectl-oidc`
   - `google-cloud-cli-local-extract`
   - `google-cloud-cli-minikube`
   - `google-cloud-cli-nomos`
   - `google-cloud-cli-pubsub-emulator`
   - `google-cloud-cli-skaffold`
   - `google-cloud-cli-spanner-emulator`
   - `google-cloud-cli-terraform-validator`
   - `google-cloud-cli-tests`
   - `kubectl`

1. Install on Windows

    ```
    (New-Object Net.WebClient).DownloadFile("https://dl.google.com/dl/cloudsdk/channels/rapid/GoogleCloudSDKInstaller.exe", "$env:Temp\GoogleCloudSDKInstaller.exe")
    
    & $env:Temp\GoogleCloudSDKInstaller.exe
    
        cd $env:Temp
        .\GoogleCloudSDKInstaller.exe
        gcloud init
    ```
### Install node.js

Ubuntu 22.04's own nodejs is version 18 which is EOL.  Install a newer version from
nodesource.

    ```
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo bash -
    sudo apt-get install nodejs
    ```

### Install ATX (Amazon Transform Custom, free code upgrade AI tool)

This is used for automated upgrading of python, java, and node.js codebases.  Experimental...
Note: This requires node.js 20 or newer to be installed.

    ```
    curl -fsSL https://desktop-release.transform.us-east-1.api.aws/install.sh | bash
    ```
