---
layout: post
title:  "Dockerize Data Science Project"
date:   2019-04-15 20:15:33 +0700
categories: [Data Engineering]
---

 This tutorial is similar to the docker introduction found on this [website](https://docs.docker.com/get-started/part2/). It has been expanded to include multiple files and scripts/csv files for data analysis. 

#### What is Docker?

 Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and ship it all out as one package.

#### Why use Docker?

 Your code works on your machine, but doesn't work on another person's machine. A reason for this is that the other person's machine doesn't have libraries or dependencies that are as up-to-date as yours. By using containers, your application will run on any other Linux machine regardless of any customized settings that machine might have that could differ from the machine used for writing and testing the code.

#### Assumptions

Let's assume we have a local folder called  `Analysis`. In it are two scripts: `Analysis_1.py` and `Analysis_2.py`. 

We want to create a container that executes both these scripts and outputs the results. 

We also have our csv files found in folder `data`. `data` is on the same folder level as `Analysis`. 

#### 1. Install Docker

Instructions [here](https://docs.docker.com/get-started/) help get Docker installed 

#### 2. Add requirements.txt and entrypoint.sh

Below are libraries to include for our container. These are saved in a file `requirements.txt`

{% highlight shell %}
pandas
seaborn
scipy
xlrd
matplotlib
{% endhighlight %}

Because we want to run multiple scripts in our file, we'll need a bash file for our container to execute. This is saved in a file `entrypoint.sh`.

{% highlight shell %}
python Analysis.py
python Analysis_2.py
{% endhighlight %}

#### 3. Configure Dockerfile

Create an empty directory on your local machine. Change directories (cd) into the new directory, create a file called Dockerfile, copy-and-paste the following content into that file, and save it. 

{% highlight shell %}
#start with 3.7.2 version of python
FROM python:3.7.2

MAINTAINER Hari Devanathan <hd2zm@tailordev.fr>

#ENV PYTHONUNBUFFERED 1

ADD Analysis/Analysis.py  Analysis.py 
ADD Analysis/Analysis_2.py Analysis_2.py 

#update packages instlaled in the os
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libatlas-base-dev gfortran

#Copy the list of python packages to install
COPY requirements.txt requirements.txt

#Copy all government data used
COPY ./data ./data

#Now install the python packages
RUN pip install -r requirements.txt

#Run script
ENTRYPOINT [ "entrypoint.sh" ]

#Allow access on port 8888
EXPOSE 8888
{% endhighlight %}


#### 4. Build Dockerfile

At the top level of your directory, you should have the following when you do `ls`.

{% highlight shell %}
Analysis
data
Dockerfile
requirements.txt
entrypoint.sh 
{% endhighlight %}

Run this command to create a container called `dataanalysis`

{% highlight shell %}
$ docker build --tag=dataanalysis
{% endhighlight %}

To see if the image is created, run `$ docker image ls`. 

{% highlight shell %}
$ docker image ls

REPOSITORY            TAG                 IMAGE ID
dataanalysis        latest              326387cea398
{% endhighlight %}

You can now run `$ docker run -p dataanalysis`, and share the output. 