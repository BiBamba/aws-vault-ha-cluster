# aws-vault-ha-cluster

## Creation of Auto-Scaling Groups
1-Create 2 auto-scaling Groups:
- consul-cluster-asg: desired(3), min(1) max(3)
- vault-cluster-asg: desired(3), min(1) max(3)

### Configuration of the clusters

#### Consul Cluster
ssh -i key-pair ec2-user@ip-address to each instance of the consul-cluster-asg

On each instance install consul
`sudo yum install -y yum-utils`

### Add the official HashiCorp Linux repository.
`sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo`
`sudo yum install consul-1.8.3-2.x86_64`

## Prepare the security credentials
####Generate the gossip encryption key
Generate the gossip encrypt key on one of the consul instance. Save that key in a file for later use
`consul keygen`

### Generate TLS certificates for RPC encryption
The following is done on one consul instance later get cpoied to the other instances

##### Create the Certificate Authority
`consul tls ca create`

#### Create the certificates for each server
run the following command three times: `consul tls cert create -server -dc dc1 -domain consul`(for consul servers)

run the following command three times: `consul tls cert create -client -dc dc1 -domain consul`(for consul client)

#### Distribute the certificates to agents
run the foolowing command on the instance 

