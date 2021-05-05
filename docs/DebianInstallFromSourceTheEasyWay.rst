Debian Install From Source - The Easy Way
=========================================

.. contents:: **Contents**


Introduction
------------

This guide will show only the required methods / commands to get ``rTorrent-PS-CH`` and `pyrocore <https://github.com/pyroscope/pyrocore/>`_ utilities up and running in `tmux <https://github.com/tmux/tmux>`_ within 20 minutes. It's really easy with the help of the amazing build script created by `@pyroscope <https://github.com/pyroscope>`_.

Note that building process of ``rTorrent-PS-CH`` has been changed (simplified) as of ``v1.6.0``, still installing a compiled build into the `system <#install-it-into-system>`_ in the end is the preferred way.


Installing build dependencies
-----------------------------

First, you need to install a few **required** packages — **and no, this is not optional in any way**. They require about ``300 MB`` disk space. These steps must be performed by the ``root`` user (i.e. in a root shell, or by writing ``sudo`` before the actual command):

.. code-block:: shell

   :installing python2
   sudo apt-get update
   sudo apt-get install sudo tmux coreutils binutils build-essential subversion git time \
       chrpath pkg-config libssl-dev libncurses5-dev libncursesw5-dev locales libcppunit-dev \
       zlib1g-dev autoconf automake libtool libxml2-dev libxslt1-dev curl ruby ruby-dev mc \
       python python-dev python-setuptools python-pkg-resources virtualenv
   curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
   sudo python2 get-pip.py
   sudo gem install fpm


Note about ``tmux``
^^^^^^^^^^^^^^^^^^^

After that all you need is to place a `.tmux.conf <https://raw.githubusercontent.com/chros73/rtorrent-ps-ch_setup/master/ubuntu-14.04/home/chros73/.tmux.conf>`_ like that in your home directory (can be ``root`` or a regular user) and you can already run ``tmux``: 

.. code-block:: shell

   cd ~; curl -sLSO https://raw.githubusercontent.com/chros73/rtorrent-ps-ch_setup/master/ubuntu-14.04/home/chros73/.tmux.conf
   tmux -2u

There is only 1 crucial option (along with the other useful ones) in that ``.tmux.conf``: ``set -g default-terminal "screen-256color"``. This is responsible for getting 256 color support. **No matter** what other tutorials / guides / intructions say about the ``TERM`` environment variable: **you shouldn't set it**! You will experience strange rendering problems! Although this will result that ``rTorrent-PS-CH`` won't start in a terminal window, but that's not a problem since we always run it in ``tmux``. (`Read more <https://sanctum.geek.nz/arabesque/term-strings/>`_ about it.)

You can also build the latest `CPU optimized version of tmux <https://github.com/chros73/build-tmux/>`_.


Compiling and installing ``rTorrent-PS-CH`` from source
-----------------------------------

Multiple versions can be supported or just the latest stable ``git`` version. If patches for multiple version can be found inside the ``patches`` directory then ``build.sh`` will compile the latest release version, in this case ``git`` option can be passed to it to build ``git`` version. If ``only_git_lt_rt`` is set to ``true`` in ``build.sh`` script then it will always build ``git`` version.


Compiling it
^^^^^^^^^^^^

It has to be compiled first preferebly with a regural user. Build script creates local CPU optimized code by default (``+`` sign is displayed in the middle of title bar of ``rTorrent-PS-CH`` instead of ``-``).

It will build ``rTorrent-PS-CH`` binary including some libraries into ``~/lib/rtorrent-ps-ch*`` directory and create symlinks to it in ``~/bin/`` and ``~/lib/`` directories.

.. code-block:: shell

   mkdir -p ~/src/; cd ~/src/
   git clone https://github.com/chros73/rtorrent-ps-ch.git
   cd rtorrent-ps-ch
   time nice -n 19 ./build.sh ch


Install it into system
^^^^^^^^^^^^^^^^^^^^^^

You need ``root access`` for this.

