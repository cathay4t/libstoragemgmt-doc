---
title: Python API User Guide
---

* [1. Connection -- lsm.Client] (#1.-connection----lsm.client)
* [2. Capability -- lsm.Capabilities]
  (#2.-capability----lsm.capabilities)
* [3. System -- lsm.System] (#3.-system----lsm.system)
* [4. Pool -- lsm.Pool] (#4.-pool----lsm.pool)
* [5. Disk -- lsm.Disk] (#5.-disk----lsm.disk)
* [6. Volume -- lsm.Volume] (#6.-volume----lsm.volume)
* [7. Access Group -- lsm.AccessGroup]
  (#7.-access-group----lsm.accessgroup)
* [8. Volume Mask] (#8.-volume-mask)
* [9. Volume Replication] (#9.-volume-replication)
* [10. iSCSI Authentication] (#10.-iscsi-authentication)
* [Appendix.A Reserved] (#appendix.a-reserved)
* [Appendix.B Reserved] (#appendix.b-reserved)
* [Appendix.C Asynchronous Job Control]
  (#appendix.c-asynchronous-job-control)
* [Appendix.D Bit Map] (#appendix.d-bit-map)
* [Appendix.E lsm Static Methods] (#appendix.e-lsm-static-methods)
* [Appendix.F Exceptions -- lsm.LsmError]
  (#appendix.f-exceptions----lsm.lsmerror)

This document provides detail about how to use LibStorageMgmt python API
for storage system management.

This document is assuming you already read the [LibStorageMgmt User
Guide][1].

Quick sample codes are:

```python
#!/usr/bin/python2
from lsm import Client, LsmError, ErrorNumber

# Make connection.
lsm_client = Client("sim://")

# Enumerate Storage Pools.
try:
    pools = lsm_client.pools()
except LsmError as lsm_err:
    # Handle errors
    if lsm_erro.code == ErrorNumber.NO_SUPPORT:
        print "Not supported"
        exit(1)
    else:
        print "%s" % lsm_err
        exit(1)

for p in pools:
    # Use pool information.
    print 'pool name:', p.name, 'freespace:', p.free_space

if lsm_client is not None:
    lsm_client.close()
    print 'We closed'
```


Quick tips:

* The well known LUN in SAN environment is named as lsm.Volume.
* The group holding multiple FC/FCoE WWPN or iSCSI IQNs is named as
  lsm.AccessGroup.
* The LUN masking is renamed as lsm.Client.volume_mask().

These common exceptions might be raised by every methods:

* `lsm.ErrorNumber.LIB_BUG`

* `lsm.ErrorNumber.PLUGIN_BUG`

* `lsm.ErrorNumber.TIMEOUT`

* `lsm.ErrorNumber.INVALID_ARGUMENT`

* `lsm.ErrorNumber.NETWORK_CONNREFUSED`

* `lsm.ErrorNumber.NETWORK_HOSTDOWN`

* `lsm.ErrorNumber.NETWORK_ERROR`

* `lsm.ErrorNumber.NO_MEMORY`


The method specific expcetions are listed under every methods'
defination.  All exceptions and their explaination are listed at
[Section Appendix.F. LsmError][2].

## 1. Connection -- lsm.Client
* [1.1. Make Connection -- `lsm.Client()`]
  (#1.1.-make-connection----lsm.client())
* [1.2. Close Connection -- `lsm.Client.close()`]
  (#1.2.-close-connection----lsm.client.close())

### 1.1. Make Connection -- `lsm.Client()`


```rst
lsm.Client(uri, password=None, tmo=30000, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Version:
    1.0
Usage:
    Make the connection to LibStorageMgmt plugin.
Parameters:
    uri (string):
        URI string.
    password (string or None):
        Optional. Password string.
        If set to None, it means no password is required.
        Default is None.
    tmo (int):
        Optional. Maximum milliseconds of runtime for each method before
        raising LsmError with ErrorNumber.TIMEOUT.
        Default values is 30000(30 seconds).
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    lsm.Client
        Instance of lsm.Client.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.PLUGIN_AUTH_FAILED
        lsm.ErrorNumber.PLUGIN_IPC_FAIL
        lsm.ErrorNumber.PLUGIN_SOCKET_PERMISSION
        lsm.ErrorNumber.PLUGIN_NOT_EXIST
        lsm.ErrorNumber.DAEMON_NOT_RUNNING
Sample:
    lsm_client = lsm.Client('sim://')
```


### 1.2. Close Connection -- `lsm.Client.close()`


```rst
lsm.Client.close()

Version:
    1.0
Version:
    1.0
Usage:
    Close the connection to LibStorageMgmt plugin.
Parameters:
    N/A
Returns:
    None
SpecialExceptions:
    N/A
Sample:
    lsm_client = lsm.Client('sim://')
    lsm_client.close()
```


## 2. Capability -- lsm.Capabilities

* [2.1. Query Capabilities -- `lsm.Client.capabilities()`]
  (#2.1.-query-capabilities----lsm.client.capabilities())
* [2.2. Check Capability -- lsm.Capabilities.supported()]
  (#2.2.-check-capability----lsm.capabilities.supported())
* [2.3. List Supported Capabilities -- lsm.Capabilities.get_supported()]
  (#2.3.-list-supported-capabilities----lsm.capabilities.get_supported())

LibStorageMgmt is using lsm.Capabilities class to indicate the ability
of current connection. On every connection, these methods are always
functional:

* lsm.Client()
* lsm.Client.close()
* lsm.Client.time_out_get()
* lsm.Client.time_out_set()
* lsm.Client.systems()
* lsm.Client.pools()
* lsm.Client.capabilities()

Other methods has allocated with at least one capability. It was
suggested to check capability before invoking certain methods.

Python sample codes:

```python
#!/usr/bin/python2
import lsm

# Make connection.
lsm_client = lsm.Client("sim://")
# Enumerate all systems
lsm_systems = lsm_client.systems()
# Get capabilities
lsm_caps = lsm_client.capabilities(lsm_systems[0])
# Check certain capability
if lsm_caps.supported(lsm.Capabilities.DISKS):
    lsm_disks = lsm_client.disks()
```


### 2.1. Query Capabilities -- `lsm.Client.capabilities()`


```rst
lsm.Client.capabilities(self, systems, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Version:
    1.0
Usage:
    Query out the capabilities of given sytem.
Parameters:
    system (lsm.System):
        Object of lsm.System to query.
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    lsm.Capabilities
        Object of lsm.Capabilities
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.NOT_FOUND_SYSTEM
Sample:
    lsm_client = lsm.Client('sim://')
    lsm_syss = lsm_client.systems()
    lsm_cap = lsm_client.capabilities(lsm_syss[0])
```


### 2.2. Check Capability -- lsm.Capabilities.supported()

```rst
lsm.Capabilities.supported(self, capability)

Version:
    1.0
Usage:
    Check whether given capacity is supported or not
Parameters:
    capability(int):
        The capability to check. Please refer 'Capability' section of
        each method for valid values.
Returns:
    Bool
        True: Given capability is supported.
        False: Not supported.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.NOT_FOUND_SYSTEM
Sample:
    lsm_client = lsm.Client('sim://')
    lsm_syss = lsm_client.systems()
    lsm_cap = lsm_client.capabilities(lsm_syss[0])
    if lsm_cap.supported(lsm.Capabilities.DISKS):
        lsm_disks = lsm_cli.disks()
    else:
        print "Disks is not supported by system %s" % lsm_syss[0].name
```


### 2.3. List Supported Capabilities -- lsm.Capabilities.get_supported()

```rst
lsm.Capabilities.get_supported(self, all_cap=False)

Version:
    1.0
Usage:
    Return a hash with all storage system supported capabilities:
    Returned hash example:
        {
            lsm.Capabilities.DISKS: 'DISKS',
        }
    Hence you can use this method to convert capability integer to
    human readable string.
    If you intend to get all capabilities in LSM library, please set
    'all_cap=True'.
Parameters:
    all_cap(bool):
        If set to True, will return all capabilities defined by LSM
        library.
Returns:
    {
            lsm.Capabilities.DISKS: 'DISKS',
    }
SpecialExceptions:
    N/A
Sample:
    lsm_client = lsm.Client('sim://')
    lsm_syss = lsm_client.systems()
    lsm_cap = lsm_client.capabilities(lsm_syss[0])
    cap_hash = lsm_cap.get_supported()
    print "Support capabilites are:"
    for cap_str in cap_hash.values():
        print cap_str
```


## 3. System -- lsm.System
* [3.1. System Properties] (#3.1.-system-properties)
* [3.2. Query systems -- `lsm.Client.systems()`]
  (#3.2.-query-systems----lsm.client.systems())

Represents a Storage Array or direct attached storage RAID. Examples
include:

* A hardware RAID card: LSI MegaRAID, HP SmartArray.
* A storage area network (SAN): EMC VNX, NetApp Filer
* A software solution running on commodity hardware: Linux targetd,
  Nexenta

### 3.1. System Properties
* `id`


    String. Free form string used to identify certain system at plugin
    level.

* `name`


    String. Human friendly name for this system.

* `status`


    Integer. Bit Map(Check Appendix.D for detail).
    The hardware health status of system. Could be any combination of
    these values:

    * `lsm.System.STATUS_UNKNOWN`


        Plugin failed to determine the status.

    * `lsm.System.STATUS_OK`


        Everything is OK.

    * `lsm.System.STATUS_ERROR`


        An error has occurred causing the system to stop.
        Example:

        * A whole disk chass down.
        * All controllers down.
        * Internal hardware(like, memory) down and no redundant part.

    * `lsm.System.STATUS_DEGRADED`


        System is still functional but lose protection of redundant
        parts. Example:

        * One or more controller offline, but existing controller is
          taking over all works.

        * One or more battery changed from online to offline.

        * One disk chasis bus is down, but another one is working.


    * `lsm.System.STATUS_PREDICTIVE_FAILURE`


        System is still functional and protected by redundant parts, but
        certain parts will soon be unfunctional. Examples:

        * One or more battery voltage low.

        * SMART information indicate some disk is dieing.

    * `lsm.System.STATUS_OTHER`

        Vendor specifice status. The 'status_info' property will explain
        the detail.

* `status_info`


    String. Free form string used for explaining system status.

    For example:
    "Disk <disk_id> is in Offline state. Battery X is nearly end of
    life"

### 3.2. Query systems -- `lsm.Client.systems()`


```rst
lsm.Client.systems(self, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Return a list of objects of lsm.System.
Parameters:
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    [lsm.System,]
        List of lsm.System instants. Empty list if no system found.
SpecialExceptions:
    N/A
Sample:
    lsm_client = lsm.Client('sim://')
    systems = lsm_client.systems()
```


## 4. Pool -- lsm.Pool
* [4.1. Pool Properties] (#4.1.-pool-properties)
* [4.2. Pool Extra Constants] (#4.2.-pool-extra-constants)
* [4.3. Query Pools -- `lsm.Client.pools()`]
  (#4.3.-query-pools----lsm.client.pools())

Pool is the only place a volume or a file system could created from.

### 4.1. Pool Properties
* `id`


    String. Free form string used to identify certain pool at plugin
    level.

* `name`


    String. Human friendly name for this pool.

* `total_space`


    Long Integer. Pool total space in bytes.

* `free_space`


    Long Integer. Pool free space in bytes.

* `status`


    Integer. Bit Map(Check Appendix.D for detail). The health status of
    pool.  The pool status will not contain volume status which means
    pool status will not change for the deletion or creation of any
    volume.  So generally, the status of pool is representing the status
    of all volumes. If pool is STATUS_ERROR, then all its volumes is
    STATUS_ERROR.

    For example: Some thin provisioning pool consumed all space of one
    pool which caused write request to volumes be rejected. Currently,
    pool status should be STATUS_OK with no free space, all impacted
    volume will has STATUS_ERROR status, other fully provisioning volume
    will still in STATUS_OK.

    Could be any combination of these values:

    * `lsm.Pool.STATUS_UNKNOWN`


        Plugin failed to query out the status of Pool.

    * `lsm.Pool.STATUS_OK`


        Pool is accessible with no issue.

    * `lsm.Pool.STATUS_OTHER`


        Vendor specifice status. The 'status_info' property will explain
        the detail.

    * `lsm.Pool.STATUS_DEGRADED`


        Pool data is accessible but lost full RAID protection due to
        I/O error or offline of one or more RAID member.
        Example:

        * RAID 6 pool lost access to 1 disk or 2 disks.
        * RAID 5 pool lost access to 1 disk.

    * `lsm.Pool.STATUS_ERROR`


        Pool data is not accessible due to some members offline.
        Example:

        * RAID 5 pool lost access to 2 disks.
        * RAID 0 pool lost access to 1 disks.

    * `lsm.Pool.STATUS_STARTING`


        Pool is reviving from STOPPED status. Pool is not accessible
        yet.

    * `lsm.Pool.STATUS_STOPPING`


        Pool is stopping by administrator. Pool is not accessible.

    * `lsm.Pool.STATUS_STOPPED`


        Pool is stopped by administrator. Pool is not accessible.

    * `lsm.Pool.STATUS_RECONSTRUCTING`


        Pool is reconstructing the hash data or mirror data.  Mostly
        happen when disk revive from offline or disk replaced.
        Pool.status_info may contain progress of this reconstruction
        job.

    * `lsm.Pool.STATUS_VERIFYING`


        Array is running integrity check on data of current pool.  It
        might be started by administrator or array itself.  The I/O
        performance will be impacted.  Pool.status_info may contain
        progress of this verification job.

    * `lsm.Pool.STATUS_GROWING`


        Pool is growing its size and doing internal jobs.
        Pool.status_info can contain progress of this growing job.

    * `lsm.Pool.STATUS_DELETING`


        Array is deleting current pool.

* `status_info`


    String. Plugin can use this property to store detailed error or
    warning information. For unsupported storage system, it can leaved
    as empty even when error occurred.

* `element_type`


    Integer. Bit Map(Check Appendix.D for detail). Define what kind of
    element could current pool create. Could be any combination of these
    values:

    * `lsm.Pool.ELEMENT_TYPE_UNKNOWN`


        Plugin failed to detemine the element_type of current pool.

    * `lsm.Pool.ELEMENT_TYPE_POOL`


        New sub-pool could be allocated from current pool.

    * `lsm.Pool.ELEMENT_TYPE_VOLUME`


        New volume could be created from current pool.

    * `lsm.Pool.ELEMENT_TYPE_FS`


        New file system could be created from current pool.

    * `lsm.Pool.ELEMENT_TYPE_DELTA`


        The pool which is holding the dalta of snapshot or replication.

    * `lsm.Pool.ELEMENT_TYPE_SYS_RESERVED`


        This pool is reserved for system internal use.

* `unsupported_actions`


    Integer. Bit Map(Check Appendix.D for detail). Indicate which
    action(s) are not supported against current pool. If set to 0, it
    means all actions indicated by lsm.Client.capabilities() is
    supported against current pool.  Otherwise, could be any combination
    of these values:

    * `lsm.Pool.UNSUPPORTED_VOLUME_GROW`


        Growing volume size via `lsm.Client.volume()` is not supported
        against current pool.

    * `lsm.Pool.UNSUPPORTED_VOLUME_SHRINK`


        Shrinking volume size via `lsm.Client.volume()` is not supported
        against current pool.

* `system_id`


    String. The ID of system which current pool belong to.

### 4.2. Pool Extra Constants

* `lsm.Pool.SUPPORTED_SEARCH_KEYS`


    A list of string. Containing all supported search key used by
    lsm.Client.pools() method.

* `lsm.Pool.TOTAL_SPACE_NOT_FOUND`


    Integer: -1. Indicate plugin failed to retrived total space of the
    pool.

* `lsm.Pool.FREE_SPACE_NOT_FOUND`


    Integer: -1. Indicate plugin failed to retrived free space of the
    pool.

### 4.3. Query Pools -- `lsm.Client.pools()`


```rst
lsm.Client.pools(self, search_key=None, search_value=None,
                 flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Return a list of pools.
    If 'search_key' and 'search_value' is defined, only pool matches
    will be contained in return list.
    Valid search keys are stored in lsm.Pool.SUPPORTED_SEARCH_KEYS:
        id          # Search lsm.Pool.id property
        system_id   # Search lsm.Pool.system_id property
    When no matches found, return a empty list
Parameters:
    search_key (string)
        Optional. The key name for the search.
    search_value (string)
        Optional. The value of search_key to match.
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    [lsm.Pool,]
        A list of lsm.Pool objects.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.UNSUPPORTED_SEARCH_KEY
Sample:
    lsm_client = lsm.Client('sim://')
    pools = lsm_client.pools()
Capability:
    No capability needed for this method.
    Every plugin support lsm.Client.pools() call.
```


## 5. Disk -- lsm.Disk
* [5.1. Disk Properties] (#5.1.-disk-properties)
* [5.2. Disk Extra Constants] (#5.2.-disk-extra-constants)
* [5.3. Query Disks -- `lsm.Client.disks()`] (
  #5.3.-query-disks----lsm.client.disks())

Disk is used to assemble storage pool or as a spare disk.

### 5.1. Disk Properties
* `id`


    String. Free form string used to identify certain disk at plugin
    level.

* `name`


    String. Human friendly name for this disk. Administrator can use
    this string to locate this disk. Example: 'Disk 0_2_22'

* `disk_type`


    Integer. Representing the disk type. Could be one of these values:

    * `lsm.Disk.TYPE_UNKNOWN` -- Plugin failed to determine the disk
      type.
    * `lsm.Disk.TYPE_OTHER` -- Vendor specific disk.
    * `lsm.Disk.TYPE_LUN` -- Remote LUN/Volume be treated as local
      disks.
    * `lsm.Disk.TYPE_ATA` -- IDE/ATA disk.
    * `lsm.Disk.TYPE_SATA` -- SATA disk.
    * `lsm.Disk.TYPE_SAS` -- SAS disk.
    * `lsm.Disk.TYPE_FC` -- FC disk.
    * `lsm.Disk.TYPE_SOP` --  SCSI over PCI-E for Solid State Storage.
    * `lsm.Disk.TYPE_SCSI` -- SCSI disk.
    * `lsm.Disk.TYPE_NL_SAS` -- NL_SAS disk (SATA disk using SAS
      interface).
    * `lsm.Disk.TYPE_HDD` -- Failback value for hard disk
      drive(rotational).
    * `lsm.Disk.TYPE_SSD` -- SSD disk.
    * `lsm.Disk.TYPE_HYBRID` -- Combination of HDD and SSD.

* `block_size`


    Integer. The size of single disk block(sector). Examples: 512, 520
    or 4096.

* `num_of_blocks`


    Long Integer. The could of disk blocks(sectors).

* `size_bytes`


    Long Integer. Holding the disk raw size in bytes. It is calculated
    by (lsm.Disk.block_size * lsm.Disk.num_of_blocks).

* `status`


    Integer. Representing the status of disk. Could be one of these
    values:

    * `lsm.Disk.STATUS_UNKNOWN`


        Plugin failed to determine the status.

    * `lsm.Disk.STATUS_OK`


        Everything is OK.

    * `lsm.Disk.STATUS_OTHER`


        Vendor specific status.

    * `lsm.Disk.STATUS_ERROR`


        Error make disk not functional.

    * `lsm.Disk.STATUS_PREDICTIVE_FAILURE`


        Disk is still functional but will fail soon.

    * `lsm.Disk.STATUS_STARTING`


        Disk is starting up.

    * `lsm.Disk.STATUS_STOPPING`


        Disk is shuting down.

    * `lsm.Disk.STATUS_STOPPED`


        Disk is stoped by administrator.

    * `lsm.Disk.STATUS_INITIALIZING`


        Disk is not functional and is initialiaing.
        It could be:

        * Initialiaing new disk.
        * Zeroing disk.
        * Scrubing disk data.

* `system_id`


    Sting. The ID of lsm.System current disk belong to.

### 5.2. Disk Extra Constants

* `lsm.Disk.BLOCK_COUNT_NOT_FOUND`


    Integer: -1. Indicate plugin failed to detect the block count.

* `lsm.Disk.BLOCK_SIZE_NOT_FOUND`


    Integer: -1. Indicate plugin failed to detect the block size.

* `lsm.Disk.SUPPORTED_SEARCH_KEYS`


    A list of string. Containing all supported search key used by
    lsm.Client.disks() method.

### 5.3. Query Disks -- `lsm.Client.disks()`


```rst
lsm.Client.disks(self,search_key=None, search_value=None,
                 flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Return a list of objects of lsm.Disk.
    If 'search_key' and 'search_value' is defined, only disk matches
    will be contained in return list. Valid search keys are stored in
    lsm.Disk.SUPPORTED_SEARCH_KEYS:
        id          # Search lsm.Disk.id property
        system_id   # Search lsm.System.system_id property
    When no matches found, return a empty list
Parameters:
    search_key (string)
        Optional. The key name for the search.
    search_value (string)
        Optional. The value of search_key to match.
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    [lsm.Disk,]
        A list of lsm.Disk objects.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.PLUGIN_ERROR
            # Plugin error when query. Please report a bug.
        lsm.ErrorNumber.UNSUPPORTED_SEARCH_KEY
            # Got invalid search key
Sample:
    lsm_client = lsm.Client('sim://')
    disks = lsm_client.disks()
Capability:
    lsm.Capabilities.DISKS
        Indicate current system support disk query via
        lsm.Client.disks() method.
    lsm.Capabilities.DISKS_QUICK_SEARCH
        This capability indicate plugin can perform a quick
        search without pull all data from storage system.
        Without this capability, plugin will pull all data from
        storage system to search given key. It only save socket data
        transfer between plugin and user API. If user want to
        do multiple search, it is suggested to query all disks
        without 'search_key' parameter, then filter in user's code
        space.
```



## 6. Volume -- lsm.Volume
* [6.1. Volume Properties] (#6.1.-volume-properties)
* [6.2. Volume Extra Constants] (#6.2.-volume-extra-constants)
* [6.3. Query Volume -- `lsm.Client.volumes()`]
  (#6.3.-query-volume----lsm.client.volumes())
* [6.6. Create Volume -- `lsm.Client.volume_create()`]
  (#6.6.-create-volume----lsm.client.volume_create())
* [6.7. Delete Volume -- `lsm.Client.volume_delete()`]
  (#6.7.-delete-volume----lsm.client.volume_delete())
* [6.8. Resize Volume -- `lsm.Client.volume_resize()`]
  (#6.8.-resize-volume----lsm.client.volume_resize())
* [6.9. Enable Volume -- `lsm.Client.volume_enable()`]
  (#6.9.-enable-volume----lsm.client.volume_enable())
* [6.10. Disable Volume -- `lsm.Client.volume_disable()`]
  (#6.10.-disable-volume----lsm.client.volume_disable())

Volume is well-known as LUN by many storage array vendors. It is a
virtual storage space which host operation system treated as a or many
block device(s).

### 6.1. Volume Properties

* `id`


    String. Free form string used to identify certain volume at plugin
    level.

* `name`


    String. Human friendly name for this volume.

* `vpd83`


    String. Fromat: '[0-9a-f]{32}', SCSI VPD0x83 as identifier of
    volume. Also knowned as 'wwid' in device-mapper-multipath.

* `block_size`


    Integer. The block(also known as sector) size of the volume.

* `num_of_blocks`


    Long integer. The count of usable volume's blocks(also known as
    sector).

* `admin_state`


    Integer. Indicate whether this volume is disabled by administrator
    or not.
    Could be one of these values:
    * `lsm.Volume.ADMIN_STATE_DISABLED`


        The volume is explicitly disabled by administrator or via method
        `lsm.Client.vovlume_disable()`. All block access will be
        rejected.

    * `lsm.Volume.ADMIN_STATE_ENABLED`


        The volume is not explicitly disabled by administrator. This is
        the normal state of a volume.

* `pool_id`


    String. The lsm.Pool.id current volume belong to.

* `system_id`


    String. The lsm.System.id current volume belong to.

* `size_bytes`


    Long integer. Usable size of volume. Calculated by
    (lsm.Pool.block_size * lsm.Pool.num_of_blocks)

### 6.2. Volume Extra Constants

* `lsm.Volume.PROVISION_THIN`


    Used by lsm.Client.volume_create() in 'provisioning' parameter.
    Indicating plugin or storage array should create a thin provisioning
    volume.

* `lsm.Volume.PROVISION_FULL`


    Used by lsm.Client.volume_create() in 'provisioning' parameter.
    Indicating plugin or storage array should create a fully allocated
    volume.

* `lsm.Volume.PROVISION_UNKNOWN`


    Used by lsm.Client.volume_create() in 'provisioning' parameter.
    Indicating leting plugin or storage to decide on creating a thin or
    full volume.

* `lsm.Volume.SUPPORTED_SEARCH_KEYS`


    A list of string. Containing all supported search key used by
    lsm.Client.volumes() method.

### 6.3. Query Volume -- `lsm.Client.volumes()`


```rst
lsm.Client.volumes(self, search_key=None, search_value=None,
                   flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Return a list of lsm.Volume objects.
    If 'search_key' and 'search_value' is defined, only volume matches
    will be contained in return list.
    Valid search keys are stored in lsm.Volume.SUPPORTED_SEARCH_KEYS:
        id          # Search lsm.Volume.id property
        system_id   # Search volumes for given system only.
        pool_id     # Search volumes for given pool only.
    This method does not check whether provided resources ID is valid
    or not, if resource ID is not exist at all, the search result will
    be a empty list.
    When no matches found, return a empty list
Parameters:
    search_key (string)
        Optional. The key name for the search.
    search_value (string)
        Optional. The value of search_key to match.
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    [lsm.Volume,]
        A list of lsm.Volume objects.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.UNSUPPORTED_SEARCH_KEY
Sample:
    lsm_client = lsm.Client('sim://')
    volumes = lsm_client.volumes()
Capability:
    lsm.Capabilities.VOLUMES
        Indicate current system support volumes query via
        lsm.Client.volumes() method.
    lsm.Capabilities.VOLUMES_QUICK_SEARCH
        This capability indicate plugin can perform a quick
        search without pull all data from storage system.
        Without this capability, plugin will pull all volumes from
        storage system to search given key. It only save socket data
        transfer between plugin and user API. If user want to
        do multiple search, it is suggested to query all volumes
        without 'search_key' parameter, then filter in user's code
        space.
```


### 6.6. Create Volume -- `lsm.Client.volume_create()`


```rst
lsm.Client.volume_create(self, pool, volume_name, size_bytes,
                         provisioning, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Create a new volume. Real volume size will be larger or equal to
    requested size.
Parameters:
    pool (lsm.Pool)
        The object of lsm.Pool where new volume should allocated from.
    volume_name (string)
        The human readable name for new volume.
        It might be ignored or altered to suit storage array
        requirements.
    size_bytes (long int)
        Volume size in bytes.
    provisioning (int)
        Indicate the data provisioning type of new volume.
        Could be one of these values:
        * lsm.Volume.PROVISION_UNKNOWN
            Plugin or storage array choose their default setting for
            data
            provisioning(Thin or Full).
        * lsm.Volume.PROVISION_THIN
            Create new thin provisioning volume.
        * lsm.Volume.PROVISION_FULL
            Create new fully allocated volume.
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    (JobID, None)       # Check Appendix.C for detail.
        or
    (None, lsm.Volume)  # The object of lsm.Volume
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.NOT_FOUND_POOL
        lsm.ErrorNumber.NAME_CONFLICT
        lsm.ErrorNumber.NOT_ENOUGH_SPACE
        lsm.ErrorNumber.SYS_LIMIT_MAX_SIZE_EXCEEDED
        lsm.ErrorNumber.SYS_LIMIT_MAX_COUNT_EXCEEDED
        lsm.ErrorNumber.POOL_NOT_READY
Sample:
    lsm_client = lsm.Client('sim://')
    lsm_pool = lsm_client.pools()[0]
    size_bytes = lsm.size_human_2_size_bytes("200GiB")
    (job_id, new_lsm_vol) = lsm_client.volume_create(
        lsm_pool, "test_volue", size_bytes)
    if job_id:
        # we get async job, wait untile finished
        # Check Appendix.C
    print "New volume created:%s(%s)" % (new_lsm_vol.name,
                                         new_lsm_vol.id)
Capability:
    lsm.Capabilities.VOLUME_CREATE
        # Indicate current system support volumes creation via
        # lsm.Client.volume_create() method.
    lsm.Capabilities.VOLUME_CREATE_THIN
        # Can create thin provisioning volume
    lsm.Capabilities.VOLUME_CREATE_FULL
        # Can create full allocated volume
```


### 6.7. Delete Volume -- `lsm.Client.volume_delete()`


```rst
lsm.Client.volume_delete(self, volume, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Delete a volume and return the allocated storage space back to
    storage pool.
    Only all of these requirements has been meet, a volume can be
    deleted,
        * Volume does not mask to any access group.
          If so, volume_unmask() should be issued before
          volume_delete()
          This method can be used to find out the masked access
          groups:
            lsm.Client.access_groups_granted_to_volume(self, volume)
    User can check LsmError if volume_delete() failed for dependency.
Parameters:
    volume (lsm.Volume)
        Object of lsm.Volume to delete.
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    JobID           # Check Appendix.C for detail.
        or
    None            # The deletion finished without error.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.NOT_FOUND_VOLUME
        lsm.ErrorNumber.POOL_NOT_READY
        lsm.ErrorNumber.IS_MASKED
Sample:
    lsm_client = lsm.Client('sim://')
    to_delete_vol = lsm_client.volumes()[0]
    job_id = lsm_client.volume_delete(to_delete_vol)
    if job_id:
        # we get async job, wait untile finished
        # Check Appendix.C
    print "Volume %s(%s) deleted" % (
        to_delete_vol.name, to_delete_vol.id)
Capability:
    lsm.Capabilities.VOLUME_DELETE
        # Indicate current system support volumes deletion via
        # lsm.Client.volume_delete() method.
```


### 6.8. Resize Volume -- `lsm.Client.volume_resize()`


```rst
lsm.Client.volume_resize(self, volume, new_size_bytes,
                         flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Resize (grow or shrink) a volume. Please be warned when shrinking the
    size of volume, there might be a data corruption or data lose.
    It's stongly suggested to perform a data backup before shrinking any
    volume.
    Size grow is mandatory if lsm.Capabilities.VOLUME_RESIZE is supported.
    The resized volume will hold larger size than requested due to
    boundary issue.
Parameters:
    volume          # Object of lsm.Volume to resize.
    new_size_bytes  # Long integer. The new size of volume in bytes.
    flags           # Optional. Reserved for future use, should be 0.
Returns:
    (JobID, None)   # Check Appendix.C for detail.
        or
    (None, lsm.Volume) # Updated lsm.Volume object.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.NOT_ENOUGH_SPACE
        lsm.ErrorNumber.POOL_NOT_READY
        lsm.ErrorNumber.SYS_LIMIT_MAX_SIZE_EXCEEDED
        lsm.ErrorNumber.NO_STATE_CHANGE
            # Requested size is the current volume size.
        lsm.ErrorNumber.NOT_ENOUGH_SPACE
            # Pool does not have enough space.
Sample:
    lsm_client = lsm.Client('sim://')
    to_resize_vol = lsm_client.volumes()[0]
    grow_diff_size_bytes = lsm.size_human_2_size_bytes("2GiB")
    new_size_bytes = to_resize_vol.size_bytes + grow_diff_size_bytes
    (job_id, new_lsm_vol) = lsm_client.volume_resize(
        to_resize_vol, new_size_bytes)
    if job_id:
        # we get async job, wait untile finished
        # Check Appendix.C
    print "Volume %s resized to %d(%s)" % (
        new_lsm_vol.name, new_lsm_vol.size_bytes,
        lsm.size_bytes_2_size_human(new_lsm_vol.size_bytes))
        Capability:
    lsm.Capabilities.VOLUME_RESIZE
        # Indicate current system support volumes grow size via
        # lsm.Client.volume_resize() method.
    lsm.Capabilities.VOLUME_RESIZE_SHRINK
        # Indicate current system support volumes shrink size via
        # lsm.Client.volume_resize() method.
```


### 6.9. Enable Volume -- `lsm.Client.volume_enable()`


```rst
    lsm.Client.volume_enable(self, volume, flags=lsm.Client.FLAG_RSVD)
Version:
    1.0

    Usage:
Enable volume for data access.
    Parameters:
volume          # Object of lsm.Volume to enable.
flags           # Optional. Reserved for future use, should be 0.
    Returns:
None            # volume is enabled now.
    SpecialExceptions:
LsmError
    lsm.ErrorNumber.NOT_FOUND_POOL
    lsm.ErrorNumber.POOL_NOT_READY
    lsm.ErrorNumber.NO_STATE_CHANGE
    Sample:
lsm_client = lsm.Client('sim://')
lsm_vol = lsm_client.volumes()[0]
job_id = lsm_client.volume_disable(lsm_vol)
if job_id:
    # we get async job, wait untile finished
    # Check Appendix.C
print "Volume %s is offline" % lsm_vol.name

job_id = lsm_client.volume_enable(lsm_vol)
if job_id:
    # we get async job, wait untile finished
    # Check Appendix.C
print "Volume %s is enabled" % lsm_vol.name
    Capability:
lsm.Capabilities.VOLUME_ENABLE
    # Indicate current system support enable volume via
    # lsm.Client.volume_enable() method.
```


### 6.10. Disable Volume -- `lsm.Client.volume_disable()`


```rst
    lsm.Client.volume_disable(self, volume, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Disable a volume for data access. This will change
    Volume.admin_state to lsm.Volume.ADMIN_STATE_DISABLED
Parameters:
    volume          # Object of lsm.Volume to disable.
    flags           # Optional. Reserved for future use, should be 0.
Returns:
    None            # volume is offline now.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.NOT_FOUND_POOL
        lsm.ErrorNumber.POOL_NOT_READY
        lsm.ErrorNumber.NO_STATE_CHANGE
Sample:
    lsm_client = lsm.Client('sim://')
    lsm_vol = lsm_client.volumes()[0]
    job_id = lsm_client.volume_disable(lsm_vol)
    if job_id:
        # we get async job, wait untile finished
        # Check Appendix.C
    print "Volume %s is disabled" % lsm_vol.name

    job_id = lsm_client.volume_enable(lsm_vol)
    if job_id:
        # we get async job, wait untile finished
        # Check Appendix.C
    print "Volume %s is enabled" % lsm_vol.name
        Capability:
    lsm.Capabilities.VOLUME_DISABLE
        # Indicate current system support enable volume via
        # lsm.Client.volume_enable() method.
```


## 7. Access Group -- lsm.AccessGroup
* [7.1. Access Group Properties] (#7.1.-access-group-properties)
* [7.2. Access Group Extra Constants]
  (#7.2.-access-group-extra-constants)
* [7.3. Query Access Group -- `lsm.Client.access_groups()`]
  (#7.3.-query-access-group----lsm.client.access_groups())
* [7.4. Create Access Group -- `lsm.Client.access_group_create()`]
  (#7.4.-create-access-group----lsm.client.access_group_create())
* [7.5. Delete Access Group -- `lsm.Client.access_group_delete()`]
  (#7.5.-delete-access-group----lsm.client.access_group_delete())
* [7.6. Add Access Group Member --
  `lsm.Client.access_group_initiator_add()`]
  (#7.6.-add-access-group-member----lsm.client.access_group_initiator_add())
* [7.7. Delete Access Group Member --
  `lsm.Client.access_group_initiator_delete()`]
  (#7.7.-delete-access-group-member----lsm.client.access_group_initiator_delete())

Access group define a group of initiators to access volume. For example,
most FC/FCoE HBA cards has two ports, each of two ports has one(or more with
NPIV) WWPN(s). When performing volume masking, a volume is often mask to a
access group containing all of these WWPNs. On EMC devices, access group is
referred as "Host Group". On NetApp ONTAP devices, access group is referred as
"Initiator Group(igroup)".

For those plugin does not support access group terminology, each of initiator
will be treated as a access group.

### 7.1. Access Group Properties

* `id`

    String. Free form string used to identify certain access group at plugin
    level. Plugin can use this property for performance improvement when
    concerting between LSM object to internal object. When displaying this
    property to user, use the ID hashed string(like md5) is suggested.

* `name`

    String. Human friendly name for this access group.

* `init_ids`

    List of string. A list of initiator ID. Example:
    ['20:00:00:1b:21:67:65:d7', '20:00:00:1b:21:67:65:d6']
    Since WWPN or WWNN can expressed in many ways, init_ids only using this
    format for WWPN:
(?:[0-9a-f]{2}:){7}[0-9a-f]{2}
# Example: 20:00:00:1b:21:67:65:d7
* `init_type`

    Integer. Indicate the type of initiator. Could be one of these values:
    * `lsm.AccessGroup.INIT_TYPE_UNKNOWN`

      Plugin failed to determine the status.
    * `lsm.AccessGroup.INIT_TYPE_OTHER`

      Vendor specific initiator type.
    * `lsm.AccessGroup.INIT_TYPE_WWPN`

      WWPN, used in FC or FCoE HBA.
    * `lsm.AccessGroup.INIT_TYPE_ISCSI_IQN`

      iSCSI IQN.
    * `lsm.AccessGroup.INIT_TYPE_ISCSI_WWPN_MIXED`

      A access group containing both iSCSI IQN and WWPN.
* `system_id`

    The id of lsm.System this access group belong to.

### 7.2. Access Group Extra Constants

* `lsm.AccessGroup.SUPPORTED_SEARCH_KEYS`

    The list of string indicating the supported 'search_key' used by
    lsm.Client.access_groups() method.

* `lsm.AccessGroup.USER_GENERATED_FAKE_AG_ID`

    The string indicate current lsm.AccessGroup object is user generated,
    only used by lsm.Client.volume_mask() when VOLUME_MASK_NEW_AG is supported.

### 7.3. Query Access Group -- `lsm.Client.access_groups()`


```rst
```rst
lsm.Client.access_groups(
    self, search_key=None, search_value=None, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Return a list of objects of lsm.AccessGroup.
    If 'search_key' and 'search_value' is defined, only access group
    matches will be contained in return list.
    Valid search keys are stored in
    lsm.AccessGroup.SUPPORTED_SEARCH_KEYS:
        id              # Search lsm.AccessGroup.id property
        name            # Search lsm.AccessGroup.name property
        init_id         # If given init_id is contained in
                        # lsm.AccessGroup.init_ids
        volume_id       # If access group is masked to given volume.
        system_id       # Return access group for given system only.
    This method does not check whether provided resources ID is valid
    or not, if resource ID is not exist at all, the search result will
    be a empty list.
    When no matches found, return a empty list
Parameters:
    search_key (string)
        Optional. The key name for the search.
    search_value (string)
        Optional. The value of search_key to match.
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    [lsm.AccessGroup,]      # A list of lsm.AccessGroup objects.
        or
    []              # Nothing found.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.PLUGIN_ERROR
            # Plugin error when query. Please report a bug.
        lsm.ErrorNumber.UNSUPPORTED_SEARCH_KEY
            # Got unsupported search key. Should be one of
            # lsm.AccessGroup.SUPPORTED_SEARCH_KEYS
Sample:
    lsm_client = lsm.Client('sim://')
    access_groups = lsm_client.access_groups(
        search_key=None, search_value=None, flags=lsm.Client.FLAG_RSVD)
Capability:
    lsm.Capabilities.ACCESS_GROUPS
        # Indicate current system support access group query via
        # lsm.Client.access_groups() method.
    lsm.Capabilities.ACCESS_GROUPS_QUICK_SEARCH
        # This capability indicate plugin can perform a quick
        # search without pull all data from storage system.
        # Without this capability, plugin will pull all data from
        # storage system to search given key. It only save socket data
        # transfer between plugin and user API. If user want to
        # do multiple search, it is suggested to query all access
        # groups without 'search_key' parameter, then filter in user's
        # code space.
```


### 7.4. Create Access Group -- `lsm.Client.access_group_create()`


```rst
lsm.Client.access_group_create(
    self, access_group_name, init_id, init_type, system_ids, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Create a access group.
Parameters:
    access_group_name   # String. The name for new access group.
                        # If provided access_group_name does not meet
                        # the requirement of storage system, storage
                        # system will chose a valid one in stead of
                        # raise a error.
    init_id             # String. The first initiator ID.
    init_type           # Integer. The type of initiator, could be one
                        # of these values:
                        #  * lsm.AccessGroup.INIT_TYPE_WWPN
                        #  * lsm.AccessGroup.INIT_TYPE_ISCSI_IQN
    system_id           # The id of lsm.System where this access group
                        # should create on.
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    lsm.AccessGroup     # Object of lsm.AccessGroup
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.INVALID_ARGUMENT
            # Not correct init_type or invalid initiator string.
        lsm.ErrorNumber.SYS_LIMIT_MAX_COUNT_EXCEEDED
            # Storage system refuse to create new access group due to
            # internal limitations. For example:
            #  * System only allow 100 access groups, when created new
                 access group after that, this error will be used.
        lsm.ErrorNumber.NAME_CONFLICT
            # Requested access group name is already been used by
            # exist access group.
        lsm.ErrorNumber.EXISTS_INITIATOR
            # Request initiator ID is already been used by other
            # access group.
Sample:
    lsm_client = lsm.Client('sim://')
    new_ag = lsm_client.access_group_create(
        "Test access group",
        '20:00:00:1b:21:67:65:d7',
        lsm.AccessGroup.INIT_TYPE_WWPN)
Capability:
    lsm.Capabilities.ACCESS_GROUP_CREATE
        # Indicate current system support create access group via
        # lsm.Client.access_group_create() method.
    lsm.Capabilities.ACCESS_GROUP_CREATE_WWPN
        # Support creating WWPN access group.
    lsm.Capabilities.ACCESS_GROUP_CREATE_ISCSI_IQN
        # Support creating iSCSI IQN access group.
```



### 7.5. Delete Access Group -- `lsm.Client.access_group_delete()`


```rst
lsm.Client.access_group_delete(self, access_group, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Delete a access group. Only access group has no volume masked to
    can be deleted.
    User can use this method to query volumes masked to give access
    group:
        lsm.Client.volumes(self, search_key='access_group_id',
                           search_value=access_group.id)
Parameters:
    access_group    # Object of lsm.AccessGroup to delete.
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    None            # Access group deleted without errors.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.NOT_FOUND_ACCESS_GROUP
        lsm.ErrorNumber.IS_MASKED
Sample:
    lsm_client = lsm.Client('sim://')
    new_ag = lsm_client.access_group_create(
        "Test access group",
        '20:00:00:1b:21:67:65:d7',
        lsm.AccessGroup.INIT_TYPE_WWPN)
    lsm_client.access_group_delete(new_ag)
Capability:
    lsm.Capabilities.ACCESS_GROUP_DELETE
        # Indicate current system support delete access group via
        # lsm.Client.access_group_delete() method.
```


### 7.6. Add Access Group Member -- `lsm.Client.access_group_initiator_add()`


```rst
lsm.Client.access_group_initiator_add(
    self, access_group, init_id, init_type, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Add defined initiator to certain access group.
Parameters:
    access_group    # Object of lsm.AccessGroup to update.
    init_id         # String. Initiator ID to add.
    init_type       # Integer. The type of initiator, could be one
                    # of these values:
                    #  * lsm.AccessGroup.INIT_TYPE_WWPN
                    #  * lsm.AccessGroup.INIT_TYPE_ISCSI_IQN
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    None            # Access group updated without errors.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.INVALID_ARGUMENT
        lsm.ErrorNumber.NOT_FOUND_ACCESS_GROUP
        lsm.ErrorNumber.NO_STATE_CHANGE
            # Requested initiator already exists in defined access
            # group.
        lsm.ErrorNumber.EXISTS_INITIATOR
            # Requested initiator ID is already used by other
            # access group.
Sample:
    lsm_client = lsm.Client('sim://')
    new_ag = lsm_client.access_group_create(
        "Test access group",
        '20:00:00:1b:21:67:65:d7',
        lsm.AccessGroup.INIT_TYPE_WWPN)
    # Add one WWPN.
    lsm_client.access_group_add_initiator(
        new_ag, ['20:00:00:1b:21:67:65:d9'])
Capability:
    lsm.Capabilities.ACCESS_GROUP_INITIATOR_ADD_WWPN
        # Indicate current system support adding access group WWPN
        # member via lsm.Client.access_group_add_initiator() method.
    lsm.Capabilities.ACCESS_GROUP_INITIATOR_ADD_ISCSI_IQN
        # Indicate current system support adding access group iSCSI
        # IQN member via lsm.Client.access_group_add_initiator()
        # method.
    lsm.Capabilities.ACCESS_GROUP_INITIATOR_ADD_MIX
        # Indicate current system support mixing iSCSI IQN and WWPN
        # in one group via lsm.Client.access_group_add_initiator()
        # method.
```


### 7.7. Delete Access Group Member -- `lsm.Client.access_group_initiator_delete()`


```rst
lsm.Client.access_group_initiator_delete(
    self, access_group, init_id, init_type, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Delete defined initiator from certain access group.
Parameters:
    access_group    # Object of lsm.AccessGroup to update.
    init_id         # String. Initiator ID to remove.
    init_type       # Integer. The type of initiator, could be one
                    # of these values:
                    #  * lsm.AccessGroup.INIT_TYPE_WWPN
                    #  * lsm.AccessGroup.INIT_TYPE_ISCSI_IQN
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    None            # Access group edited without errors.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.NOT_FOUND_ACCESS_GROUP
        lsm.ErrorNumber.NO_STATE_CHANGE
            # Requested initiator does not exist in defined access
            # group.
        lsm.ErrorNumber.LAST_INIT_IN_ACCESS_GROUP
            # Defined access group is masked to volumes and
            # we are removeing its last initiator. This is not
            # allowed in LSM. Try to unmask volumes first.
Sample:
    lsm_client = lsm.Client('sim://')
    new_ag = lsm_client.access_group_create(
        "Test access group",
        '20:00:00:1b:21:67:65:d7',
        lsm.AccessGroup.INIT_TYPE_WWPN)
    # Delete one WWPN.
    lsm_client.access_group_del_initiator(
        new_ag, ['20:00:00:1b:21:67:65:d7'])
Capability:
    lsm.Capabilities.ACCESS_GROUP_DEL_INIT
        # Indicate current system support deleting access group member
        # via lsm.Client.access_group_del_initiator() method.
```


## 8. Volume Mask
* [8.1. Query Masked Access Group -- `lsm.Client.access_groups_granted_to_volume()`] (#8.1.-query-masked-access-group----lsm.client.access_groups_granted_to_volume())
* [8.2. Query Masked Volume -- `lsm.Client.access_groups_granted_to_volume()`] (#8.2.-query-masked-volume----lsm.client.access_groups_granted_to_volume())
* [8.3. Volume Mask -- `lsm.Client.volume_mask()`] (#8.3.-volume-mask----lsm.client.volume_mask())
* [8.4. Volume Unmask -- `lsm.Client.volume_unmask()`] (#8.4.-volume-unmask----lsm.client.volume_unmask())

Volume mask is also known as "LUN Masking". The lsm.MaskInfo represent a
masking relationship between lsm.Volume and lsm.AccessGroup.

Masking information can be queried via these ways:

    * Need lsm.Volume:
```rst
lsm.Client.volumes(
    self, search_key='access_group_id',
    search_value=lsm.AccessGroup.id)
    * Need lsm.AccessGroup:
```rst
lsm.Client.access_groups(
    self, search_key='volume_id', search_value=lsm.Volume.id)
    * Need MaskInfo:
```rst
lsm.Client.mask_infos(
    self, search_key='volume_id', search_value=lsm.Volume.id)
```rst
lsm.Client.mask_infos(
    self, search_key='access_group_id',
    search_value=lsm.AccessGroup.id)

### 8.1. Query Masked Access Group -- `lsm.Client.access_groups_granted_to_volume()`


### 8.2. Query Masked Volume -- `lsm.Client.access_groups_granted_to_volume()`


### 8.3. Volume Mask -- `lsm.Client.volume_mask()`


```rst
lsm.Client.volume_mask(self, access_group, volume, flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Mask certain volume to  defined access group.
    If lsm.Capabilities.VOLUME_MASK_NEW_AG is supported,
    user can pass a fake AccessGroup object to request creating
    access group when masking volume. Example:

        access_group = AccessGroup(
            AccessGroup.USER_GENERATED_FAKE_AG_ID,
            '', list_of_initiator_ids, init_type, system_id)
        lsm_client_obj.volume_mask(volume, access_group)

    The support of lsm.Capabilities.VOLUME_MASK_NEW_AG is only used
    by plugin which does not support access group create but want to
    do volume masking against new initiator.
Parameters:
    access_group    # Object of lsm.AccessGroup
    volume          # Object of lsm.Volume
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    None
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.NOT_FOUND_VOLUME
            # Defined lsm.Volume does not exists.
        lsm.ErrorNumber.NOT_FOUND_ACCESS_GROUP
            # Defined lsm.AccessGroup does not exists.
        lsm.ErrorNumber.EMPTY_ACCESS_GROUP
            # Defined lsm.AccessGroup is empty. LSM does not allow
            # masking volume to empty access group.
        lsm.ErrorNumber.NO_STATE_CHANGE
            # Already masked.
Sample:
    lsm_client = lsm.Client('sim://')
    new_ag = lsm_client.access_group_create(
        "Test access group",
        '20:00:00:1b:21:67:65:d7',
        lsm.AccessGroup.INIT_TYPE_WWPN)
    lsm_pool = lsm_client.pools()[0]
    size_bytes = lsm.size_human_2_size_bytes("200GiB")
    (job_id, new_lsm_vol) = lsm_client.volume_create(
        lsm_pool, "test_volue", size_bytes)
    new_vol = lsm_client.volume_create(
        lsm_pool, "test_volue", size_bytes)
    if job_id:
        # we get async job, wait untile finished
        # Check Appendix.C
    print "New volume created:%s(%s)" % (new_lsm_vol.name,
                                         new_lsm_vol.id)
    lsm_client.volume_mask(new_ag, new_vol)
Capability:
    lsm.Capabilities.VOLUME_MASK
        # Indicate current system support volume masking via
        # lsm.Client.volume_mask() method.
    lsm.Capabilities.VOLUME_MASK_NEW_AG
        # Indicate current system support creating new access group
        # when masking volume.
```


### 8.4. Volume Unmask -- `lsm.Client.volume_unmask()`


```rst
lsm.Client.volume_unmask(self, access_group, volume flags=lsm.Client.FLAG_RSVD)

Version:
    1.0
Usage:
    Unmask the volume.
Parameters:
    access_group    # Object of lsm.AccessGroup
    volume          # Object of lsm.Volume
    flags (int)
        Optional. Reserved for future use.
        Should be set as lsm.Client.FLAG_RSVD.
Returns:
    None            # Unmask done without error.
SpecialExceptions:
    LsmError
        lsm.ErrorNumber.NOT_FOUND_VOLUME
            # Defined lsm.Volume does not exist.
        lsm.ErrorNumber.NOT_FOUND_ACCESS_GROUP
            # Defined lsm.AccessGroup does not exist.
        lsm.ErrorNumber.NO_STATE_CHANGE
            # Already unmasked.
Sample:
    lsm_client = lsm.Client('sim://')
    lsm_vol = lsm_client.volumes()[0]
    lsm_ag = lsm_client.access_groups_granted_to_volume(lsm_vol)[0]
    lsm_client.volume_unmask(lsm_ag, lsm_vol)
Capability:
    lsm.Capabilities.VOLUME_UNMASK
        # Indicate current system support volume unmasking via
        # lsm.Client.volume_unmask() method.
```


## 9. Volume Replication
Volume replication is used for creating duplicate data on another volume.
Every lsm.VolumeReplication object representing a replication relationship of
source volume and target volume.
Each lsm.VolumeReplication object only contain one source and one target
volume. For those array supporting multiple target volumes in one replication
pair(like EMC DMX/VMAX SRDF), they will be split into multiple replication
pairs.

There is no delete method of lsm.VolumeReplication, you can use one of these
instead:

 1. Use lsm.Client.volume_replication_split() to pause the copy or
       synchronization.
 2. Delete the target volume.

## 10. iSCSI Authentication

## Appendix.A Reserved

## Appendix.B Reserved

## Appendix.C Asynchronous Job Control
* [Appendix.C.1 Query a Job -- `lsm.Client.job_status()`] (#appendix.c.1-query-a-job----lsm.client.job_status())
* [Appendix.C.2 Sample Code to Wait a Job] (#appendix.c.2-sample-code-to-wait-a-job)

For time consuming methods like volume_create(), volume_delte() and etc, some
plugin support asynchronous job control which return a job ID for user to
trace the progress of time consuming tasks. Please check method defination
for detail.

### Appendix.C.1 Query a Job -- `lsm.Client.job_status()`


    lsm.Client.job_status(self, job_id)

    Usage:
Query the status of a asynchronous job.
If a job failed, A lsm.LsmError be raise with lsm.ErrorNumber and
error message.
    Parameters:
job_id      # String. The Job ID provided by methods.
    Returns:
(job_status, job_progress, data)
            # job_status:   Integer. Could be one of these values:
            #                * lsm.JobStatus.INPROGRESS
            #                  The job is still running.
            #                * lsm.JobStatus.COMPLETE
            #                  The job finished without error.
            # job_progress: Integer. Percent number of progress.
            # data:         Object of lsm.Volume or lsm.Pool or etc.
            #               Please refer to job-creating method for
            #               returned data type.
    SpecialExceptions:
N/A
    Sample:
lsm_client = lsm.Client('sim://')
lsm_pool = lsm_client.pools()[0]
size_bytes = lsm.size_human_2_size_bytes("200GiB")
(job_id, new_lsm_vol) = lsm_client.volume_create(
    lsm_pool, "test_volue", size_bytes, lsm.Volume.PROVISION_UNKNOWN)
if job_id:
    while True:
        (status, percent, new_lsm_vol) = lsm_client.job_status(job_id)
        if status == JobStatus.INPROGRESS:
            time.sleep(0.1)
            continue
        elif status == JobStatus.COMPLETE:
            print("New volume %s created" % new_lsm_vol.id)
            break
        else:
            raise Exception("Unkonwn job status: %d" % status)

### Appendix.C.2 Sample Code to Wait a Job

    @staticmethod
    def wait_job(lsm_client, job_id, item):
if job_id is None:
    retrun None
else:
    while True:
        (status, percent, item) = self.c.job_status(job)
        if status == JobStatus.INPROGRESS:
            time.sleep(0.1)
            continue
        elif status == JobStatus.COMPLETE:
            return item
        else:
            raise Exception("Unkonwn job status: %d" % status)


## Appendix.D Bit Map
We use "Bit Map" to hold multiple status of resources in one integer.
Sample codes:

    #!/usr/bin/python2
    import lsm

    lsm_client = lsm.Client("sim://")      # Make connection.
    lsm_systems = lsm_client.systems()     # Enumerate all systems
    for lsm_system in lsm_systems:
if lsm_system.status & lsm.System.STATUS_OK:
    print "System %s is in OK status" % lsm_system.name

## Appendix.E lsm Static Methods
* [Appendix.E.1 `lsm.size_bytes_2_size_human()`] (#appendix.e.1-lsm.size_bytes_2_size_human())
* [Appendix.E.2 `lsm.size_human_2_size_bytes()`] (#appendix.e.2-lsm.size_human_2_size_bytes())

### Appendix.E.1 `lsm.size_bytes_2_size_human()`


    lsm.size_bytes_2_size_human(size_bytes)

    Usage:
Convert integer(size in bytes) to human readable size string.
Output string will follow the rule of IEC binary prefixes on size:
    http://en.wikipedia.org/wiki/Gibibyte
Output string format:
    "%.2f %s" % (float_number, unit)
Unit conversion will be:
    2 ** 10             KiB
    2 ** 20             MiB
    2 ** 30             GiB
    2 ** 40             TiB
    2 ** 50             PiB
    2 ** 60             EiB
    Parameters:
size_bytes      # Long integer, count of bytes
    Returns:
size_human      # String. "%.2f %s", Example: "2.51 GiB"
    SpecialExceptions:
N/A
    Sample:
#!/usr/bin/python2
from lsm import size_bytes_2_size_human

print size_bytes_2_size_human(2 * 2 ** 30)
# Get "2.00 GiB"

### Appendix.E.2 `lsm.size_human_2_size_bytes()`


    lsm.size_human_2_size_bytes(size_human)

    Usage:
Convert human readable size string to long integer.
Supported input string format:
    '1.9KiB'            # long(1024*1.9)
    '1 KiB'             # long(2**10)
    '1B'                # long(1)
    '2K'                # treated as '2KiB'
    '2k'                # treated as '2KiB'
    '2KB'               # long(2* 10**3)
Unit conversion will be:
    2 ** 10             KiB     |   10 ** 3         KB
    2 ** 20             MiB     |   10 ** 6         MB
    2 ** 30             GiB     |   10 ** 9         GB
    2 ** 40             TiB     |   10 ** 12        TB
    2 ** 50             PiB     |   10 ** 15        PB
    2 ** 60             EiB     |   10 ** 18        EB
    Parameters:
size_human      # String.
    Returns:
size_bytes      # Long integer, count of bytes
    SpecialExceptions:
N/A
    Sample:
#!/usr/bin/python2
from lsm import size_human_2_size_bytes

print size_human_2_size_bytes("2.0GiB")
# long(2 * 2 ** 10)

## Appendix.F Exceptions -- `lsm.LsmError`


* `lsm.ErrorNumber.LIB_BUG`

    There is a bug of LSM library been triggered. Please report this bug to
    upstream or package maintainer.

* `lsm.ErrorNumber.PLUGIN_BUG`

    There is a bug of LSM plugin been triggered. Please report this bug to
    upstream or package maintainer.

* `lsm.ErrorNumber.JOB_STARTED`

    This is only used in C API to indicate a ASYNC job was created.

* `lsm.ErrorNumber.TIMEOUT`

    Indicate a timeout of requested method.

* `lsm.ErrorNumber.DAEMON_NOT_RUNNING`

    The **lsmd** daemon is not running. Please start it with:

systemctl start lsmd
# or
service lsmd start

* `lsm.ErrorNumber.NAME_CONFLICT`

    Requested name is already used by other volume/fs/access group.

* `lsm.ErrorNumber.EXISTS_INITIATOR`

    Defined initiator already exist in other access group when creating
    access group or adding initiator into access group.

* `lsm.ErrorNumber.INVALID_ARGUMENT`

    The method got invalid data from argument.

* `lsm.ErrorNumber.NO_STATE_CHANGE`

    Indicate requested no state change for requested action. It offten caused
    by repeated call.

* `lsm.ErrorNumber.NETWORK_CONNREFUSED`

    Plugin failed to connect to storage array or SDK as network connection
    refused. Often caused by firewall.

* `lsm.ErrorNumber.NETWORK_HOSTDOWN`

    Plugin failed to connect to storage array or SDK as target host is
    unreachable. Often caused by storage array down or SDK host down.

* `lsm.ErrorNumber.NETWORK_ERROR`

    Plugin failed to connect to storage array or SDK as facing general
    network error.

* `lsm.ErrorNumber.NO_MEMORY`

    Plugin failed to allocate enough memery.

* `lsm.ErrorNumber.NO_SUPPORT`

    Not supported by plugin or target storage system.

* `lsm.ErrorNumber.IS_MASKED`

    Has volume masked to access group which prevent requested action.

* `lsm.ErrorNumber.NOT_FOUND_ACCESS_GROUP`

    Provided object of lsm.AccessGroup in argument of current method does not
    exist on storage system.

* `lsm.ErrorNumber.NOT_FOUND_FS`

    Provided object of lsm.FileSystem in argument of current method does not
    exist on storage system.

* `lsm.ErrorNumber.NOT_FOUND_JOB`

    Provided job_id in argument of current method does not exist on storage
    system.

* `lsm.ErrorNumber.NOT_FOUND_POOL`

    Provided object of lsm.Pool in argument of current method does not
    exists.

* `lsm.ErrorNumber.NOT_FOUND_FS_SS`

    Provided object of lsm.FsSnapshot in argument of current method does not
    exists.

* `lsm.ErrorNumber.NOT_FOUND_VOLUME`

    Provided object of lsm.Volume in argument of current method does not
    exists.

* `lsm.ErrorNumber.NOT_FOUND_NFS_EXPORT`

    Provided object of lsm.NfsExport in argument of current method does not
    exists.

* `lsm.ErrorNumber.NOT_FOUND_SYSTEM`

    Provided object of lsm.System in argument of current method does not
    exist.

* `lsm.ErrorNumber.NOT_LICENSED`

    Storage Array or SDK is not licensed to fullfil request action.

* `lsm.ErrorNumber.NO_SUPPORT_ONLINE_CHANGE`

    Requested action require target object been marked as offline.

* `lsm.ErrorNumber.NO_SUPPORT_OFFLINE_CHANGE`

    Requested action require target object been marked as online.

* `lsm.ErrorNumber.PLUGIN_AUTH_FAILED`

    The plugin failed to authentication with storage system.

* `lsm.ErrorNumber.PLUGIN_IPC_FAIL`

    The plugin cannot establish a IPC to client via socket.

* `lsm.ErrorNumber.PLUGIN_SOCKET_PERMISSION`

    The plugin does not have sufficient permission to run or create a socket.

* `lsm.ErrorNumber.PLUGIN_NOT_EXIST`

    The requested plugin does not exist. It might happen when requested plugin
    got deleted and newly added after lsmd started. Suggest user to check
    URI or restart lsmd.

* `lsm.ErrorNumber.NOT_ENOUGH_SPACE`

    No enough free space in pool to fullfil request actions.

* `lsm.ErrorNumber.TRANSPORT_COMMUNICATION`

    A failure occured when communicate with LSM plugin.

* `lsm.ErrorNumber.TRANSPORT_SERIALIZATION`

    Failed to convert data recived from LSM plugin into LSM objects.
    It's a LSM library bug.

* `lsm.ErrorNumber.TRANSPORT_INVALID_ARG`

    Invalid argument used in IPC communication between client and plugin.
    It's a LSM library bug.

* `lsm.ErrorNumber.LAST_INIT_IN_ACCESS_GROUP`

    Refuse to remove the last initiator from access group.

* `lsm.ErrorNumber.UNSUPPORTED_SEARCH_KEY`

    Indicate 'search_key' argument is not supported by certain query methods.

* `lsm.ErrorNumber.EMPTY_ACCESS_GROUP`

    Refused to do volume_mask() against empty(no initiator) access group.

* `lsm.ErrorNumber.POOL_NOT_READY`

    Requested pool is not ready, refuse to do creation, resize, deletion and
    etc.

[1]: user_guide.html
[2]: #appendix.f-exceptions----lsm.lsmerror