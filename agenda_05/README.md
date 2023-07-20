# SHC Members Out Of Sync

## Problem-1:
 Large number of CSV Lookup replication, ui-prefs or user-prefs configuration files across SHC members will create problem in SHC and will result in SHC members out of sync.

## Solution
* Reduce the scheduled search interval so it will update CSV lookup less frequently or convert CSV lookups to KVstore lookups.
* Deny the ui-prefs.conf and/or user-prefs from replicating across search head cluster members.

### How to prevent user-prefs from replicating

Add this to `server.conf`
```
[shclustering]
conf_replication_include.ui-prefs = false
conf_replication_include.user-prefs = false
```

## SPL
#### Number of conf replication
```
index=_internal host=<SHC_members> sourcetype=splunkd_conf 
| rename data.* as * 
| eval asset_uri=coalesce('asset_uri{}','dst{}'), ko_type=mvindex(asset_uri,2) 
| timechart span=30s usenull=f count by ko_type limit=20
```
#### Lookup files (csv) replication
```
index=_internal host=<SHC_members> sourcetype=splunkd_conf "data.asset_uri{}"=*lookups*
| rename data.* as * 
| eval asset_uri=coalesce('asset_uri{}','dst{}'), ko_name=mvindex(asset_uri,3) 
| timechart span=30s usenull=f count by ko_name limit=20
```


## Problem-2:
When running a search, splunk will create a `status.csv` file in the `$SPLUNK_HOME/var/run/splunk/dispatch/<sid>/` folders. If search query has high cardinality byfields it may generate large `status.csv` which will cause heartbeat failure between SHC member and SHC Captain, this will result in SHC members out of sync.

## Solution
Write search query effectively so that it will not have large `status.csv` file.

## SPL
#### Heartbeat Failure
```
index=_internal host=<SHC_members> sourcetype=splunkd component=SHCMasterHTTPProxy log_level=ERROR TERM(too) TERM(large)
```

#### Number of heartbeat sent
```
index=_internal host=<SHC_members> sourcetype=splunkd source=*/metrics.log* TERM(group=shclustering) TERM(name=member_heartbeat_payload) 
| timechart span=5m max(num_heartbeats) as max_num_heartbeats by host
```