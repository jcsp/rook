# Enable external Ceph management tools

## TL;DR

Some tools want to use Rook to run containers, but not manage the logical Ceph resources like Pools.  We should make Rook's pool management optional.

## Background

Currently in Rook 0.8, creating and destroying a Filesystem (or ObjectStore)
in a Ceph cluster also creates and destroys the associated Ceph filesystem
and pools.

The current design works well when the Ceph configuration is within the
scope of what Rook can configure itself, and the user does not modify
the Ceph configuration of pools out of band.

## Limitations

The current model is problematic in some cases:

- A user wants to use Ceph functionality outside of Rook's subset, and
  therefore create their pools by hand before asking Rook to run
  the daemon containers for a filesystem.
- A user externally modifies the configuration of a pool (such as the
  number of replicas), they probably want that new configuration, rather than
  for Rook to change it back to match the Rook Filesystem settings.
- A risk-averse user wants to ensure that mistaken edits to their Rook config cannot
  permanently erase Ceph pools (i.e. they want to only delete pools through
  an imperative interface with confirmation prompts etc).

## Proposal

To add two boolean fields to each of FilesystemSpec and ObjectStoreSpec,
controlling the creation and deletion of pools (and also filesystems, in the
case of FilesystemSpec).

- `skipPoolCreation` : do not attempt to create pools/filesystems
- `preservePoolsOnRemove` : do not delete pools on deletion of Rook entity

For convenience, both of these are defined such that their default (false)
value results in the historical behaviour of creating pools and deleting
them automatically.

The existing metadata and data pool fields would become optional, with
validation enforcing their presence only if skipPoolCreation is false.

The pool creation and filesystem creation are controlled by the same
setting because they are both Ceph logical entities (a more literally named
setting like "ControlLogicalObjects" would be needlessly obscure, hence
naming the settings in terms of pools).

## Impact

- Rook Operator: the updates to controller code are small, being simple conditional
blocks around activity related to managing pools.

- Migration: this would have no impact on existing or future users who want
the existing behaviour


