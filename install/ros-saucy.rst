
============================
Install Instructions for ROS
============================

Installing on Ubuntu 13.10 (Saucy) or Linux Mint 16
---------------------------------------------------

ROS Hydro has no precompiled package for these OS, so it must be installed
from source. Please refer to the instructions at
http://wiki.ros.org/hydro/Installation/Source
http://wiki.ros.org/hydro/Installation/Debian

1. Install bootstrap dependencies

.. code:: bash

    sudo apt-get install python-rosdep python-rosinstall-generator \
        python-wstool python-rosinstall build-essential

2. Initialize rosdep

.. code:: bash

    sudo rosdep init
    rosdep update

3. Create a catkin workspace

.. code:: bash

    mkdir ~/ros_catkin_ws
    cd ~/ros_catkin_ws

- You can also make it at ``~/src/ros_catkin_ws``, as I do

4. Set up Desktop install. I first tried the Desktop-Full install, but was not
   successful because gazebo would not install.

.. code:: bash

    rosinstall_generator desktop --rosdistro hydro --deps --wet-only > \
        hydro-desktop-wet.rosinstall
    wstool init -j8 src hydro-desktop-wet.rosinstall

5. Resolve dependencies. On Ubuntu, the following should work (untested):

.. code:: bash

    rosdep install --from-paths src --ignore-src --rosdistro hydro -y

- On Linux Mint, I had to add the ``--os=ubuntu:saucy`` option.

.. code:: bash

    rosdep install --from-paths src --ignore-src --rosdistro hydro -y
        --os=ubuntu:saucy

6. At some point, shiboken may have been installed. This will cause problems
   during the build, so you should remove it now:

.. code:: bash

    sudo apt-get remove shiboken libshiboken*

- See https://github.com/ros-visualization/qt_gui_core/issues/35

7. Check that your libGL.so links are all set up correctly. One of my links
   was not set up correctly, revealed by executing

.. code:: bash

    ls -l /usr/lib/x86_64-linux-gnu/libGL.so

resulting in a red symbolic link (meaning it was dead). I replaced the link
by using the commands:

.. code:: bash

    cd /usr/lib/x86_64-linux-gnu
    sudo ln -sf ../libGL.so libGL.so

8. Finally, we can build and install:

.. code:: bash

     sudo ./src/catkin/bin/catkin_make_isolated --install
        --install-space /opt/ros/hydro_desktop_src

The ``--install-space /opt/ros/hydro_desktop_src`` flag will install to
``/opt/ros/hydro_desktop_src`` instead of to the source file location.
I do not put the files in ``/opt/ros/hydro``, since that would conflict
with the target of any ROS repository packages (if and when they are
made for Saucy).
