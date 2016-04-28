---
layout: post
title:  "python 2.7.11 set-up"
date:   2016-04-11 21:11:19 +0200
categories: python science coding set-up
---

building python from source
---------------------------

intro
=====

In the following I will describe my scientific python set-up.
I'm currently working with *Linux Mint 17*. 
This OS is shipped with Python 2.7.6; yet I like the additional 
 features
 of *Python 2.7.11* (once I updated my own programs I will shift to Python 3) and
 thus, this guide will be about the set-up of this very version.

pre-requisites
==============

So lets get started; we'll begin with installing compiling Python 2.7.11, which
 requires a bunch of pre-requisites. 
For Linux Mint (which basically is a less annoying version of Ubuntu 14.04), 
 we require:

{% highlight bash %}
sudo apt-get install autotools-dev blt-dev bzip2 dpkg-dev g++-multilib gcc-multilib \
                     libbluetooth-dev libbz2-dev libexpat1-dev libffi-dev libffi6 \
                     libffi6-dbg libgdbm-dev libgpm2 libncursesw5-dev libreadline-dev \
                     libsqlite3-dev libssl-dev libtinfo-dev mime-support net-tools \
                     netbase python-crypto python-mox3 python-pil python-ply quilt \
                     tk-dev zlib1g-dev
{% endhighlight %}

download and compile python
===========================

Personally, I like to keep source code organised in `$HOME/.Source`, so that's
 where we'll start. Here is a the install script which installs Python into 
 my `$HOME/.local` folder.

{% highlight bash %}
LOC=$HOME/.local
SOURCED=$HOME/.Source
PVERSION=2.7.11

cd $SOURCED
wget https://www.python.org/ftp/python/${PVERSION}/Python-${PVERSION}.tar.xz
tar xfz Python-${PVERSION}.tar.xz
cd Python-${PVERSION}
./configure --prefix=$LOC --enable-ipv6

make -s -j4 |& tee compile.log
make install |& tee install.log
{% endhighlight %}

bootstrapping
============

Now we need to bootstrap and update pip (and setup-tools) and prepare the 
 virtual environment.

{% highlight bash %}
cd $LOC
bin/python -m ensurepip --user --upgrade
bin/pip install -U pip setuptools
bin/pip install -U --user virtualenv
bin/pip install -U --user virtualenvwrapper
bin/pip install --user cython
{% endhighlight %}


setting up the virtual environment
==================================

As you can see, in my 
 [.bashrc](https://github.com/jdcapa/bashrc.d/blob/main/05.ENV_PYTHON),
 I added the following:

{% highlight bash %}
# <.bashrc file content>
if [ -e $HOME/.virtualenvs ]; then
    export WORKON_HOME=$HOME/.virtualenvs
    source ${PYTHON_LOC}/bin/virtualenvwrapper.sh
    export PROJECT_HOME=$HOME/.devel
fi
{% endhighlight %}

This ensures that whenever my bash is sourced, I'm in my local Python 
 environment. 
The virtualenv programme also allows you to create new temporary environments 
 for your personal enjoyment.

The command `python --version` should now return 'Python 2.7.11'. 
Make sure you reload your `.bashrc`


scientific python
-----------------

intro
=====

While I like Python as a language, it would be nothing without the broad support
 of the scientific community.
Here, the `numpy`, `scipy` and `matplotlib` libraries are what separates python
 from the rest.
Where the two former packages linear algebra (heavy mathematical operations) can
 be  accelerated to a C-like level, while keeping the beauty of the python
 code.
The latter takes care of visualisation and beats most other (even commercial) 
 packages when it comes to customisability.


BLAS
====

Anyway, let's make it happen.
For both `numpy` and `scipy` we need a
 [BLAS](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms)
 library.
Head over to [Intel](https://registrationcenter.intel.com/en/forms/?productid=2558&licensetype=2).
At the time of writing they offer a free public licence on their MKL libraries.
While there are other open source options, these are less troublesome and easier
 to set up (it requires registration).
*Open source options are 
 [Open BLAS](https://hunseblog.wordpress.com/2014/09/15/installing-numpy-and-openblas/)
 and 
 [ATLAS](http://williambert.online/2012/03/how-to-install-accelerated-blas-into-a-python-virtualenv/),
 for which an extra set of libraries (LAPACK and CBLAS) are required.*

I installed the Intel MKL libraries (version 11.3.2.181 in /opt/intel) and 
 sourced their environment set-up script in my 
 [.bashrc](https://github.com/jdcapa/bashrc.d/blob/main/05.ENV_MKL).

{% highlight bash %}
# <.bashrc file content>
if [ -e /opt/intel/mkl/bin/mklvars.sh ]; then
    source /opt/intel/mkl/bin/mklvars.sh intel64
    INTELMKL=/opt/intel/mkl/lib/intel64
    LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$INTELMKL  # This gets exported at a later stage.
fi
{% endhighlight %}


numpy
=====
After this we can install numpy:


{% highlight bash %}
LOC=$HOME/.local
SOURCED=$HOME/.Source

cd $SOURCED
git clone https://github.com/numpy/numpy
cd numpy
cp site.cfg.example site.cfg
{% endhighlight %}

The `site.cfg` needs to be edited to include your BLAS set-up, which for me meant
 adding the following to account for the Intel MKL libraries.

{% highlight bash %}
# <site.cfg file content>
[mkl]
library_dirs = /opt/intel/mkl/lib/intel64
include_dirs = /opt/intel/mkl/include
mkl_libs = mkl_rt
lapack_libs =
{% endhighlight %}

Since I'm using the GNU g++ and gfortran compilers I needed to adjust the 
 compiler and library flags according to 
 [Intels suggestions](https://software.intel.com/en-us/articles/numpyscipy-with-intel-mkl).

{% highlight bash %}
export CFLAGS="-fopenmp -m64 -mtune=native -O3 -Wl,--no-as-needed"
export CXXFLAGS="-fopenmp -m64 -mtune=native -O3 -Wl,--no-as-needed"
export LDFLAGS="-ldl -lm"
export FFLAGS="-fopenmp -m64 -mtune=native -O3"
{% endhighlight %}



Finally we can compile and install numpy.

{% highlight bash %}
python setup.py config  |& tee config.log
python setup.py build   |& tee build.log
python setup.py install |& tee setup.log
{% endhighlight %}

scipy
=====

Now open a new terminal (easiest way to get rid of the temporary variables) and
 install `scipy`.

{% highlight bash %}
LOC=$HOME/.local
SOURCED=$HOME/.Source

cd $SOURCED
git clone https://github.com/scipy/scipy.git
cd scipy/
python setup.py config  |& tee config.log
python setup.py build   |& tee build.log
python setup.py install |& tee setup.log
{% endhighlight %}

other packages
==============

I usually add some other python libraries:

{% highlight bash %}
pip install --user ipython
pip install --user matplotlib
pip install --user h5py
pip install --user pandas
pip install --user nose
pip install --user pyyaml
{% endhighlight %}

This should leave you with a crisp, fresh python set-up
Let me know if you have suggestions or improvements.

*-jdcapa*
