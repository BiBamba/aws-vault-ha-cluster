# aws-vault-ha-cluster

## Creation of Auto-Scaling Groups
1-Create 2 auto-scaling Groups:
- consul-cluster-asg: desired(3), min(1) max(3)
- vault-cluster-asg: desired(3), min(1) max(3)

### Configuration of the clusters

#### Consul Cluster
1- Create this director `/usr/local/etc/consul` and `/usr/local/etc/consul/certs` on each consul server and consul client node 
ssh -i key-pair ec2-user@ip-address to each instance of the consul-cluster-asg

On each instance install consul
`sudo yum install -y yum-utils`

### Add the official HashiCorp Linux repository.
`sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo`

`sudo yum install consul-1.8.3-2.x86_64`

## Prepare the security credentials
#### Generate the gossip encryption key
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
run the foolowing command on the instance where you have created the Certificates and keys.
`scp consul-agent-ca.pem <dc-name>-<server/client>-consul-<cert-number>.pem <dc-name>-<server/client>-consul-<cert-number>-key.pem <USER>@<PUBLIC_IP>:/usr/local/etc/consul/certs/`

### Configure Consul agents

Create a configuration file /usr/local/etc/consul/consul.json for each consul agent:

1-Consul servers
- Consul Servers config file:
`sudo vim /usr/local/etc/consul/consul.json` then paste the content of aws-vault-ha-cluster/consul-cluster/consul_server.json of this repo
`sudo chmod 640 /usr/local/etc/consul/consul.json`

- Consul systemd unit file:
`sudo vim /etc/systemd/system/consul.service` then paste the content of aws-vault-ha-cluster/consul-cluster/consul.service

- Start and verify the Consul cluster state
Be sure that the ownership and permissions are correct on the directory you specified for the value of data_dir, and then start the Consul service on each system and verify the status.
`sudo systemctl start consul`

Check the Consul agent status.
`sudo systemctl status consul`

After starting all Consul server agents, let's check the Consul cluster status:
`consul members`

### Setup Consul client agents on Vault nodes
The Vault servers require both the Consul and Vault binaries on each node. 

On each instance install consul and vault
`sudo yum install -y yum-utils`

Add the official HashiCorp Linux repository.
`sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo`

Install consul
`sudo yum install consul-1.8.3-2.x86_64`

Install vault
`sudo yum install vault`

Consul Client Agent Configuration

`sudo vim /usr/local/etc/consul/consul.json` then paste the content of aws-vault-ha-cluster/vault-cluster/consul-clients-configs/consul_client.json of this repo
`sudo chmod 640 /usr/local/etc/consul/consul.json`

Consul systemd Unit file:
`sudo vim /etc/systemd/system/consul.service` then paste the content of aws-vault-ha-cluster/vault-cluster/consul-clients-configs/consul.service

- Start and verify the Consul cluster state
Be sure that the ownership and permissions are correct on the directory you specified for the value of data_dir, and then start the Consul service on each system and verify the status.
`sudo systemctl start consul`

Check the Consul agent status.
`sudo systemctl status consul`

### Configure the Vault servers

Vault Configuration
On each vault node run:

`sudo mkdir /etc/vault`
`sudo touch /etc/vault/vault.json`
`sudo vim /etc/vault/vault.json` then paste the content of aws-vault-ha-cluster/vault-cluster/vault_server.json of this repo
`sudo chmod 640 /etc/vault/vault.json`

Vault Server systemd Unit file

On each vault node run:
`sudo vim /etc/systemd/system/vault.service` then paste the content of aws-vault-ha-cluster/vault-cluster/vault.service

`systemctl daemon-reload`

`sudo systemctl start vault`
`sudo systemctl status vault`

Initialize Vault
`vault operator init`

On each vault node run:
`vault operator unseal <unseal_key_1>`
`vault operator unseal <unseal_key_2>`
`vault operator unseal <unseal_key_3>`




