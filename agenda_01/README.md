# SHC Architecture

For Search Head Cluster, we'll require minimum 3 nodes (I know you can create SHC with just 1 member only but to function SHC we need mimimum 3 nodes). To access SHC members Web UI, we'll require load balancer in front of all the SHC members to distribute traffic so that users can access SHC members through single interface. SHC is using RAFT consensus algorithm for captain election with the help of KVStore.

More details about SHC architecture is avilable on https://docs.splunk.com/Documentation/Splunk/latest/DistSearch/SHCarchitecture