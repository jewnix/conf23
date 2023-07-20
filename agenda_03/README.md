# Captaincy Switching

## Problem:

### Captaincy switch at every few minutes

In some scenarios (mentioned below), we'll face issue with Captain stability and SHC will trigger transfer of captain at every few minutes from one node to another & this cycle goes on.

## Cause:

- KVStore member stuck in recovering state or kvstore replication lag can trigger captaincy switch every few minutes until KVStore members are synced. When KVStore collection `SearchHeadClusterHealthStates` does not have up-to-date information on one of the member then this will create problem because it has wrong information about previous captain.
- Captaincy transfer is triggered when below conditions occur:
    - prevent_out_of_sync_captain is true – introduced in Splunk Enterprise 6.6; default is true
    - conf replication is older than 180 seconds
    - Last captain transfer greater than 4 x election_timeout_ms

### Scenario & Log output

#### Assumption
In below document when we mention KVStore collection that means KVStore collection `SearchHeadClusterHealthStates`

#### Scenario
In this scenario captaincy transfer occuered at every few minutes between SH1 & SH2 because SH3 contain old SHC captain info in KVStore collection and KVStore status was `Rollback` so it didn't update latest content in KVStore collection.

In below logs problem started on SH1 when it tries to transfer captaincy to SH2. 

 - SH2 became SHC captain and updated info in KVStore collection, at that time SH3 was KVStore captain.
 - SH1 transferred captaincy to SH2 but failed to update KVStore collection, at that time SH3 was KVStore captain.
 - SH2 transferred captaincy to SH1 but failed to update KVStore collection, at that time SH3 was KVStore captain.
 - SH1 (Captain) restarted and transferred captaincy to SH2. Around same time SH3 (KVstore captain) restarted and after restart KVStore on that node went in `Rollback` status (On this node KVStore collection contain SHC captain as SH2) & SH4 became KVStore captain, when SH4 became KVStore primary it got info from SH1 (SHC captain) to update KVStore collection with new SHC captain.


```
05-30-2022 04:48:36.868 +0000 INFO  SHCMaster [69879 CallbackRunnerThread] - Trying to transfer captaincy to node https://sh1:8089, since the node is a better candidate
05-30-2022 04:48:36.881 +0000 INFO  SHCRaftConsensus [31075 TcpChannelThread] - Voting for https://sh1:8089 in term 348
05-30-2022 04:48:36.882 +0000 INFO  SHCRaftConsensus [31075 TcpChannelThread] - All hail leader https://sh1:8089 for term 348
05-30-2022 04:48:46.389 +0000 INFO  SHCRaftConsensus [31075 TcpChannelThread] - New commitIndex: 295
05-30-2022 04:49:08.194 +0000 ERROR KVStorageProvider [69867 SHPHouseKeepingThread] - An error occurred during the last operation ('saveBatchData', domain: '2', code: '4'): Failed to send "update" command with database "s_systemMX8edh8vqo2ngaR2K53MLFyt_SearchsfHc1FLBBz6+EjHq4ZyH+gbn": Failed to read 4 bytes: socket error or timeout
05-30-2022 04:49:08.194 +0000 WARN  SHPConfReplicationBookmarkFromKVStore [69867 SHPHouseKeepingThread] - writeSHCStateToStateStore: failed to write SHC state to KVstore: 
```

## SPL Query - Search to identify captaincy switch & kvstore related errors
```
index=_internal host=<SHC_members> sourcetype=splunkd 
((component=SHCMaster TERM(transfer)) OR 
component=SHCRaftConsensus OR 
(component=KVStorageProvider TERM(SHPHouseKeepingThread)) OR component=SHPConfReplicationBookmarkFromKVStore)
```

## Solution
Re-sync stale KVStore member(s)
