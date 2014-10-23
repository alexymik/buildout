========================
GroupServer installation
========================

:Authors: `Michael JasonSmith`_; `Richard Waid`_; `Marek Kuziel`_; 
          `Alice Murphy`_; `Dan Randow`_
:Contact: Michael JasonSmith <mpj17@onlinegroups.net>
:Date: 2014-10-21 (see `History`_)
:Organization: `GroupServer.org`_
:Copyright: This document is licensed under a
  `Creative Commons Attribution-Share Alike 4.0 International License`_
  by `OnlineGroups.net`_.

..  _Creative Commons Attribution-Share Alike 4.0 International License:
    http://creativecommons.org/licenses/by-sa/4.0/

.. highlight:: console

Installation can be tricky: all of the above has to be configured
and set up correctly or installation will fail. We wrote this
documentation for people with moderate experience in Linux system
administration. If you get stuck, please ask us questions in
`GroupServer Development`_. Other more detailed guides would be
gratefully accepted.

.. _GroupServer Development: http://groupserver.org/groups/development

:Note: GroupServer is developed on `Ubuntu`_, and is know to run
       on `CentOS`_. We will gladly accept any modifications you
       have that will make GroupServer run on more platforms.

Quick start
===========

#.  Install the required packages (see `Requirements`_).
#.  Download the latest version of GroupServer from 
    <http://groupserver.org/downloads> and extract the archive
    (see `Download`_).
#.  Create a new hostname for your GroupServer site. Yes, you will need 
    a new one (see `Pick a Host Name`_).
