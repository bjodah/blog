.. title: Dealing with temporaries in bash.rst
.. slug: dealing-with-temporaries-in-bash
.. date: 2016-01-06 02:42:10 UTC+01:00
.. tags: bash
.. category: 
.. link: 
.. description: 
.. type: text

It is quite common that I want to create a temporary file or directory during the
exectuion of a bash script. For this we may use ``mktemp`` and ``mktemp -d`` respectively.

The problem that arises is that you are not sure that your clean up code gets executed
if the user aborts the script by e.g. hitting Ctrl+c. An elegant solution was given here
http://unix.stackexchange.com/a/149093/119480

to illustrate the solution let's consider the following script:

.. code:: bash

   #!/bin/bash
   TMPDIR=$(mktemp -d)
   echo $TMPDIR
   cleanup() {
       rm -r $TMPDIR
       exit
   }
   trap cleanup INT TERM
   echo "A" | tee $TMPDIR/a.txt
   sleep 3  # Try hitting Ctrl-C here (use Ctrl-Z to inspect files)
   echo "B" | tee $TMPDIR/b.txt
   cleanup
   echo "C" | tee $TMPDIR/b.txt # will never be reached


The above code removes the temporary directory created at exit, no matter if the
script is allowed to finnish or aborted by the user invoking ``kill`` on the PID
(defaults to SIGTERM) or hits Ctrl+C (sends the SIGINT signal).
