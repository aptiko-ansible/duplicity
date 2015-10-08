=========
duplicity
=========

Overview
========

This is an Ansible role for making a server backup itself using
duplicity_.  Essentially it creates a ``custom_backup`` script which
is similar to duplicity but which automatically passes several options
to duplicity so that you don't need to specify them (backup location,
gpg options, and so on). So, for example, to restore file
``/one/two/three``::

    custom_backup restore --file-to-restore=one/two/three one/two/three

To restore to ``/var/tmp`` instead::

    custom_backup restore --file-to-restore=one/two/three /var/tmp/three

To restore as it was three days ago::

    custom_backup restore --time=3D --file-to-restore=one/two/three /var/tmp/three

Run ``custom_backup help`` and ``man duplicity`` for more options.

This role also sets up scheduled backups with cron.

You may want to generate GPG keys first; to do this run::

    gpg --gen-key

and answer the defaults (RSA and RSA, 2048 bits, does not expire; real
name "Backup for *hostname*"; email address "root@*hostname*"; no
comment; empty passphrase). Then, export these keys to files::

  gpg --export-secret-keys --armor root@$hostname >$hostname.key
  gpg --export --armor root@$hostname >$hostname.pub

Then specify their contents as the ``gpg_pub_key`` and
``gpg_priv_key`` variables; for example, in ``host_vars/myhostname``::

  gpg_pub_key: |
    -----BEGIN PGP PUBLIC KEY BLOCK-----
    Version: GnuPG v1.4.12 (GNU/Linux)

    mQENBFOMQ4wBCAC2jC3JkCGD3YRdoZ184FRYb4W9+wz1By5wE1b4mZN8mXZMAObU
    ...

You probably want to put the private key in the vault.

Selecting what to backup
========================

Short version: Everything will be backed up, except for stuff that is
reproduced during installation (such as ``/usr``), can be recreated
(such as ``/var/cache``), is temporary (such as ``/tmp`` and
``/var/tmp``), or should be backed up in another way (such as live
database files).

The ``/etc/duplicity/includes_excludes`` directory contains files.
Each file contains stuff in the following format::

  --exclude=/my/great/directory/but/not/this
  --include=/my/great/directory

and so on. (You can also use ``--exclude-device-files``,
``--include-file-list`, ``--exclude-file-list``, etc.; i.e., all the
options described in the FILE SELECTION section of the duplicity man
page.) The contents of all these files will be added to the duplicity
command line. By default, ``/bin``, ``/boot``, ``/dev``,
``/initrd.img``, ``/lib``, ``/lib32``, ``/lib64``, ``/lost+found``,
``/media``, ``/mnt``, ``/opt``, ``/proc``, ``/run``, ``/sbin``,
``/srv``, ``/sys``, ``/tmp``, ``/usr``, ``/var/cache``, ``/var/tmp``,
``/var/lib/postgres``, ``/var/lib/mysql`` and ``/vmlinuz`` are excluded,
and all the rest is included (backup databases with a pre- script that
dumps them in ``/var/backups``; see "Pre- and post- scripts" below).
The file that excludes these is
``/etc/duplicity/includes_excludes/base``.

If you make your ansible role drop another file in
``/etc/duplicity/includes_excludes``, you must be certain you understand
what you are doing. You must understand clearly the FILE SELECTION
section of the duplicity man page. Note that the order with which the
files of ``/etc/duplicity/includes_excludes`` will be processed is
unspecified, and you don't know if other inclusion/exclusion options
will be present before or after your file.  You must be reasonably
certain you will not interfere with the file selection of files that
will be processed after your file. You may not attempt to include files
that are excluded by other files (including the base exclusions listed
above), because the result would depend on the order of file processing.

Pre- and post- scripts
======================

The ``/etc/duplicity/pre`` and ``/etc/duplicity/post`` directories
contain scripts that will be executed before and after backup. Each
ansible role that needs this should drop executable files in these
directories. These files are executed in no particular order.
``/etc/duplicity/pre`` is particularly useful for storing database
dumps.
  
Variables
=========

- ``gpg_priv_key``, ``gpg_pub_key``: The GPG keys; they must be the
  full contents of the ASCII armored files (``.asc``).  These keys are
  installed in GPG and marked as trusted.
- ``duplicity_target``: No default.
- ``duplicity_environment``: A dictionary of environment variable names
  mapping to values; this can be useful in order to setup credentials
  when saving to an ftp server (``FTP_PASSWORD``), an Amazon S3 bucket
  (``AWS_ACCESS_KEY_ID``, ``AWS_SECRET_ACCESS_KEY``), or a Google
  Storage bucket (``GS_ACCESS_KEY``, ``GS_SECRET_ACCESS_KEY``).
- ``duplicity_verbosity``: Default 4. See also the
  ``duplicity_remove_all_inc_of_but_n_full`` parameter below.
- ``duplicity_full_if_older_than``: How often to do full backups; see
  the duplicity documentation for more information. Default 3M.
- ``duplicity_remove_older_than``: Oldest backup to keep. See the
  duplicity remove-older-than parameter. Default two years.
- ``duplicity_remove_all_inc_of_but_n_full``: Delete incremental sets
  of all backup sets that are older than the count:th last full
  backup; see the duplicity documentation for more information. Note:
  the related ``duplicity`` command is always run with
  verbosity=error, regardless what has been set in
  ``duplicity_verbosity``. This is because at the usual
  verbosity=warning, on some machines, not understood exactly when,
  the command shows an apparently harmless message "No extraneous
  files found, nothing deleted in cleanup." The default for this
  variable is 2; it can also be set to "" so that no incremental sets
  are removed.
- ``duplicity_extra_parms``: Extra parameters to duplicity.  If you
  backup with ssh and have not added the target host key to
  known_hosts, you might want to use
  ``--ssh-options="-oStrictHostKeyChecking=no"``.
- ``duplicity_when_to_run``: A crontab specification of the time it
  should run; default ``0 4 * * *``, i.e. 4am daily.

.. _duplicity: http://duplicity.nongnu.org/

Meta
====

Written by Antonis Christofides

| Copyright (C) 2011-2015 Antonis Christofides
| Copyright (C) 2013 Ministry of Environment of Greece
| Copyright (C) 2014 National Technical University of Athens

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see http://www.gnu.org/licenses/.
