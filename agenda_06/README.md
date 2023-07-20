# Concurrency - Quota Pyramid

### We will not even discuss real-time searches, because they are evil!

## Problem:

We find ourselves many times with users complaining that their searches get queued, or datamodels not populating due to queued searches or concurrency reaching its limits. Hunitng down the culprit can be hard.

## Cause:
### Misconfigured quotas
As most of you know, there are a few types of search quotas: User/Role and Instance/Cluster. When configuring those quotas, you will need to take in to consideration the workload and users behavior.

#### User/Role Quotas:
For the user, there is the `srchJobsQuota` setting, and for the role there is `cumulativeSrchJobsQuota`. To enable role based quota, you will need to set `enable_cumulative_quota = true` in limits.conf. When leaving the `enable_cumulative_quota` with the default setting which is `false`, the `cumulativeSrchJobsQuota` will take precenece over the `srchJobsQuota`, and users within that role won't be able to exceed the commulitive quota.

When enabling cumulative quotas, a user belonging to multiple roles can eat up the slots from every role he is a member of, starting from the one with the highest quota, and working himself downwards.
* **By default, User and role/commulitive quotas, are calculated per instance only**

#### Instance/Cluster Quotas:
Each instance has it's own concurrency limit, which has been around for a while (`max_searches_per_cpu x number_of_cpus + base_max_searches`), 

* Ad-hoc searches always have a higher priority

## Solution:

**All cluster-wide quotas are calculated by the captain!**
* When enabling `enable_cumulative_quota`, take into account the amount of user in that role, and make sure that the `cumulativeSrchJobsQuota` is set according to the average serch per user, and limit the per user quota accordingly.
* Don't set the `srchJobsQuota` so high that a single user can consume all slots of the roles they are a member of.
* Calculate the total capacity of the SHC, and base the user/role quotas on that.
* Set `shc_role_quota_enforcement = true` for the captain to force cluster-wide user/role quota enforcements.
* Set `shc_adhoc_quota_enforcement = overflow` for the captain to allow users to proxy the search to a idle member.
