# MongoDB 3-Node Replica Set Cluster with Vagrant and Ansible

This project creates a 3-node MongoDB replica set cluster using Vagrant for VM provisioning and Ansible for configuration management.

## Architecture

| Node   | IP Address      | Role      | CPU | Memory |
|--------|-----------------|-----------|-----|--------|
| mongo1 | 192.168.8.21    | Primary   | 2   | 4GB    |
| mongo2 | 192.168.8.22    | Secondary | 2   | 4GB    |
| mongo3 | 192.168.8.23    | Secondary | 2   | 4GB    |

## Prerequisites

- [Vagrant](https://www.vagrantup.com/downloads) (2.2+)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) (6.1+)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) (2.9+)

### Install Prerequisites on Ubuntu/Debian

```bash
# Install VirtualBox
sudo apt update
sudo apt install -y virtualbox

# Install Vagrant
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y vagrant

# Install Ansible
sudo apt install -y ansible
```

## Usage

### Start the Cluster

```bash
# Navigate to the project directory
cd /home/objecthuang/Learn/mongodb

# Start all VMs and provision with Ansible
vagrant up
```

This will:
1. Create 3 Ubuntu 22.04 VMs with VirtualBox
2. Install MongoDB 7.0 on all nodes
3. Configure replica set authentication with a shared keyfile
4. Initialize the replica set with mongo1 as the primary

### Check Cluster Status

```bash
# SSH into the primary node
vagrant ssh mongo1

# Check replica set status
mongosh --eval "rs.status()"

# Check replica set configuration
mongosh --eval "rs.conf()"
```

### Connect to MongoDB

From your host machine (requires MongoDB client):

```bash
# Connect to replica set
mongosh "mongodb://192.168.8.21:27017,192.168.8.22:27017,192.168.8.23:27017/?replicaSet=rs0"
```

From within any VM:

```bash
vagrant ssh mongo1
mongosh
```

### Manage VMs

```bash
# Check status of all VMs
vagrant status

# Stop all VMs (preserves data)
vagrant halt

# Restart all VMs
vagrant up

# Destroy all VMs (removes all data)
vagrant destroy -f

# SSH into a specific node
vagrant ssh mongo1
vagrant ssh mongo2
vagrant ssh mongo3

# Re-run Ansible provisioning
vagrant provision
```

## File Structure

```
.
├── Vagrantfile                 # Vagrant configuration
├── ansible.cfg                 # Ansible configuration
├── README.md                   # This file
├── inventory/
│   └── hosts                   # Ansible inventory
└── playbooks/
    ├── site.yml                # Main Ansible playbook
    └── templates/
        └── mongod.conf.j2      # MongoDB configuration template
```

## Configuration

### MongoDB Settings

You can modify these settings in `playbooks/site.yml`:

- `mongodb_version`: MongoDB version (default: 7.0)
- `mongodb_replica_set_name`: Replica set name (default: rs0)
- `mongodb_port`: MongoDB port (default: 27017)
- `mongodb_bind_ip`: Bind IP address (default: 0.0.0.0)

### VM Resources

Modify in `Vagrantfile`:

- Memory: Change `vb.memory = 4096` (in MB)
- CPUs: Change `vb.cpus = 2`
- IPs: Modify the `NODES` array

## Troubleshooting

### Check MongoDB logs

```bash
vagrant ssh mongo1
sudo tail -f /var/log/mongodb/mongod.log
```

### Check MongoDB service status

```bash
vagrant ssh mongo1
sudo systemctl status mongod
```

### Restart MongoDB service

```bash
vagrant ssh mongo1
sudo systemctl restart mongod
```

### Force re-initialize replica set

```bash
vagrant ssh mongo1
mongosh --eval "rs.reconfig(rs.conf(), {force: true})"
```

## Security Notes

- This setup uses keyfile authentication between replica set members
- The keyfile is automatically generated and distributed to all nodes
- For production, consider:
  - Enabling user authentication
  - Using TLS/SSL encryption
  - Configuring firewall rules
  - Using proper network segmentation
