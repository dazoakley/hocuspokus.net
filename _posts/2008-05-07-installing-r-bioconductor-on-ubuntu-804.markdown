---
layout: post
title: Installing R/BioConductor on Ubuntu 8.04
permalink: /2008/05/installing-r-bioconductor-on-ubuntu-804
tags:
- bioconductor
- bioinformatics
- howto
- linux
- r
- ubuntu
---

The new version of Ubuntu is out, (as if you haven't heard that by now), so that means a fresh install to
play about with and working just the way I want! :D

One of the tools that I currently need (for the thesis work) is [R](http://www.r-project.org/) and the
[BioConductor](http://www.bioconductor.org/) libraries. So here's a quick run down on getting them up and
installed on Hardy...

First up, run these commands in a terminal:

{% highlight bash %}
sudo apt-get install build-essential g77 gfortran
sudo apt-get install refblas3 refblas3-dev zlib1g-dev
sudo apt-get install r-base
{% endhighlight %}

This will then install the R base packages and some of the BioConductor packages, along with the gcc and fortran compilers and some other libraries that will be required for the next step.

{% highlight bash %}
sudo -s
R
{% endhighlight %}

Now at the R prompt, type the followingâ€¦

{% highlight r %}
source("http://www.bioconductor.org/biocLite.R")
biocLite()
{% endhighlight %}

Now sit back for a few minutes while your system configures BioConductor for you.
