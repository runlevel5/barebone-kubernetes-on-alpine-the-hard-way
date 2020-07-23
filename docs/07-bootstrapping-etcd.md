# Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/etcd-io/etcd). In this lab you will bootstrap a three node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites

The commands in this lab must be run on each controller VM: `kcontroller1`, `kcontroller2`, and `kcontroller3`.

## Bootstrapping an etcd Cluster Member

### Download and Install the etcd Binaries

Install etcd and ectdctl:

```
sudo apk add etcd
```

### Configure the etcd Server

```
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```

The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance:

```
INTERNAL_IP=$(hostname -i)
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

```
ETCD_NAME=$(hostname -s)
```

Create a new `/etc/etcd/conf.yml`:

```
sudo rm /etc/etcd/conf.yml
cat <<EOF | sudo tee /etc/etcd/conf.yml
# This is the configuration file for the etcd server.

# Human-readable name for this member.
name: ${ETCD_NAME}

# Path to the data directory.
data-dir: /var/lib/etcd

# Path to the dedicated wal directory.
wal-dir:

# Number of committed transactions to trigger a snapshot to disk.
snapshot-count: 10000

# Time (in milliseconds) of a heartbeat interval.
heartbeat-interval: 100

# Time (in milliseconds) for an election to timeout.
election-timeout: 1000

# Raise alarms when backend size exceeds the given quota. 0 means use the
# default quota.
quota-backend-bytes: 0

# List of comma separated URLs to listen on for peer traffic.
listen-peer-urls: https://${INTERNAL_IP}:2380

# List of comma separated URLs to listen on for client traffic.
listen-client-urls: https://${INTERNAL_IP}:2379,https://127.0.0.1:2379

# Maximum number of snapshot files to retain (0 is unlimited).
max-snapshots: 5

# Maximum number of wal files to retain (0 is unlimited).
max-wals: 5

# Comma-separated white list of origins for CORS (cross-origin resource sharing).
cors:

# List of this member's peer URLs to advertise to the rest of the cluster.       
# The URLs needed to be a comma-separated list.                                  
initial-advertise-peer-urls: https://${INTERNAL_IP}:2380                             
                                                                                 
# List of this member's client URLs to advertise to the public.                  
# The URLs needed to be a comma-separated list.                                  
advertise-client-urls: https://${INTERNAL_IP}:2379                                  
                                                                                 
# Discovery URL used to bootstrap the cluster.                                   
discovery:                                                                       
                                                                                 
# Valid values include 'exit', 'proxy'                                           
discovery-fallback: 'proxy'                                                      
                                                                                 
# HTTP proxy to use for traffic to discovery service.                            
discovery-proxy:                                                                 
                                                                                 
# DNS domain used to bootstrap initial cluster.                                  
discovery-srv:                                                                   
                                                                                 
# Initial cluster configuration for bootstrapping.                               
initial-cluster: "kcontroller1=https://172.42.42.101:2380,kcontroller2=https://172.42.42.102:2380,kcontroller3=https://172.42.42.103:2380"                                                                
                                                                                 
# Initial cluster token for the etcd cluster during bootstrap.                   
initial-cluster-token: 'etcd-cluster-0'                                            
                                                                                 
# Initial cluster state ('new' or 'existing').                                   
initial-cluster-state: 'new'                                                     
                                                                                 
# Reject reconfiguration requests that would cause quorum loss.                  
strict-reconfig-check: false                                                     
                                                                                 
# Accept etcd V2 client requests                                                 
enable-v2: true                                                                  
                                                                                 
# Enable runtime profiling data via HTTP server                                  
enable-pprof: true                                                        
                                                                          
# Valid values include 'on', 'readonly', 'off'                            
proxy: 'off'                                                   
                                                               
# Time (in milliseconds) an endpoint will be held in a failed state.
proxy-failure-wait: 5000                                            
                                                                    
# Time (in milliseconds) of the endpoints refresh interval.         
proxy-refresh-interval: 30000

# Time (in milliseconds) for a dial to timeout.                                  
proxy-dial-timeout: 1000                                                         
                                                                                 
# Time (in milliseconds) for a write to timeout.                                 
proxy-write-timeout: 5000                                                        
                                                                                 
# Time (in milliseconds) for a read to timeout.                                  
proxy-read-timeout: 0                                                            
                                                                                 
client-transport-security:                                                       
  # Path to the client server TLS cert file.                                     
  cert-file: /etc/etcd/kubernetes.pem                                                                    
                                                                                 
  # Path to the client server TLS key file.                                      
  key-file: /etc/etcd/kubernetes-key.pem                                                                     
                                                                                 
  # Enable client cert authentication.                                           
  client-cert-auth: true                                                        
                                                                                 
  # Path to the client server TLS trusted CA cert file.                          
  trusted-ca-file: /etc/etcd/ca.pem                                                              
                                                                                 
  # Client TLS using generated certificates                                      
  auto-tls: false                                                                
                                                                                 
peer-transport-security:                                                         
  # Path to the peer server TLS cert file.                                       
  cert-file: /etc/etcd/kubernetes.pem                                                                    
                                                                                 
  # Path to the peer server TLS key file.                                        
  key-file: /etc/etcd/kubernetes-key.pem                                                                     
                                                                                 
  # Enable peer client cert authentication.                                      
  client-cert-auth: true                                                        
                                                                                 
  # Path to the peer server TLS trusted CA cert file.                     
  trusted-ca-file: /etc/etcd/ca.pem                                                       
                                                                          
  # Peer TLS using generated certificates.                          
  auto-tls: false                                                   
                                                                    
# Enable debug-level logging for etcd.                              
debug: false                                                        
                                                                    
logger: zap 

logger: zap                                                               
                                                                    
# Specify 'stdout' or 'stderr' to skip journald logging even when running under systemd.
log-outputs: [stderr]                                               
                                                                                        
# Force to create a new one member cluster.                         
force-new-cluster: false                                            
                                                                    
auto-compaction-mode: periodic                                      
auto-compaction-retention: "1"

EOF
```

### Start the etcd Server

```
  sudo rc-service etcd start
  sudo rc-update add etcd boot
```

> Remember to run the above commands on each controller node: `kcontroller1`, `kcontroller2`, and `kcontroller3`.

## Verification

List the etcd cluster members:

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

> output

```
3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379, false
f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379, false
ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379, false
```

Next: [Bootstrapping the Kubernetes Control Plane](08-bootstrapping-kubernetes-controllers.md)