#.  Edit ``config.cfg`` (see `Configure GroupServer`_).
#.  Enable prepared transaction support in PostgreSQL (see `Configure
    PostgreSQL`_).
#.  Run the following (see `Run the Installer`_)::

      $ ./gs_install_ubuntu.sh

#.  `Start Zope`_::

      $ ./bin/instance fg

#.  Visit your new site.

You can `remove GroupServer`_ in three steps.

Requirements
============

The installation script for Ubuntu will install all the libraries
listed below (see `Run the Installer`_). The packages that are
required to install GroupServer are listed for Ubuntu, CentOS,
and RedHat Enterprise Linux.

+-------------+--------------------------------------------------------+
|             | Packages                                               |
+-------------+--------------------------------+-----------------------+
| System      | Ubuntu                         | CentOS 6.1 or         |
|             |                                | RHEL 6.1              |
+=============+================================+=======================+
| Python      | ``python2.7``                  | ``python-devel``      |
|             +--------------------------------+-----------------------+
|             | ``python2.7-dev``              | ``python-setuptools`` |
|             +--------------------------------+-----------------------+
|             | ``python-virtualenv``          | See `CentOS and       |
|             |                                | RHEL`_ below    .     |
+-------------+--------------------------------+-----------------------+
| GNU C++     | ``g++``                        | ``gcc-c++``           |
+-------------+--------------------------------+-----------------------+
| Make        | ``build-essential``            | ``make``              |
+-------------+--------------------------------+-----------------------+
| PostgreSQL  | ``postgresql``                 | ``postgresql``        |
| Database    +--------------------------------+-----------------------+
|             | ``postgresql-server-dev-9.1``  | ``postgresql-server`` |
|             +--------------------------------+-----------------------+
|             | ``libpq-dev``                  | ``postgresql-libs``   |
|             +--------------------------------+-----------------------+
|             |                                | ``postgresql-devel``  |
+-------------+--------------------------------+-----------------------+
| Postfix     | ``postfix``                    | ``postfix``           |
| Email       +--------------------------------+-----------------------+
| Server      | ``postfix-dev``                |                       |
+-------------+--------------------------------+-----------------------+
| Redis       | ``redis-server``               | ``redis``             |
+-------------+--------------------------------+-----------------------+
| JPEG Support| ``libjpeg62-dev``              | ``libjpeg-devel``     |
+-------------+--------------------------------+-----------------------+
| zlib (PNG)  | ``zlib1g-dev``                 | ``zlib``              |
| Support     |                                +-----------------------+
|             |                                | ``zlib-devel``        |
+-------------+--------------------------------+-----------------------+
| SMTP Test   | ``swaks``                      | ``swaks``             |
+-------------+--------------------------------+-----------------------+

Download
========

GroupServer is distributed as a tar-file. To download the latest
version of GroupServer visit <http://groupserver.org/downloads>
and click ``Download``. You must then extract GroupServer from
the tar-file into a directory such as ``/opt``, ``/home`` or
``/usr/local``.

When the tar-file is extracted a new directory will be made.
This directory contains the configuration files for GroupServer.
In addition, the installation process will download and install
some dependencies into the directory (see `Run the Installer`_).
Finally, GroupServer will be run from the same directory (see
`Start Zope`_).

:Permissions: You may need to be the ``root`` user to extract the
       archive. If you do then you **must** change the ownership
       of the new GroupServer directory and all of its
       contents. GroupServer can only be run by users with normal
       privileges.

Set up
======

To install GroupServer you must get the following installed and
configured:

* **Postfix** handles both the incoming and outgoing email.
* **Zope** provides the web framework and basic web server support.
* **PostgreSQL** stores the posts.
* **ZODB** stores some web content and the user-information.
* **Redis** provides an application cache.

.. figure:: setup.*
   :width: 100%
   :alt: The architecture of GroupServer
   :align: center

   GroupServer listens for connections on a single port (8080, by
   default) and serves up either the administration interface
   (ZMI) or the normal web interface depending on the name used
   to connect to the web server (virtual hosting). Email comes
   into the server via the web interface, and goes out using
   SMTP. The data is stored in a variety of locations.

Setting up GroupServer is done in four steps: first `pick a host name`_,
then `configure GroupServer`_, `run the Installer`_ to install the 
system, and finally `start Zope`_.

Pick a host name
----------------

Your new site needs its own hostname. This is the name that is
typed into the *web browser* to access your new GroupServer
site. For a trial system, the name can be set up in the
``hosts(5)`` file.

#.  Edit ``/etc/hosts`` as ``root``.
#.  Add the new host name to the ``localhost`` entry, which is
    normally the first one. For example, to add the name
    ``gstest`` change the line to the following::

      127.0.0.1    localhost gstest

#. Save the ``hosts`` file.

Configure GroupServer
---------------------

The configuration of GroupServer is mostly carried out by modifying the
``config.cfg`` file, which is located in the root of the GroupServer 
folder [#cfgFile]_. First you must configure the `GroupServer Site`_
itself. Next the `Zope`_ system, which will run your GroupServer site, 
needs to be configured, before the `database storage`_.

GroupServer site
~~~~~~~~~~~~~~~~

You will need to check all the configuration for your initial site.

``host``
  The domain name used by your new GroupServer site. It must be
  the same as what you picked a host name earlier (see `Pick a
  host name`_).

``admin_email``
  When GroupServer is installed, an example site and group are
  created. So you can use the administration functions you must
  log in as an administrator. This is the email address of that
  administrator. Posts to the example group will be sent to the
  administrator at this address. This email address **must**
  work.
  
``admin_password``
  The password of the administrator of the new GroupServer site. The
  password will be used to log in, and can be changed after the site has
  been created.

``support_email``
  The email address where support messages are sent, and were email
  notifications are send from. For testing this can be set to your own
  email address.

``smtp_host``
  The SMTP host that will be used to send email from
  GroupServer. It defaults to ``localhost``, assuming you will be
  running Postfix on the same machine as GroupServer.

Zope
~~~~

Zope_ is used to provide the web-framework for GroupServer, and a
basic web-server. The server listens for connections on a single
port (the ``zope_port``) and provides the GroupServer UI if
connections are made using the ``host`` name, or the Zope
Management Interface (ZMI) if connections are made with the
``zope_host``.
 
The ``zope_host`` and ``zope_port`` are probably correct for a
simple test-system. However, for security we recommend you change
the name and password of the Zope administrator.

``zope_host``
  The name of the host that will run Zope. It defaults to the
  local machine (``localhost``).
  
``zope_port``
  The IP port that Zope will listen to. It defaults
  to 8080. (Zope will have to run as ``root`` to use port 80, and
  this is discouraged.)
  
``zope_admin``
  The name of the user who will administer Zope. This is used to
  log into the Zope Management Interface (ZMI).
  
``zope_password``
  The password for the Zope administrator. It can (and should) be
  changed after GroupServer has been set up.

:Note: The IP-address of the ``zope_host`` and ``host`` (see
       `GroupServer site`_) must be the same.

Database storage
~~~~~~~~~~~~~~~~

GroupServer stores most of its data in PostgreSQL. Two passwords need to be
set by you to protect this data.

``pgsql_password``
  The password required to attach to the PostgreSQL database. The install
  system will create a PostgreSQL database, and protect it with this
  ``pgsql_password``.
  
``relstorage_password``
  `The RelStorage system`_ will store data in a PostgreSQL database for
  Zope. This data is protected by the ``relstorage_password``.

.. _the RelStorage system: https://pypi.python.org/pypi/RelStorage

Configure PostgreSQL
--------------------

`The RelStorage system`_ that is used by GroupServer requires *prepared
transaction* support to be enabled in PostgreSQL. To enable prepared
transaction support carry out the following steps.

#. Edit the PostgreSQL configuration file. On Ubuntu you must be ``root``
   to edit this file, which is located at
   ``/etc/postgresql/9.1/main/postgresql.conf``.

#. Find the line that reads

   .. code-block:: cfg

     max_prepared_transactions = 0

#. Change the line to read

   .. code-block:: cfg

     max_prepared_transactions = 1

#. Restart PostgreSQL. On Ubuntu this is done using the following command::

     $ sudo service postgresql restart

Run the installer
=================

The installer for Ubuntu is a Bash script. (For `CentOS and
RHEL`_ you will have to carry out the steps by hand.) To run the
GroupServer installer enter the following command::

  $ ./gs_install_ubuntu.sh

You will be prompted for your password. This is required to check
that your Ubuntu system has met all the `requirements`_. Next the
installer ensures that the `set up`_ is correct.

The rest of the installation process should be completely
automatic. The system will create a *sandbox* for your
GroupServer site, with its own version of Python, placed in
``./bin/``. It will then configure the PostgreSQL databases that
store the data for your site. Finally, it will start the
`buildout`_ system that will **download** and install all the
requirements for GroupServer (around 43MB of packages) including:
  
* `eGenix.com mx Base`_ (4.4MB)
* `SQL Alchemy`_ (4.3MB)
* lxml_ (2.8MB)
* Pillow_ (2.4MB)
* `Zope 2.13`_ (1.4MB)

.. _eGenix.com mx Base: http://www.egenix.com/products/python/mxBase
.. _SQL Alchemy: http://www.sqlalchemy.org/
.. _lxml: http://lxml.de/
.. _Pillow: https://pypi.python.org/pypi/Pillow/2.3.1
.. _Zope 2.13: http://docs.zope.org/zope2/releases/2.13/

:Note: You need a functioning network connection to download the
       packages.

It is a good idea to make a cup of coffee, or go to lunch, while
buildout processes.

CentOS and RHEL
---------------

The process to install GroupServer on CentOS or RedHat Enterprise
Linux is manual. The basic idea is as follows, but it lacks
testing.

:Note: Commands that have to be run as ``root`` are shown on
       lines that begin with a ``#``. Commands that must be run
       as a normal user are shown on lines that begin with a
       ``$``.

#. Install the requirements_.

#. Create the two database users specified in ``config.cfg``,
   using ``createuser``::

     # createuser -D -S -R -l gsadmin
     # createuser -D -S -R -l gszodbadmin

#. Create the two databases specified in ``config.cfg`` using
   ``createdb``::

     # createdb -Ttemplate0 -O gsadmin -EUTF-8 groupserver
     # createdb -Ttemplate0 -O gszodbadmin -EUTF-8 groupserverzodb

#. Get the Python ``virtualenv`` package::

     # easy_install virtualenv

#. Set up a virtual Python environment for GroupServer::

     $ virtualenv --no-site-packages .

#. Grab the ``argparse`` module::

     $ ./bin/easy_install argparse==1.1

#. Fetch the system that builds GroupServer::

     $ ./bin/easy_install zc.buildout==1.5.2

#. Run the ``buildout`` process::

     $ ./bin/buildout -N

Start Zope
----------

Your GroupServer site is supported by Zope. To start Zope run the
following command::

  $ ./bin/instance fg

Zope will have started when the message ``Zope Ready to handle
requests`` is displayed in the terminal.

You should be able to view your GroupServer site at
`http://{host}:{zope_port}`. If you kept the defaults, the
address will be <http://gstest:8080>.
  
