# **k3d**
**Creating Kubernetes Cluster with K3D on Windows**

 
![image](https://user-images.githubusercontent.com/38450758/209392230-1804188c-b498-4be2-be60-e1bb44e0fe6b.png)

**We can use any lightweight tool to create a Kubernetes cluster on our local machine like:** 

•	Docker Desktop free docker product

•	K3D Open source project maintained by Rancher

•	Minikube Open source project – Kubernetes SIGs (Special Interest Group)

•	Kind Open source project – Kubernetes SIGs (Special Interest Group)

•	MicroK8s Open source project maintained by Canonical


**In this lab we are going to use K3D for creating Kubernetes cluster**

**K3D** is a lightweight wrapper to run k3s with docker. Unlike k3s, docker containers can be used to create Kubernetes nodes. 
It makes it useful to create local multi node clusters on the same machine. This is particularly useful for devs and testers alike, 
as they would not need to deal with complication of setting up multi node Kubernetes setup.

**Install kubectl on our local machine using the instructions below**

•	**Windows**

    export VER=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)

    curl -LO https://storage.googleapis.com/kubernetes-release/release/$VER/bin/windows/amd64/kubectl.exe

    chmod +x kubectl.exe

    mkdir -p $HOME/bin/

    mv kubectl $HOME/bin/

•	**Linux**

    export VER=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)

    curl -LO https://storage.googleapis.com/kubernetes-release/release/$VER/bin/linux/amd64/kubectl

    chmod +x kubectl

    mv kubectl /usr/local/bin/

•	**MacOS**

    export VER=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)

    curl -LO https://storage.googleapis.com/kubernetes-release/release/$VER/bin/darwin/amd64/kubectl

    chmod +x kubectl

    mv kubectl /usr/local/bin/



**Installing Docker Desktop**

**Note**: We need to install WSL2 on windows system

Start a PowerShell Administrator privileges and run the following to install WSL2:

    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart


**Restart the windows machine then download the WSL2 from WSL2**

Now open the PowerShell with admin privileges:

**Set WSL2 as the default in windows**

    wsl --set-default-version 2

Lastly, grab a Linux distro from the Microsoft Store. If you don’t know which to choose, just go with Ubuntu 20.04. 
You can always change it later. If you don’t have access to the Microsoft Store (e.g. it’s blocked on a work system), 
you can find manual install instructions here.

At this point you may as well launch and set up your distro, which will likely be nothing more than creating an account 
name and password. If you have a common username on other systems that you will likely be connecting to, it may be desirable 
to use that as your username in your Linux distro, but ultimately, you can use whatever username you want.


**Docker Desktop**

We can go to the main product page and look for a specific version. Whichever way we install it, make sure you select the 
option to install the WSL2 components.

**Now reboot the system one more time**

**Tip**: If we don’t plan on using Docker all the time, we may wish to adjust the settings to not have it start when we 
log in, it will save the resources also.

**Note**: If we have Docker on our laptop, then we can use the k3d tool, hosted by Rancher Labs. It installs a lightweight 
version of Kubernetes called k3s and runs it within a Docker container, meaning it will work on any computer that has Docker.



**Installing k3d**

![image](https://user-images.githubusercontent.com/38450758/209396479-42ba05d9-4860-4dc4-897d-e7d23c62bf55.png)
The easiest way to get k3d running on Windows is with Chocolatey. To install Chocolatey package, we can run the following 
commands with admin privileges in PowerShell:

    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

**Now close PowerShell and open a new administrative instance and run the following to install k3d and a couple other useful tools:**

    choco install k3d -y

    choco install jq -y

    choco install yq -y

    choco install kubernetes-helm -y


**Now let’s configure tab completion for k3d:**

**Create user profile file if it doesn't exist**

    if ( -not ( Test-Path $Profile ) ) { New-Item -Path $Profile -Type File -Force }

**Append the k3d completion to the end of the user profile**

    k3d completion powershell | Out-File -Append $Profile

**To see running containers**

    $ docker container ps       

![image](https://user-images.githubusercontent.com/38450758/209396584-e54928f3-ceff-41b1-87a1-316bc1f56907.png)
1.	Rancher/k3d-proxy: This is a Helper Container Image that supports some functionality of k3d

2.	Rancher/k3s:latest: Servers and Agents


**Create cluster using k3d in power shell without admin privileges or use Gitbash for better result**

    $ k3d cluster create <Cluster-Name>

    $ Move-Item ~\.kube\config.k3d* ~\.kube\config -Force


**List the Kubernetes nodes**
    
    $ kubectl get nodes --output wide
    
![image](https://user-images.githubusercontent.com/38450758/209396650-0de22e5f-72ed-4ccc-a948-585ffa68eda6.png)

 
**All Kubernetes cluster nodes run as containers in docker.**

    $ docker container ls
    
![image](https://user-images.githubusercontent.com/38450758/209396693-1c76637d-e0ff-4081-a8a5-6c4cf395c91d.png)
We can see two containers by default: a k3s instance and the k3d proxy. The k3d proxy is used to route traffic
in to the API server, which we can see configured by looking at ~/.kube/config, where we may see something like 
server: https://0.0.0.0:52038, which would be the same port that Docker shows as routing to port 6443 of the k3d 
proxy container.

However, the main reason I want to use k3d instead of minikube is that I want to have multiple nodes so that I can 
realistically test out taints, tolerations, affinity rules, etc. To create a multi-node system. 


**Delete the existing cluster**
    
    $ k3d cluster delete <Cluster-Name>


**Create a new multi-node cluster**
    
    $ k3d cluster create <Cluster-Name> --servers 2 --agents 3 --port "8888:80@loadbalancer" --port "8889:443@loadbalancer"

**Rename the config if needed**
    
    $ if ( Test-Path ~\.kube\config.k3d* ) { Move-Item ~\.kube\config.k3d* ~\.kube\config -Force }

**List the Kubernetes nodes**
    
    $ kubectl get nodes --output wide

**We can see the nodes in docker**
    
    $ docker container ls

<br>

**If you want to deploy OpenFaaS (Serverless) in this kubernetes cluster then follow below link**

Click here [How to deploy OpenFaaS](https://github.com/santoshcisco/OpenFaaS/blob/main/README.md)

<br>
<br>
<br>
<br>
