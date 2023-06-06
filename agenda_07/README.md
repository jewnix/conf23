# Deployer Slow To Deploy Apps


### Useful SPL
#### Events from the SHC Deployer

##### Log Output:
```
{
   component: ConfDeployment
   data: { [-]
     apps: [ [+]
     ]
     spent: 1.3209629999999999
     status: succeeded
     target_label: sh2
     target_uri: https://192.168.48.106:8089
     task: sendDeployableApps
   }
   datetime: 06-05-2023 18:07:50.177 +0000
   log_level: INFO
}
```
##### SPL:
```
index=_internal host=*deployer* sourcetype=splunkd_conf component=ConfDeployment data.task=sendDeployableApps data.status="*" data.target_label="*"
| timechart span=1m count by data.status
```

## Solution

### How bundles are pushed to memebers.
The Deployer creates tarball for each app, it queries `/services/shcluster/member/members` list of members, and pushes the tarballs to each member sequentually. 

### Parallelizing the Deployment
Setting the `deployerPushThreads` allows you to set the maximum number of threads used on the deployer to push apps and configs to members. If you set it to auto, will tell the deployer to auto-tune the number of threads to one thread per member. and it will push those bundles to the members in parallel.

If you set it to a value higher than the number of members, it will only use as many threads as there are members in that SHC.

#### Log output
```
06-06-2023 12:48:52.471 +0000 DEBUG ConfReplication [308 AppsDeployDataProviderExecutorWorker-2] - sendDeployableAppsImpl target=https://192.168.48.106:8089, serverName=sh2
06-06-2023 12:48:52.471 +0000 DEBUG ConfReplication [307 AppsDeployDataProviderExecutorWorker-1] - sendDeployableAppsImpl target=https://192.168.54.191:8089, serverName=sh3
06-06-2023 12:48:52.471 +0000 DEBUG ConfReplication [306 AppsDeployDataProviderExecutorWorker-0] - sendDeployableAppsImpl target=https://192.168.109.54:8089, serverName=sh1
06-06-2023 12:48:52.475 +0000 DEBUG ConfReplication [306 AppsDeployDataProviderExecutorWorker-0] - sendDeployableAppsImpl target=https://192.168.109.54:8089 toUpdate=1 toDelete=0
06-06-2023 12:48:52.476 +0000 DEBUG ConfReplication [308 AppsDeployDataProviderExecutorWorker-2] - sendDeployableAppsImpl target=https://192.168.48.106:8089 toUpdate=1 toDelete=0
06-06-2023 12:48:52.476 +0000 DEBUG ConfReplication [307 AppsDeployDataProviderExecutorWorker-1] - sendDeployableAppsImpl target=https://192.168.54.191:8089 toUpdate=1 toDelete=0
```