* The host is the one you picked earlier (see `Pick a Host
  Name`_).
* The port is the one that site listens to (see `Configure
  GroupServer`_).

Use ``Control-c`` to stop Zope.

:See also: The documentation *Starting and stopping GroupServer*
           has more information on running GroupServer, including
           running it as a daemon. We document the setup required
           to receive email with GroupServer in *Configuring
           Postifx*. Finally the steps required to configure a
           proxy is documented in *Configuring a web proxy*.

Remove Groupserver
==================

To remove GroupServer, you must remove the database, the
associated database user, and the directory that contains the
GroupServer install.

1. To remove the database, run the following commands::
  
    $ dropdb gstestdb -U postgres
    $ dropuser gstest -U postgres

  ``gstestdb``
    The name of the test database.
    
  ``postgres``
    The name of the admin of PostgreSQL.
    
  ``gstest``
    The name of the PostgreSQL user.

2. Remove the directory that contains the GroupServer install::

    $ rm -r groupserver-14.03

History
=======

======= ==========  ====================================================
Version Date        Change
======= ==========  ====================================================
14.xx   2014-10-21  Adding the setup diagram.
14.xx   2014-10-14  Reducing the number of ports to one.
14.06   2014-06-23  Moving the sections for configuring the proxy and
                    Postfix to their own documents.
