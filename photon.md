---
title: Photon Platform Integration
---
The docker volume plugin also supports Photon platform. The plugin supports all volume provisioning and managenment operations, defined by the Docker Volume plugin interface, on both platforms (vSphere and Photon). The volumes created in photon platform are done via the open Photon platform API using the Photon controller. 

## Configuration: Photon Driver

The configurations used by a driver while performing operations are read from a JSON file and the default location where it looks for it /etc/docker-volume-vsphere.conf. You can also override it to use  a different configuration file by providing --config option and the full path to the file. Finally the parameters passed on the CLI override the one from the configuration file.

The docker photon volume driver creates and manages the volumes on a photon platform. Photon platform API is used to create and manage volumes. You need following configurations for photon driver in the configuration file.

```
{
    "Driver": "<driver name - vsphere/vmdk/photon>"
    "MaxLogAgeDays": 28,
    "MaxLogSizeMb": 100,
    "LogPath": "/var/log/docker-volume-vsphere.log",
    "LogLevel": "info",
    "Target" : "http://<photon_controller_ip>:<target port>",
    "Project" : "<21-digit photon project ID>",
    "Host" : "<32-digit photon VM ID "
}
```
<table class="table table-striped table-hover ">
  <thead>
    <tr>
      <th>Parameter Name</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>target</td>
      <td>The endpoint of the Photon Controller with hostname and port in format: http://{photon_host}:{photon_port} </td>
    </tr>
    <tr>
      <td>project</td>
      <td>The project ID in Photon to which this host belongs.</td>
    </tr>
    <tr>
      <td>host</td>
      <td>The ID of docker host in Photon.</td>
    </tr>
</tbody>
</table>


<div class="panel panel-info">
  <div class="panel-heading">
    <h3 class="panel-title">Details of Photon Platform</h3>
  </div>
  <div class="panel-body">
     The project and VM IDs mentioned above can be obtained via the “photon” CLI, see https://github.com/vmware/photon-controller/wiki/Compile-Photon-Controller-CLI.
  </div>
</div> 

## Using the Service in Docker

### Creation and management of docker volumes
The docker volume commands are completely supported by vDVS plugin. This section demonstrates use of various commands with examples.


#### Size
You can specify the size of volume while creating a volume. Supported units of sizes are mb, gb and tb. By default if you don’t specify the size, a 100MB volume is created.

```
docker volume create --driver=vsphere --name=MyVolume -o size=10gb
```

#### File System Type (fstype)
You can specify the filesystem which will be used it to create the volumes. The docker plugin will look for existing filesystesm in /sbin/mkfs.fstype but if the specified filesystem is not found then it will return a list for which it has found mkfs. The default filesystem if not specified is ext4.

```
docker volume create --driver=vsphere --name=MyVolume -o size=10gb -o fstype=xfs
docker volume create --driver=vsphere --name=MyVolume -o size=10gb -o fstype=ext4 (default)

```

#### Flavour 
You can specify the flavour of persistent disk available in the Photon controller while creating a volume. The flavour in Photon implies certain limits on the volume being created.
More information about flavours is available [here](https://github.com/vmware/photon-controller/wiki/Flavors).

```
docker volume create --driver=photon --name=CloneVolume -o flavor=<Photon persistent disk flavor name>
```
### Listing Volumes
Docker volume list can be used to volume names & their DRIVER type
```
docker volume ls
DRIVER              VOLUME NAME
vsphere                MyVolume@vsanDatastore
vsphere                minio1@vsanDatastore
vsphere                minio2@vsanDatastore
photon                 redis-data@vsanDatastore
```
### Docker volume inspect
You can use `docker volume inspect` command to see vSphere attributes of a particular volume.
```
docker volume create —driver=vmdk —name=MyVolume -o size=2gb -o vsan-policy-name=myPolicy -o fstype=xfs
```
```
docker volume inspect MyVolume
[
    {
        "Driver": "vmdk",
        "Labels": {},
        "Mountpoint": "/mnt/vmdk/MyVolume",
        "Name": "MyVolume",
        "Options": {
            "fstype": "xfs",
            "size": "2gb",
            "vsan-policy-name": "myPolicy"
        },
        "Scope": "global",
        "Status": {
            "access": "read-write",
            "attach-as": "independent_persistent",
            "capacity": {
                "allocated": "32MB",
                "size": "2GB"
            },
            "clone-from": "None",
            "created": "Wed Mar  1 20:06:02 2017",
            "created by VM": "esx1_swarm01",
            "datastore": "vsanDatastore",
            "diskformat": "thin",
            "fstype": "xfs",
            "status": "detached",
            "vsan-policy-name": "myPolicy"
        }
    }
]
```
Note: For disk formats zeroedthick and thin, the allocated size would be total size plus the size of replicas.


### Remove volume
You can remove the volume with following command
```
# docker volume rm db_data
db_data
```


For more information about Photon plaform, please refer [Wiki page](https://github.com/vmware/photon-controller/wiki).
