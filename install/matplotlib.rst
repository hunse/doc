
=====================
Installing Matplotlib
=====================

First, install the dependencies

.. code:: bash

    sudo apt-get install \
        libfreetype6 libfreetype6-dev \
        libpng12-0 libpng12-dev \
        python-tk tk8.5 tk8.5-dev python3-tk


Next, download the matplotlib source to a folder like ``~/src``:

.. code:: bash

    git clone https://github.com/matplotlib/matplotlib


Add the following lines to the file ``setup.cfg``:

.. code:: aconf

    [gui_support]
    tkagg = True

    [rc_options]
    backend = TkAgg


Run the following commands to build matplotlib:

.. code:: bash

    python setup.py config
    python setup.py build


The command to install depends on your environment:

1. In a virtualenv, do
    ``python setup.py install``
2. To install to your user directory, do
    ``python setup.py install --user``
3. To install to your ``/usr`` folder, do
    ``sudo python setup.py install``