14.03   2014-03-25  Clarifying the Requirements_ wording.
14.03   2014-03-20  Updating to Ouzo.
12.11   2012-11-27  Adding the sections `CentOS and RHEL`_ and
                    `Configure PostgreSQL`_.
12.11   2012-11-19  Adding a link to the Postfix documentation for 
                    Ubuntu.
12.11   2012-10-25  Removing some odd dependencies.
12.05   2012-04-30  Updating the `Configure GroupServer`_ and
                    `Run the Installer`_ sections.
12.05   2012-04-24  Removing unnecessary dependencies, and using 
                    ``pip`` in the *Run Buildout* section. 
11.08   2011-12-19  Adding the packages required for XML support and 
                    XSLT support on RHEL and CentOS to the list of
                    `Requirements`_.
11.08   2011-12-16  Added the CentOS packages to the list of
                    `Requirements`_, with much thanks to  `Patrick 
                    Leckey.
                    <http://groupserver.org/r/post/6Jfujbedywmu6Wtahz1PeL>`_
11.08   2011-11-15  Altering the `requirements`_ to switch the 
                    ``build-essential`` dependency to ``make`` on `the
                    advice of David Sturman.
                    <http://groupserver.org/r/post/1ezm2nM9kQHSJSOfn0Rsm0>`_
11.08   2011-10-27  Adding the `Download`_ section, and clarifying some
                    more of the documentation.
11.08   2011-10-26  Correcting some mistakes, and clarifying the 
                    documentation on `the advice of Ross Chesley
                    <http://groupserver.org/r/topic/4PF50PHIWeYtaMMzwG3624>`_
11.08   2011-09-01  Reordering the subsections of *Configure Zope*.
11.07   2011-07-08  Adding the ``build-essential`` depenency and the
                    cut-n-paste ``apt-get`` block to the `Requirements`_.
11.06   2011-07-05  Adding the prologue.
11.06   2011-07-04  Updating the notes, because of a change to the 
                    name of the initial GroupServer instance.
11.06   2011-06-17  Added postfix configuration and spooling notes.
11.05   2011-05-26  Fixed a typing mistake, and mentioned that the 
                    ``pgsql_dbname`` and ``pgsql_user`` had to be 
                    different.
10.09   2010-09-01  Changed how the configuration options are set.
1.0β²   2010-07-15  Improved the buildout instructions.
1.0β²   2010-07-07  Changed the Zope 2.10 (Python 2.4) instructions to
                    Zope 2.13 (Python 2.6) instructions.
1.0β    2010-06-04  Removed a duplicated instruction from the 
                    `Quick Start`_, and bumped the version number.
1.0α    2010-05-31  Typo and minor improvement fixes.
1.0α    2010-05-01  Fixes because upstream broke our buildout.
1.0α    2010-04-29  Better automatic configuration, so the Configure
                    GroupServer section has been removed.
1.0α    2010-04-28  Improved the documentation for ``gs_port`` and 
                    added documentation for the ``gs_admin`` and
                    ``gs_user`` configuration options.
1.0α    2010-04-23  Added a link to the downloads page. Clarified the
                    security changes that are made to PostgreSQL. 
1.0α    2010-04-06  Fixed some quoting in the requirements.
1.0α    2010-03-31  Fixed the `Requirements`_, added 
                    `Remove GroupServer`_ and `History`_
1.0α    2010-03-25  Fixed the config options, added `Quick Start`_
1.0α    2009-10-04  Updated to reflect the new egg-based system
======= ==========  ====================================================

Resources
=========

- Code repository: https://source.iopen.net/groupserver/
- Questions and comments to http://groupserver.org/groups/development
- Report bugs at https://redmine.iopen.net/projects/groupserver

.. [#cfgFile] The ``cfg`` files are interpreted by the Python 
   `ConfigParser`_ module, which accepts a syntax very similar to 
   Windows INI files.
.. _ConfigParser:
   https://docs.python.org/2/library/configparser.html

.. _GroupServer.org: http://groupserver.org/
.. _OnlineGroups.net: https://onlinegroups.net/
.. _Ubuntu: http://www.ubuntu.com/
.. _CentOS: http://centos.org/
.. _Buildout: http://www.buildout.org/en/latest/
.. _Zope: http://zope.org
..  _Michael JasonSmith: http://groupserver.org/p/mpj17
..  _Richard Waid: http://groupserver.org/p/richard
..  _Marek Kuziel: http://groupserver.org/p/marek
..  _Alice Murphy: http://groupserver.org/p/alice
..  _Dan Randow: http://groupserver.org/p/danr