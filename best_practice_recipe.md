




## summary 


- Checklist for writing a recipe

 1.  Document your dockerfile with labels and comments.
 2. When debugging use multiple RUN commands. When finalizing a recipe, condense separate RUN commands into single or a few RUN commands to minimize image size once the recipe is working.
 3. Ensure any security information is ephemeral, that is used and the deleted within a single RUN command.
 4. Clean the installation process, removing any build stages and caches.
 5. Abstract package versions, if possible
 6. Consider build reproducibility, if possible
 7. Consider adding runtime tests, if possible
 8. Know and set some useful environment variables

---


### adding comments

- For one, it should have a extensive set of labels:


```def
LABEL maintainer="Aardvark"
LABEL version="1.0.0"
LABEL tags="ubuntu/18.04"
LABEL description="This container provides the fortune, cowsay and lolcat commands. \
It will by default combine all these commands, piping the output from fortune to cowsay and \
add colour via lolcat. "

```

- Second, it will be easier to maintain if comments are added.

```def

# Use the ubuntu base image
FROM ubuntu:18.04

# Adding labels
LABEL maintainer="Pawsey Supercomputing Centre"

# Use apt-get to install desired packages
RUN apt-get -y update && \
  apt-get -y install fortune cowsay lolcat
```


---

## Debugging with mutiple RUN and finalizing with single RUN


- Finally, it is good practice to split any complex steps when building a container into separate RUN commands 
- but then once a built works as desired to **join everything into** a simple run command 
- and take care to clean up any unnecessary files that were used in building applications. 

- This reduces the container size. 

- A pedagogical example would be:


```
RUN apt-get -y update && \
  apt-get -y install cowsay fortun lolcats
```
- Here there are two typos it might be easier to have each install command on a single line:

```
RUN apt-get -y update
RUN apt-get -y install cowsay
RUN apt-get -y install fortun
RUN apt-get -y install lolcats
RUN apt-get -y install lolcats
```

- The container also **contains all the files** need to run apt-get and recently cached files. These are **unlikely to be used** when running the container so the recipe should also remove them once they have been used.


```
# Use apt-get to install desired packages
RUN apt-get -y update && \
  # install packages \
  apt-get -y install fortune cowsay lolcat \
  # and clean-up apt-get related files \
  && apt-get clean all \
  && rm -r /var/lib/apt/lists/*

```

---

## Ensuring no security information present in container


## Ensuring reproducibility

- A big part of using containers is runtime reproducibility.

- However, it can be worth reflecting on build time reproducibility, too. 

- In other words, **if I re-run an image build** with the same Dockerfile over time, am I getting the same image as output? 
 
- This can be important for an image curator, to be able to guarantee to some extent that the image they’re build and shipping is the one they mean to.

- Consider:

    1.  relying on explicit versioning of the key packages in your image as much as possible.
    This may include OS version, key library and key dependency versions, end-user application version (this latter is possibly obvious); typically this involves a trade-off between versioning too much and too little;
    2. avoid **generic package manager update commands**, which make you lose control on versions.
    
     - In particular, you should avoid **apt-get upgrade**, **pip install --upgrade**, **conda update** and so on;

    3. avoid downloading **sources/binaries** for latest versions, **specify the version number** instead.


---


## Using environment variables

- Dockerfile installations are **non-interactive by nature**,
-  i.e. no installer can ask you questions during the process. 
- In Ubuntu/Debian, you can define a variable prior to running any apt command,
-  that informs a shell in Ubuntu or Debian that you are not able to interact, 
- so that no questions will be asked:

```
ENV DEBIAN_FRONTEND="noninteractive"
```


- Another pair of useful variables, again to be put at the beginning of the Dockerfile, are:

```
ENV LANG="C.UTF-8" LC_ALL="C.UTF-8"
```

- These variables are used to specify the language localisation, or locale, to the value C.UTF-8 in this case.

- Leaving this undefined can result in warnings or even in unintended behaviours for some programs (both at build and run time).



---

## Including runtime tests

Although not all containers make use of external libraries for functionality or performance, there are a number of uses cases where it is very useful **to provide separate tests** within a container. 

- A prime example is where containers make use of **MPI libraries**.

-  In (almost) all cases, the container should use the MPI libraries of **the host system** rather than any provided within the container itself.!!!!

-  However, debugging MPI issues by running the container application may be tricky.

- Thankfully, there is already a **ready-to-use set of MPI tests**, the OSU Micro-Benchmarks. 

- This package provides a large number of MPI related tests. 

### HOW????
- By comparing the results of these tests within a container to the same test running on the host system, you might be able to identify issues running MPI within the container:

```
# build desired ABI compatibile MPI library
# here this example shows how we might build OPENMPI
# first build openmpi
ARG OPENMPI_VERSION="4.1.1"
ARG OPENMPI_DIR="v4.1"
ARG OPENMPI_CONFIGURE_OPTIONS="--prefix=/usr/local"
ARG OSU_VERISON="5.9"
ARG OSU_CONFIGURE_OPTIONS="--prefix=/usr/local"
ARG OSU_MAKE_OPTIONS="-j${CPU_CORE_COUNT}"
RUN mkdir -p /tmp/openmpi-build \
      && cd /tmp/openmpi-build \
      && wget https://download.open-mpi.org/release/open-mpi/${OPENMPI_DIR}/openmpi-${OPENMPI_VERSION}.tar.gz \
      && tar xzf openmpi-${OPENMPI_VERSION}.tar.gz \
      && cd openmpi-${OPENMPI_VERSION}  \
      # build openmpi \
      && ./configure ${OPENMPI_CONFIGURE_OPTIONS} \
      && make ${OPENMPI_MAKE_OPTIONS} \
      && make install \
      && ldconfig \
      # remove the build directory now that the library is installed \
      && cd / \
      && rm -rf /tmp/openmpi-build \
      # now having built openmpi, build the osu benchmarks \
      # download, extract and build \
      && cd /tmp/osu-build \
      && wget http://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-${OSU_VERISON}.tar.gz  \
      && tar xzf osu-micro-benchmarks-${OSU_VERSION}.tar.gz \
      && cd osu-micro-benchmarks-${OSU_VERSION}  \
      && ./configure ${OSU_CONFIGURE_OPTIONS} \
      && make ${OSU_MAKE_OPTIONS} \
      && make install \
      && ldconfig \
      # remove the build directory now that the library is installed \
      && cd / \
      && rm -rf /tmp/osu-build
```

### a trick 

- The approach of having a simple test related to any Parallel API contained within the container may reduce the number of issues you will encounter deploying containers on a variety of systems.
-  It also maybe useful to even **add a script that reports the libraries used** by containerized applications at runtime:


```
#!/bin/bash
# list all applications of interest as space separated list
apps=(...)
# loop over all apps and report there dependencies
echo "Checking the runtime libraries used by :"
echo ${apps[@]}
echo ""
for app in ${apps[@]}
do
    echo ${app}
    ldd ${app}
done
```

- and have this script in /usr/bin/applications-dependency-check:

```
# add ldd script
ARG LDD_SCRIPT=./applications-dependency-check
RUN cp -p ${LDD_SCRIPT} /usr/bin/applications-dependency-check \
      && chmod +x /usr/bin/applications-dependency-check
CMD /usr/bin/applications-dependency-check
```


- The Docker instruction **CMD** can be used to set the default command that gets executed by docker run <IMAGE> (without arguments) or singularity run <IMAGE>. 

- If you don’t specify it, the default will be the CMD specified by the base image, or the shell if the latter was not defined. Such a script may even be a useful default command.









