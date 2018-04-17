# DataStax opscenter with separate cluster for opscenter keyspace

## About

This github covers setting up a separate DataStax cluster to hold the Opscenter keyspace to for the main DataStax cluster.  Two main docker containers are used:   DataStax Server and DataStax opscenter.


The DataStax images are well documented at this github location  [https://github.com/datastax/docker-images/](https://github.com/datastax/docker-images/)


## Getting Started
1. Prepare Docker environment
2. Pull this github into a directory  `https://github.com/jphaugla/datastaxOpscDCDocker.git`
3. Follow notes from DataStax Docker github to pull the needed DataStax images.  Directions are here:  [https://github.com/datastax/docker-images/#datastax-platform-overview](https://github.com/datastax/docker-images/#datastax-platform-overview).  Don't get too bogged down here.  The pull command is provided with this github in pull.sh. It is requried to have the docker login and subscription complete before running the pull.  The also included docker-compose.yaml handles most everything else.
4. Open terminal, then: `docker-compose up -d`
5. Verify DataStax is working for both hosts:
```bash
docker exec dse cqlsh -u cassandra -p cassandra -e "desc keyspaces";
```
```bash
docker exec dseops cqlsh -u cassandra -p cassandra -e "desc keyspaces"
```
6. Also note, in the home directory for the github repository directory the docker volumes should be created as subdirectories.  To manipulate the dse.yaml file, the local conf subdirectory will be used.  The other dse directories are logs, cache and data.  The directory that will change is the opscenter-clusters as the opscenter configuration file will be visible here.

## Set up Opscenter

This Opscenter storage is documented here:  
[https://docs.datastax.com/en/latest-opscenter/opsc/configure/opscStoringCollectionDataDifferentCluster_t.html](https://docs.datastax.com/en/latest-opscenter/opsc/configure/opscStoringCollectionDataDifferentCluster_t.html)

This tutorial provides specific commands for this environment, so it shouldn't be necessary to refer to the docs, but they're a handy reference if something goes awry.  


1. Open up browser to opscenter and add each of the two seed nodes to be managed by Opscenter.  Note:   Each node is assigned to a separate cluster.  The dse node is in the "main" cluster and dseops is in OPSCstore.  The idea is to have the opscenter cluster for the "main" cluster to be stored in the "OPSCstore" cluster.  Use the host names dse and dseops to add the clusters to opscenter.  This link describes how to add each of the 1 node clusters:
[https://docs.datastax.com/en/opscenter/6.1/opsc/online_help/opscAddingCluster_t.html](https://docs.datastax.com/en/opscenter/6.1/opsc/online_help/opscAddingCluster_t.html)
NOTE:  A listing of the opscenter-clusters directory should show the OPSCenter.conf and main.conf files.

2. When the clusters are added to opscenter, a separate file will be created in the opscenter node for each cluster managed by ospcenter.  The following steps will edit the mmain.conf file in opscenter so the "main" cluster will have its opcenter data stored in the "OPSCstore" cluster.  All the seed nodes for "OPSCstore" should be included in the main.diff file.  Verify the file main.diff has dseops as the seed_host. 

4. Get the main.conf file from the opscenter node using docker
```bash
docker cp opsc:/opt/opscenter/conf/clusters/main.conf .
```
5. append the edited main.diff to the main.conf file
```bash
cat main.diff >> main.conf
```
6. replace the main.conf file in the image.  
```bash
docker cp main.conf opsc:/opt/opscenter/conf/clusters
```
7. Restart the ospcenter node using docker.  
```bash
docker restart opsc
```
## Conclusion
At this point, DSE is set up with the opscenter storage for the cluster "main" stored in the cluster "OPSCstore".  To verify, a new keyspace will be created in the opscenter cluster called "OPSCstore" called "OpsCcenter_Storage_main".

##  Additional Notes/tips

* To get the IP address of the DataStax nodes:
```bash
export DSE_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' dse)
```
```bash
export DSE_IP2=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' dseops)
```
```bash
export OPS_IP=$(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' opsc)
```
use `echo $DSE_IP`, `echo $DSE_IP2` and `echo $OPS_IP` to view
