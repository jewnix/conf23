# Deployer Slow To Deploy Apps

## Problem:

### Slow SHC App Deployment

When a SHC is trying to deploy a large bundle that has many apps and lookups to many SHC Members, it can take quite a while for it to deploy.

## Cause:

By default, the Deployer only uses a single thread to deploy the bundle to the SHC Members. The way the bundle is created is by it having a separate tarball for each app within the bundle, and it pushes out those bundles to all the members sequentially. When there are many apps inside that bundle, it takes a lot of time to extract each app from that bundle, and it does that for every member.

## Solution

### Parallelizing the Deployment
Setting the `deployerPushThreads` allows you to set the maximum number of threads used on the deployer to push apps and configs to members. If you set it to `auto`, will tell the deployer to auto-tune the number of threads to one thread per member. and it will push those bundles to the members in parallel.

If you set it to a value higher than the number of members, it will only use as many threads as there are members in that SHC.

You can set it to a lower number if the Deployer is not sized to push out to that many members.

### Log output

Example of debug logs when using `deployerPushThreads = auto`. As you can see, in this case we have three SHC Members, and it creates a worker for each of them.

```
06-06-2023 12:48:52.471 +0000 DEBUG ConfReplication [308 AppsDeployDataProviderExecutorWorker-2] - sendDeployableAppsImpl target=https://192.168.48.106:8089, serverName=sh2
06-06-2023 12:48:52.471 +0000 DEBUG ConfReplication [307 AppsDeployDataProviderExecutorWorker-1] - sendDeployableAppsImpl target=https://192.168.54.191:8089, serverName=sh3
06-06-2023 12:48:52.471 +0000 DEBUG ConfReplication [306 AppsDeployDataProviderExecutorWorker-0] - sendDeployableAppsImpl target=https://192.168.109.54:8089, serverName=sh1
06-06-2023 12:48:52.475 +0000 DEBUG ConfReplication [306 AppsDeployDataProviderExecutorWorker-0] - sendDeployableAppsImpl target=https://192.168.109.54:8089 toUpdate=1 toDelete=0
06-06-2023 12:48:52.476 +0000 DEBUG ConfReplication [308 AppsDeployDataProviderExecutorWorker-2] - sendDeployableAppsImpl target=https://192.168.48.106:8089 toUpdate=1 toDelete=0
06-06-2023 12:48:52.476 +0000 DEBUG ConfReplication [307 AppsDeployDataProviderExecutorWorker-1] - sendDeployableAppsImpl target=https://192.168.54.191:8089 toUpdate=1 toDelete=0
```
