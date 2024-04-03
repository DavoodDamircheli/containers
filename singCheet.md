
```bash
singularity selftest -> run a selftest
singularity --version -> verify installation
which singularity -> verify installation

sudo singularity build --sandbox /tmp/debian docker://debian:latest ->  Build a base sandbox from DockerHub
sudo singularity exec /tmp/debian apt-get update -y ->  Make changes to it
sudo singularity exec --writable /tmp/debian apt-get install git->  Make changes to it
sudo singularity build /tmp/mondebian.simg /tmp/debian -> Build your custom image
sudo singularity exec /tmp/mondebian.simg ls -> Browse your custom image

no writeable!
sudo singularity build /tmp/ubuntusingular.simg ubuntu.def
sudo singularity exec /tmp/ubuntusingular.simg ls

writeable
sudo singularity build --sandbox /tmp/centossingular.simg centos.def
sudo singularity exec /tmp/centossingular.simg touch test1.txt

sudo singularity build --writable lolcow.img shub://GodloveD/lolcow
container is writable, but is still mounted as read-only when executed with commands such as run, exec, and shell

singularity run shub://ajreling/Singularity-CentOS
```

## Quick recap


|    to                    |     you need to |
|build a sandbox           | ```singularity build --sandbox..```|
|Modify a sandbox          |sudo singularity shell --writable...|
|Build an image from sandox|sudo sinulairty build... |



```
sudo singularity shell --writable --no-home -B /home/davood/projects/beta_perigrain_v2:ubuntu-min-tools/opt/periv2 ubuntu-min-tools
```


#Adding the Mirror and Installing
   33  sudo wget -O- http://neuro.debian.net/lists/xenial.us-ca.full | sudo tee /etc/apt/sources.list.d/neurodebian.sources.list
   34  sudo apt-key adv --recv-keys --keyserver hkp://pool.sks-keyservers.net:80 0xA5D32F012649A5A9
   35  sudo apt-get update
   36  sudo apt-get install -y singularity-container
   37  singularity --version
   
#Installing from git
 git clone https://github.com/singularityware/singularity.git
	cd singularity
	./autogen.sh
	./configure --prefix=/usr/local
	make
	sudo make install
-------------------------------------------------------------------------
#centos.def 

BootStrap: yum
OSVersion: 7
MirrorURL: http://mirror.centos.org/centos-%{OSVERSION}/%{OSVERSION}/os/$basearch/
Include: yum
%runscript
    echo "This is what happens when you run the container..."
%post
    echo "Hello from inside the container"
    yum -y install vim-minimal
    

cat examples/debian.def 
BootStrap: debootstrap
OSVersion: stable
MirrorURL: http://ftp.us.debian.org/debian/
%runscript
    echo "This is what happens when you run the container..."
%post
    echo "Hello from inside the container"
    apt-get update
    apt-get -y install vim


$ cat examples/docker.def 
BootStrap: docker
From: ubuntu:latest
IncludeCmd: yes
%runscript
    echo "This is what happens when you run the container..."
%post
    echo "Hello from inside the container"
    echo "Install additional software here"
        
-------------------------------------------------------------------------