It installs (copies) the compiled ``rtorrent-ps-ch*`` directory into ``/opt/`` directory and creates symlinks to it in ``/usr/local/bin/``, ``/usr/local/lib/`` and ``/opt/`` directories. (More `hint <http://web.archive.org/web/20171223124216/https://wiki.debian.org/sudo>`_ about ``sudo`` for Debian.)

.. code-block:: shell

   sudo ./build.sh install


Other tasks
^^^^^^^^^^^

The build script provides other useful tasks as well.


Compiling non CPU optimized builds
""""""""""""""""""""""""""""""""""

If you want to turn off optimization for some reason (e.g. packaging the build) it can be done by rebuilding it with:

.. code-block:: shell

   optimize_build=no time nice -n 19 ./build.sh ch


Compiling vanilla build
"""""""""""""""""""""""

You can even build an optimized version of vanilla ``rtorrent`` (only including necessary patches if there's any).

It will build the binary including some libraries into ``~/lib/rtorrent-ps-ch-vanilla*`` directory and create symlink to it in ``~/bin/`` directory. (Note that installing, packaging a vanilla build is not supported.)

.. code-block:: shell

   time nice -n 19 ./build.sh vanilla


Creating tarballs
"""""""""""""""""

You can ``create tarballs`` of an optimized/unoptimized/vanilla build from the ``~/lib/rtorrent-ps-ch*`` build directory if you like:

.. code-block:: shell

   ./build.sh pkg2tgz

You should copy the resulted ``*.tar.gz`` tarball from ``/tmp/rtorrent-ps-ch-dist`` to somewhere safe.


Creating deb package
""""""""""""""""""""

You can even ``create a package`` of an unoptimized (``optimize_build=no``), installed build with ``fpm`` if you like (so you can distribute it later):

.. code-block:: shell

   debfullname="yourname" debemail="youremailaddress" ./build.sh pkg2deb

You should copy the resulted ``*.deb`` package from ``/tmp/rtorrent-ps-ch-dist`` to somewhere safe.


Creating Arch Linux package
"""""""""""""""""""""""""""

You can also ``create a package`` of an unoptimized (``optimize_build=no``), installed build with ``pacman`` (``fpm`` from the AUR should be used!) if you like (so you can distribute it later):

.. code-block:: shell

   debfullname="yourname" debemail="youremailaddress" ./build.sh pkg2pacman

You should copy the resulted ``*.tar.xz`` package from ``/tmp/rtorrent-ps-ch-dist`` to somewhere safe.


If all went well
^^^^^^^^^^^^^^^^

Check the result by running ``rtorrent`` (you don't need a config file for this) in a ``tmux`` window, not terminal window!

You can delete the ``~/src/rtorrent-ps-ch/`` directory later if all went well with:

.. code-block:: shell

   cd ~ && [ -d ~/src/rtorrent-ps-ch/ ] && rm -rf ~/src/rtorrent-ps-ch/



Install ``pyrocore`` utils for regular user
-------------------------------------------

You should run these under your normal user account:

Install
^^^^^^^

.. code-block:: shell

   cd ~ && mkdir -p ~/bin ~/.local
   git clone "https://github.com/pyroscope/pyrocore.git" ~/.local/pyroscope
   ~/.local/pyroscope/update-to-head.sh
   touch ~/.bash_completion
   grep /\.pyroscope/ ~/.bash_completion >/dev/null || \
       echo >>.bash_completion ". ~/.pyroscope/bash-completion.default"
   . /etc/bash_completion

You can check whether all went well with:

.. code-block:: shell

   pyroadmin --version

Update
^^^^^^

If you want to update ``pyrocore`` utils later:

.. code-block:: shell

   cd ~/.local && tar -czf pyroscope-$(date +'%Y-%m-%d').tar.gz pyroscope    # make backup first
   cd ~ && ~/.local/pyroscope/update-to-head.sh                              # update it
   pyroadmin --version                                                       # check for success


Summary
-------

It's really that simple, it only took about 20 minutes.

