## 3. Exploring a bioinformatics container

 [\<\< Episode 2](2.containers.md)
 | [TOC](README.md) |
 [Episode 4 \>\>](4.wgs_workflow.md)
______


### Objectives

This episode will detail the process of looking for a bioinformatics container in a web repository,
inspecting it and using it.


---
### Looking for a bioinformatics container

Move in the directory relevant to this Episode:

    cd bio-workshop-18/episode3_fastqc_container/

Imagine we want to run a bioinformatics application using a container,
for instance [FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) for quality control of raw sequence data.

As mentioned in [Episode 2](2.containers.md),
a container project of very good quality for bioinformatics is [Biocontainers](https://github.com/BioContainers/containers).
There are two main Web registries that host images from Biocontainers:
- [QUAY Hub](https://quay.io) : `quay.io/` prefix for the repository
- [Docker Hub](https://hub.docker.com) : default with `docker`

With the web browser, go to https://quay.io and use the search box on the top right to look for `fastqc`;
if the search box is not visible, make the browser window wider and make sure you are using the Desktop version of the website.
The first hit should be `biocontainers/fastqc`. Click on the *Tags* button on the left to see a list of available versions.
At the time of writing, the most recent one has the tag `0.11.7--4`.

Now, let us go back to our terminal, and `pull` the image we just identified on QUAY Hub:

    docker pull quay.io/biocontainers/fastqc:0.11.7--4

Note how we had to specify all the ingredients for the repository: registry, author, package, tag.
Unlike in Docker Hub, in QUAY Hub there is usually no `latest` tag in the repositories, so the tag name always needs
to be specified.

Also note that, during the Pawsey workshop, you might get messages saying `Already exists`, with the pull taking a very short time.
This would happen because the container images were pre-loaded in the cloud virtual machines, to save time during the workshop.


---
### Inspecting the container

It is always a good idea to quickly inspect a newly pulled container, and `bash` is certainly useful to this end.
Let us run the image in interactive mode:

    docker run -it --rm quay.io/biocontainers/fastqc:0.11.7--4 bash

Now from inside the container, let us find out the version of FastQC:

    fastqc --version

As expected from the image tag, the output is:

    FastQC v0.11.7

Another good question is, where has the application been installed in the container?

    which fastqc

Now the output is:

    /usr/local/bin/fastqc

Knowing where the package is installed can be useful in certain situations, e.g. when package libraries need to be used.
Most of the biocontainer images in QUAY Hub have the applications installed in `/usr/local/`.
In contrast, in images from Docker Hub applications are often installed in `/usr/`.

We can now exit the container:

    exit


---
### Analysing raw sequencing data with the container

Let us get some raw sequencing data:

    cp -p ../data_files/SRR6166481_sub_1.fastq.gz .

Without containers, the simplest way to run a FastQC quality control analysis on the available input file would be:

    fastqc SRR6166481_sub_1.fastq.gz

How do we need to modify our command to use the containerised version of FastQC we pulled above?
We just need to prepend `docker run`, followed by the relevant **flags** (see [Episode 1](1.containers.md), we will be running in non-interactive mode) and the **full image name**:

    docker run --rm -v $(pwd):/data -w /data quay.io/biocontainers/fastqc:0.11.7--4 fastqc SRR6166481_sub_1.fastq.gz

Very often, yet not always, this is all we need to containerise the run of an application.
More examples will be proposed in [Episode 4](4.wgs_workflow.md).

Finally, remember how this type of commands can be embedded in a bash script,
such as the ones included in `episode3_fastqc_container/`.


---
### Conclusion

- When looking for container images for bioinformatics packages on the Web, the best places to search are [QUAY Hub](https://quay.io) and [Docker Hub](https://hub.docker.com);
- An image is copied locally from a Web repo using `docker pull <image path>`;
- Very often, a containerised package can be run straight away by modifying the original execution line as follows:

        docker run <docker flags> <container ID> <original line>


______
 [\<\< Episode 2](2.containers.md)
 | [TOC](README.md) |
 [Episode 4 \>\>](4.wgs_workflow.md)