reflink-snap
============

This utility creates snapshots of a source directory using reflinks, and purges
old snapshots. This will only work if the underlying filesystem supports
reflinks, e.g. an XFS filesystem created with `mkfs.xfs -m reflink=1`. It
essentially copies the entire tree over to the snapshot directory location.


Usage
-----

Syntax:

    /opt/reflink-snap/snapshot [ -n ] -s <src_dir> -d <snapshot_dir> -c <count> [ -p <prefix> ]

Options:

    -n                  Dry run, do not perform any actions, just display what
                        it would do
    -s <src_dir>        The source tree to be snapshotted
    -d <snapshot_dir>   The destination directory under which to create a
                        snapshot
    -c <count>          Maximum number of snapshots to retain
    -p <prefix>         Prefix for the snapshot directory name

Notes:

* This will copy `<src_dir>` to `<snapshot_dir>/<prefix><timesamp>`. All files
  will be copied using reflinks to minimize disk space usage.
* When it's done copying it will remove old snapshot directories as necessary in
  order to retain no more than `<count>` snapshots.
* `<snapshot_dir>` must be on the same filesystem as `<src_dir>`.
* It will skip over any files/directories inside `<src_dir>` that are on a
  different filesystem.
* `<prefix>` defaults to `snapshot_`.

Example cronjob:

    0 0 * * * /opt/reflink-snap/snapshot -s / -d /.snapshots -c 7 > /var/log/reflink-snap.log 2>&1

This example will copy and retain up to a week's worth of daily snapshots under
`/.snapshots/snapshot_<snapshot_timestamp>`.


Caveats
-------

If `<snapshot_dir>` is within `<src_dir>` it should be located at the root level
of `<src_dir>`. That way it is excluded when copying `<src_dir>` to
`<snapshot_dir>/.../`. If you don't follow this precaution new snapshots will
end up including a copy of all previous snapshots, and you'll end up with
exponential file/directory growth.

This utility will take a while to run if you have lots of files/directories.

The produced tree may not be consistent if another process writes to the source
tree while the snapshot is in progress. If you're using ZFS or Btrfs you should
use its built-in snapshot functionality instead of this utility, as those
snapshots will be fully consistent and will create much more quickly. However,
this utility is useful if you're running a filesystem that supports reflinks but
not snapshots.
