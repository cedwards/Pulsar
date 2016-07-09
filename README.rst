HubbleStack Pulsar
==================

Pulsar is designed to monitor for file system events, acting as a real-time File 
Integrity Monitoring (FIM) agent. Pulsar is composed of a custom Salt beacon
that watches for these events and hooks into the returner system for alerting and 
reporting.

In other words, you can recieve real-time alerts for unscheduled file system modifications *anywhere* you
want to recieve them.

SPM Packages (Recommended)
==========================

Each of the four HubbleStack components have been packaged for use with Salt's
Package Manager (SPM). Note that all SPM installation commands should be done
on the *Salt Master*.

Required Configuration
----------------------

Salt's Package Manager (SPM) installs files into `/srv/spm/{salt,pillar}`.
Ensure that this path is defined in your Salt Master's `file_roots`:

.. code-block:: yaml

    file_roots:
      - /srv/salt
      - /srv/spm/salt

Note: Remember to restart the Salt Master after making this change to the
configuration.

Installation
------------

Installation is as easy as downloading and installing a package. (Note: in
future releases you'll be able to subscribe directly to our HubbleStack SPM
repo for updates and bugfixes!)

.. code-block:: shell

    wget https://spm.hubblestack.io/latest/hubblestack_pulsar-latest.spm
    spm local install hubblestack_pulsar-latest.spm

You should now be able to sync the new modules to your minion(s) using the
`sync_modules` Salt utility:

.. code-block:: shell

    salt \* saltutil.sync_beacons

Once these modules are synced you are ready to begin running the Pulsar beacon.

Installation (Manual)
=====================

Place ``pulsar.py <_beacons/pulsar.py>`` in your ``_beacons/`` directory in your Salt
fileserver (whether roots or gitfs) and sync it to the minion(s).

.. code-block:: shell

    git clone https://github.com/hubblestack/pulsar.git hubblestack-pulsar.git
    cd hubblestack-pulsar.git
    cp _beacons/pulsar.py /srv/salt/_beacons/
    salt \* saltutil.sync_beacons

Copy the ``hubblestack_pulsar.sls`` into your Salt pillar and target it to selected minions.

.. code-block:: shell

    base:
      '*':
        - hubblestack_pulsar

.. code-block:: shell

    salt \* saltutil.refresh_pillar

Usage
=====

Once Pulsar is fully running there isn't anything you need to do to interact
with it. It simply runs quietly in the background and sends you alerts. That's
all!

Pulsar Configuration
--------------------

The default Pulsar configuration (found in ``<pillar/hubblestack_pulsar.sls>``
is meant to act as a template. Every environment will have different needs and
requirements, and we understand that, so we've designed Pulsar to be flexible.

.. code-block:: yaml

    beacons:
       pulsar:
         /etc: { recurse: True, auto_add: True }
         /bin: { recurse: True, auto_add: True }
         /sbin: { recurse: True, auto_add: True }
         /boot: { recurse: True, auto_add: True }
         /usr/bin: { recurse: True, auto_add: True }
         /usr/sbin: { recurse: True, auto_add: True }
         /usr/local/bin: { recurse: True, auto_add: True }
         /usr/local/sbin: { recurse: True, auto_add: True }
         
         return:
           splunk:
             batch: True
           slack:
             batch: False
         stats: True
         batch: True
         checksum: sha256

Excluding Paths
---------------

There may be certain paths that you want to exclude from this real-time
FIM tool. This can be done using the ``exclude:`` keyword beneath any
defined path.

.. code-block:: yaml

    beacons:
       pulsar:
         /var: { recurse: True, auto_add: True }
           exclude:
             - /var/log
             - /var/spool
             - /var/cache
             - /var/lock

Contribute
==========

If you are interested in contributing or offering feedback to this project feel
free to submit an issue or a pull request. We're very open to community
contribution.
