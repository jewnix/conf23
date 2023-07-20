# Corrupted Replica Set (MongoDB)

## Problem:

### Scenarios where MongoDB contains wrong replica set members

In some scenario(mentioned below), SHC KVStore cluster contains incorrect members in SHC replica set.

## Cause:

- Decommissioning of SHC members, after decommissioning of SHC members sometimes replica set does not update with correct information.
- Multiple SHC – accidentally setting static captain from a different SHC will update replica set with incorrect members.

**NOTE:** MongoDB supports up to 50 max members in replica set; it will not allow addition of 51st SHC member to the replica set.

## SPL Query - To find out Primary and Secondary KVStore members
```
index=_introspection sourcetype=kvstore component=KVStoreReplicaSetStats host=<SHC_members>
| spath data.replSetStats.members{}.name output=searchhead 
| spath data.replSetStats.members{}.stateStr output=state 
| table _time, searchhead, state
```

## Solution
To update replica set
- Remove decommissioned SHC members from SHC using `shcluster remove shcluster-member` (if any).
- Find out the PRIMARY member before the problem started OR member with the most up to date DB.
- Note down the GUID of that member from $SPLUNK_HOME/etc/instance.cfg
- On SHC captain run following command to recreate the replica set. `splunk resync kvstore -source <GUID from good member>`  
**NOTE:** This resyncs entire kvstore