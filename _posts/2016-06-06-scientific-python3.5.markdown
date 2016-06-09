---
layout: post
title:  "Scientific Python3.5"
date:   2016-06-06 21:11:19 +0200
categories: python science coding set-up
---

## Intro


In two previous posts I discussed the compilation of
 [Python2.7](python/science/coding/set-up/2016/04/11/python-2.7-setup.html)
 and the [scientific Python2.7 set-up](python/science/coding/set-up/2016/04/12/scientific-python.html).
Due to some [valueble insights](https://wiki.python.org/moin/Python2orPython3#Which_version_should_I_use.3F),
 I recently decided to move all my projects to Python3.
While this is a fairly [straightforward process](https://docs.python.org/3/howto/pyporting.html) process,
 the python3 installation process differs slightly from its 2.7 counterpart.


## Pre-requisites

The requirements for Linux Mint (which basically is a less annoying version of Ubuntu 14.04),
 are quite similar to the Python3.5 set-up:

{% highlight bash %}
sudo apt-get install autotools-dev blt-dev bzip2 dpkg-dev g++-multilib gcc-multilib \
                     libbluetooth-dev libbz2-dev libexpat1-dev libffi-dev libffi6 \
                     libffi6-dbg libgdbm-dev libgpm2 libncursesw5-dev libreadline-dev \
                     libsqlite3-dev libssl-dev libtinfo-dev mime-support net-tools \
                     netbase python-crypto python-mox3 python-pil python-ply quilt \
                     tk-dev zlib1g-dev
{% endhighlight %}

However, we also need the latest tk/tcl and bz2 libraries.
Personally, I like to keep source code organised in `$HOME/.Source`, so that's
 where we'll start. 
The prefix (install) directory is  the `$HOME/.local` folder; feel free to change that
 to `/usr/local` or anything else you prefer.
So let's start with a short tk/tcl/bz2 install script:

{% highlight bash %}
LOC=$HOME/.local
SOURCED=$HOME/.Source
TVERSION=8.6.5
BZVERSION=1.0.6

#TCL
cd $SOURCED
wget http://prdownloads.sourceforge.net/tcl/tcl${TVERSION}-src.tar.gz
tar xvzf tcl${TVERSION}-src.tar.gz
cd tcl${TVERSION}/unix
./configure --prefix=$LOC |& tee config.log
make -j4 |& tee build.log
make install |& tee install.log

#TK
cd $SOURCED
wget http://prdownloads.sourceforge.net/tcl/tk${TVERSION}-src.tar.gz
tar xvzf tk${TVERSION}-src.tar.gz
cd tk${TVERSION}/unix
./configure --prefix=$LOC --with-tcl=$SOURCED/tcl${TVERSION}/unix |& tee config.log
make -j4 |& tee build.log
make install |& tee install.log

#BZ2
cd $SOURCED
wget http://www.bzip.org/${BZVERSION}/bzip2-${BZVERSION}.tar.gz
tar zxvf bzip2-${BZVERSION}.tar.gz
cd bzip2-${BZVERSION}
make -f Makefile-libbz2_so
make
make install PREFIX=$LOC
cp ./libbz2.so.${BZVERSION} ${LOC}/lib
ln -s ${LOC}/lib/libbz2.so.${BZVERSION} ${LOC}/lib/libbz2.so.1.0
{% endhighlight %}


## Download and compile python3.5


Now we can start with Python.

{% highlight bash %}
PVERSION=3.5.1

cd $SOURCED
wget https://www.python.org/ftp/python/${PVERSION}/Python-${PVERSION}.tar.xz
tar xvJf Python-${PVERSION}.tar.xz
cd Python-${PVERSION}
export CPPFLAGS="-I${LOC}/include"
./configure --prefix=$LOC |& tee config.log
make -s -j4 |& tee build.log
make install |& tee install.log
{% endhighlight %}


## Basic packages

Congratulations, you got Python3.5 installed.
Now let's install some basic packages.


{% highlight bash %}
pip3 install -U --user pip setuptools
pip3 install -U --user virtualenv
pip3 install -U --user cython
pip3 install -U --user pyqt5
{% endhighlight %}

### Scientific Python


## BLAS

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


## Numpy


After this we can install [numpy](http://www.numpy.org/):

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
python3 setup.py config  |& tee config.log
python3 setup.py build   |& tee build.log
python3 setup.py install |& tee setup.log
{% endhighlight %}


## SciPy


Now open a new terminal (easiest way to get rid of the temporary variables) and
 install [scipy](https://www.scipy.org/).

{% highlight bash %}
LOC=$HOME/.local
SOURCED=$HOME/.Source

cd $SOURCED
git clone https://github.com/scipy/scipy.git
cd scipy/
python3 setup.py config |& tee config.log
python3 setup.py build |& tee build.log
python3 setup.py install --prefix=$LOC |& tee setup.log
{% endhighlight %}

## Other packages

I usually add some other python libraries:

{% highlight bash %}
pip3 install --user ipython
pip3 install --user pandas
pip3 install --user nose
pip3 install --user h5py

{% endhighlight %}

This should leave you with a crisp, fresh python3 set-up
Let me know if you have suggestions or improvements.



*-jdcapa*
