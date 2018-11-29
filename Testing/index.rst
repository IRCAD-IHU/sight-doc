************
Testing
************

.. toctree::
    :maxdepth: 2

   

.. _CTest: http://cmake.org/
.. _CMake: http://cmake.org/
.. _CppUnit: https://sourceforge.net/projects/cppunit
.. _sight: https://git.ircad.fr/Sight/sight

sight_ uses CTest_ and CppUnit_ for unit testing.

Building
--------

When building sight_ with CMake_, you will need to enable the ``BUILD_TESTS`` option, e.g. with the ``-DBUILD_TESTS=ON`` command line option.

Launching unit tests
--------------------

In you build directory, you can launch the unit tests with the ``ctest`` command in the following way:

.. code-block:: shell

    # Launch the tests sequentially
    ctest .

    # Launch the tests using 4 jobs, similar to the -j option of make
    ctest -j 4 .

    # You can also use the make or ninja commands to do so
    make test

    ninja test

Additional data
---------------

Additional data need to be download to run all the unit tests. 
They are available on the repository `sight-data <https://git.ircad.fr/Sight/sight-data>`_. 
You can download the data on this repository with this `URL <https://git.ircad.fr/Sight/sight-data/-/archive/master/sight-data-master.tar.gz>`_ 
or with this git command 

.. code:: bash

    git clone --depth 1 https://git.ircad.fr/Sight/sight-data.git

You can then specify the directory, where the data are located, with the ``FWTEST_DATA_DIR`` environment variable.

