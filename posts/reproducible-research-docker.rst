.. title: Reproducible research with Docker
.. date: 2015-04-12 17:05:21 UTC+01:00
.. tags: docker, reproducible research
.. slug: reproducible-research-docker

When I analyse data in my research I often tend to write small scripts
for each of the steps. Those scripts often rely on 3rd party packages
being installed. It has happened more than once that a few years down
the road those scripts no longer work since the 3rd party package I
was depending on changed its API. Until now I have looked at it as a
hopeless (and time consuming) battle...


Docker to the rescue
--------------------
`Docker <docker.io>`_ is a tool which has gained a tremendous traction
in the web community recently. However, it can also be tamed
into a useful tool for scientific computing.

I will try to guide you through the steps necessary to set things
up. I will assume familiarity with Linux, git, bash and basic idea of
how Docker works.

Separate raw from generated data (input vs. output)
---------------------------------------------------
Here are a few things I found made my life much easier when trying to
do reproducible research (based on scripts):

1. Distinguish between input and output and make sure that generated
   data is built out-of-source. 
2. Use version control (from day one) for the input and the environment
   description.
3. Rebuild from scratch frequently to avoid introducing hidden
   dependencies. Preferably: for each commit on a different host
   (continuous integration)

We will start simple with a contrived example. Say we have:

1. Some raw data of relative humidity ("rh") that is believed to be
   normally distributed. 
2. A small Python script for calculating the mean and standard
   deviation (in reality the analysis might be something more novel).
3. A script for generating a "report" of said analysis.

Let's set things up from the command line:

::

    $ mkdir rh; cd rh
    $ mkdir input output environment
    $ echo output/ >.gitignore
    $ cat <<EOF >input/data.txt
    23.16
    22.87
    25.23
    25.01
    23.93
    24.56
    EOF


OK, the data is in place, let's write a script to do the analysis:

::

   $ cat <<EOF >input/analysis.py

.. code-block:: python

    #!/usr/bin/env python
    import numpy as np
    
    def stddev(srcpath='data.txt', format='{0:12.5g} {1:12.5g}'):
        arr = np.loadtxt(srcpath)
        avg = np.mean(arr)
        s = (np.sum((arr-avg)**2/len(arr)))**0.5
        return format.format(avg, s)

    if __name__ == '__main__':
        import argh
        argh.dispatch_command(stddev)

    EOF

::

    $ chmod +x ./input/analysis.py

Now let's write a script for generating the report.

::

    $ cat <<'EOF' >input/generate_report.sh

.. code-block:: bash

    #!/bin/bash
    AVG=$(./analysis.py --format '{0:12.5g}')
    DEV=$(./analysis.py --format '{1:12.5g}')
    echo "The average of the data was $AVG and the standard deviation
    was estimated to be $DEV">../output/report.txt
    EOF

::

    $ chmod +x ./input/generate_report.sh

Alright, now let's try to make make this procedure reproducible by
locking down all versions used (i.e. versions of Python, NumPy, bash).
We will do so by relying on a specific (long term support) Linux
distribution.

::

    $ cat <<'EOF' >environment/Dockerfile

.. code-block:: dockerfile

    FROM ubuntu:14.04.2
    ENV DEBIAN_FRONTEND noninteractive

    RUN apt-get update && \
        apt-get --quiet --assume-yes install \
        python-numpy python-argh && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
    EOF

And now let's write a small script orchestrating the full process:

::

   $ cat <<'EOF' >generate_output.sh

.. code-block:: bash

   #!/bin/bash -x
   MY_DOCKER_IMAGE="./environment"
   MY_HASH=$(docker build $MY_DOCKER_IMAGE | tee /dev/tty | tail -1 | cut -d' ' -f3)
   docker run --rm -v $(pwd)/input:/input -v $(pwd)/output:/output -w /input -i $MY_HASH ./generate_report.sh

   EOF

::

   $ chmod +x generate_output.sh

And that's it. Obviously the relative overhead of tracking all the
dependencies for such a small example is ridiculously high but most of
the above code is of boiler plate character and may easily be copied
between projects. Now let's make sure the script works (note that
docker requires root privileges so use ``sudo`` or add your user to
"docker" group):

::

    $ sudo ./generate_output.sh
    $ cat ./output/report.txt
    The average of the data was       24.127 and the standard deviation
    was estimated to be      0.88861

Great, so this would be a good point to set up version control,
e.g. by using git:

::

    $ git init .
    $ git add -A
    $ git commit -m "Initial commit"

This should make the analysis reproducible for the foreseeable future
(we are assuming that both Docker, the Ubuntu 14.04.2 image and the
Ubuntu 14.04 package repositories will stay around indefinitely which
probably isn't true).

Room for improvement
--------------------
By using docker we can get more benefits for free. We can avoid
involuntarily relying on internet access during the generation of our
output by passing the flag ``--net='none'`` to ``docker run``.

We can also force the ``./input`` directory to be read-only during the
build process to better enforce the distinction between raw and
generated data.

.. code-block:: bash

   docker run --rm -v $(pwd)/input:/input:ro -v $(pwd)/output:/output -w /input --net='none' -i $MY_DOCKER_IMAGE ./generate_report.sh

One thing you will soon notice is that docker runs as UID=0 (root),
which means that files generated in output will not be owned by your
current user. One way to circumvent this is to have a small script in
``input/`` setting the appropriate ownership after having run the
``./generate_report.sh`` script. We will need to provide our preferred 
UID and GID to the docker image through the use of environment
variables:

.. code-block:: bash

   docker run --rm -e HOST_UID=$(id -u) -e HOST_GID=$(id -g) -v $(pwd)/input:/input:ro -v $(pwd)/output:/output -w /input -i $MY_DOCKER_IMAGE ./entrypoint.sh

.. code-block:: bash

    #!/bin/bash
    # this is entrypoint.sh
    ./generate_report.sh
    chown -R $HOST_UID:$HOST_GID ../output



Installing latest docker on Ubuntu 14.04
----------------------------------------
To install the latest version of Docker in Trusty you may proceed as follows:

::

   $ wget -qO- https://get.docker.io/gpg | sudo apt-key add -
   $ sudo sh -c "echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
   $ sudo apt-get update
   $ sudo apt-get install lxc-docker apparmor
   $ sudo docker -d &
