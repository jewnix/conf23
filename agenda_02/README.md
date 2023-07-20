# SHC KVStore Collection

As you already know that Splunk SHC is using RAFT consensus algorithm for captain election but do you know that there are 2 KVStore collections who keep SHC members information and those are used during captain election, to track status of all members and store conf replication bookmarks.

KVStore collections:
- SearchHeadClusterMemberInfo
- SearchHeadClusterHealthStates

When captaincy will transfer, splunk will re-populate member info from RAFT and KVStore collections.

## Collection - SearchHeadClusterMemberInfo
- This KVStore collection keeps information about all SHC members like guid, management port, KVStore port, etc.
- Tracks all members in SHC
- Allow captain to calculate search quota based on available members after captaincy change

When any SHC members stop sending heartbeat to the captain for heartbeat_timeout specified under [shclutering] stanza in server.conf, that SHC member will move from UP to DOWN status. During this time if captaincy will change then new captain will not aware about DOWN member because of missing heartbeat. This kvstore collection will help new captain to know about all SHC members (UP & DOWN members) and it will help facilitate to calculate quota to decide maximum number of allowed searches.

#### KVStore content:
REST Call
```
$SPLUNK_HOME/bin/splunk _internal call "/servicesNS/nobody/system/storage/collections/data/SearchHeadClusterMemberInfo"
```

Content
```
[
    {
        "RollingUpgrade": false,
        "members": [
            {
                "serverName": "sh1",
                "guid": "3C56ADD3-9636-43B8-818D-B5A685746403",
                "raftServerId": "https://sh1:8089",
                "HostAndPort": "10.202.35.126:8089",
                "KVStoreHostAndPort": "sh1:8191",
                "RawPort": 9887,
                "UseSSL": false
            },
            {
                "serverName": "sh2",
                "guid": "A72A2ED4-4A16-4E8D-AF8F-384C6631097E",
                "raftServerId": "https://sh2:8089",
                "HostAndPort": "10.202.33.220:8089",
                "KVStoreHostAndPort": "sh2:8191",
                "RawPort": 9887,
                "UseSSL": false
            },
            {
                "serverName": "sh3",
                "guid": "546D1B73-ED2F-44A5-AD11-77113E2D9620",
                "raftServerId": "https://sh3:8089",
                "HostAndPort": "10.202.38.47:8089",
                "KVStoreHostAndPort": "sh3:8191",
                "RawPort": 9887,
                "UseSSL": false
            }
        ],
        "_user": "nobody",
        "_key": "SHC_UNIQUE_KEY_FOR_MEMBERINFO_V1"
    }
]
```


## Collection - SearchHeadClusterHealthStates
- This KVStore collection stores conf replication bookmark information.
- Persists replication bookmark over captaincy changes

When SHC will change captain from one node to another, new captain will read information about last conf replication from this KVStore collection. When you have `prevent_out_of_sync_captain = true` (default), SHC will not allow out of sync node to become a SHC captain & SHC will determine this detail from this KVStore content. This will prevent duplicate changes when captaincy switched multiple times in short period of span and new captain will aware about last conf replication changes on SHC members.

#### KVStore content:
REST Call
```
$SPLUNK_HOME/bin/splunk _internal call "/servicesNS/nobody/system/storage/collections/data/SearchHeadClusterHealthStates"
```

Content
```
[
    {
        "LastUpdateCaptain": "https://sh1:8089",
        "LastUpdateTime": "Tue Jun 13 15:36:24 2023",
        "ConfRepl_Bookmark": [
            {
                "Member_Uri": "https://sh2:8089",
                "806081c55c3655b37a7270ccdc7b6b690b30de9a": "1686670579"
            },
            {
                "Member_Uri": "https://sh1:8089",
                "DummyOpId": "1686670584"
            },
            {
                "Member_Uri": "https://sh3:8089",
                "806081c55c3655b37a7270ccdc7b6b690b30de9a": "1686670579"
            }
        ],
        "_user": "nobody",
        "_key": "SHC_UNIQUE_KEY_FOR_STATE_V1"
    }
]
```