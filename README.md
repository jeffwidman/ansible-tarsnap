Tarsnap Ansible role
=========

Installs and manages `tarsnap` + `tarsnapper` on CentOS/RHEL 7.

Tarsnapper is a python wrapper for maintaining a GFS backup system.

Also includes logrotate policy.

Role Variables
--------------

The main variable you need to adjust is `tarsnapper_jobs_sourcefile`. This is the
Tarsnapper jobs file, where you specify what you want Tarsnapper to backup.

By default, `tarsnapper_jobs_sourcefile` looks for a file called
`tarsnapper_jobs.yml.j2`. I have included an example in the templates directory
called `tarsnapper_jobs.yml.j2.example`.

The simplest thing to do is copy this file to your playbook directory, add your
jobs, and then delete the '.example' extension.

One gotcha: Ansible first looks for files in the role's template directory and
only if it doesn't find the file will it look in the playbook directory. So if
`tarsnapper_jobs.yml.j2` exists in both `/roles/tarsnap/templates` and in the root
of your playbook directory, the role template will win.

There's a crontask that by default runs once a day to call Tarsnapper. If you want
backups more/less frequently, adjust the crontask accordingly.
If you want to be emailed the output of the job, just set `MAILTO` in your crontab.

Other Notes:
--------------

This role is a little convoluted because there's a cron task that calls a bash
shell script that runs tarsnapper which runs tarsnap.

Why a shell script?
Adds some error handling and logs the output of the tarsnapper jobs to the logfile.

Why Tarsnapper?
Having lots of snapshots doesn't take up much additional space or cost because
tarsnap does an amazing job of diff'ing the backups and only storing the deltas.
Unfortunately, as you accumulate more and more snapshots, restoring a backup
becomes painfully slow because Tarsnap has to parse all those diffs to reconstruct
the backup.
For most production scenarios, a faster recovery time is more important than
maintaining daily backups for years and years. That's where Tarsnapper comes in.
It speeds up your recovery time by deleting intermediate backups, thereby
forcing Tarsnap to merge snapshot deltas together. You can choose which snapshots
get deleted using a configurable grandfather-father-son (GFS) scheme.


Requirements
------------

Tested on CentOS 7.
Untested but will likely work on CentOS 6 and equivalent RHEL versions.

Example Playbook
----------------

    - hosts: servers
      roles:
        - { role: tarsnap, tarsnapper_jobs_sourcefile: path/to/jobs/file, tags: ['tarsnap'] }

License
-------

MIT

Author Information
------------------

Jeff Widman jeff@jeffwidman.com
