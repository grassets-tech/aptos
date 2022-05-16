# Aptos
Aptos validator node setup for Incentivized Testnet

Official documents:
> [Run a Validator Node](https://aptos.dev/tutorials/validator-node/intro)
> [Testnet announcement](https://medium.com/aptoslabs/launch-of-aptos-incentivized-testnet-registration-2e85696a62d0)

## Manual playbook:

### Set up vars
```
echo "export WORKSPACE=testnet" >> $HOME/.bash_profile
echo "export PUBLIC_IP=$(curl -s ifconfig.me)" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Install docker
```
sudo apt-get install ca-certificates curl gnupg lsb-release -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
```

### Install docker compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Install Aptos CLI
```
wget -qO aptos-cli.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-cli-v0.1.1/aptos-cli-0.1.1-Ubuntu-x86_64.zip
sudo unzip -o aptos-cli.zip -d /usr/local/bin
sudo chmod +x /usr/local/bin/aptos
```

### Install Validator node
```
mkdir ~/$WORKSPACE && cd ~/$WORKSPACE
wget -qO docker-compose.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/docker-compose.yaml
wget -qO validator.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/validator.yaml
wget -qO fullnode.yaml https://raw.githubusercontent.com/aptos-labs/aptos-core/main/docker/compose/aptos-node/fullnode.yaml
```
Generate keys

> This will create three files: private-keys.yaml, validator-identity.yaml, validator-full-node-identity.yaml for you.

```
aptos genesis generate-keys --output-dir ~/$WORKSPACE
```

IMPORTANT: Backup your key files somewhere safe. These key files are important for you to establish ownership of your node, and you will use this information to claim your rewards later if eligible. Never share those keys with anyone else.

Configure validator
```
aptos genesis set-validator-configuration \
  --keys-dir ~/$WORKSPACE --local-repository-dir ~/$WORKSPACE \
  --username grassets \
  --validator-host $PUBLIC_IP:6180 \
  --full-node-host $PUBLIC_IP:6182
```
  
Generate root key
```
  aptos key generate --output-file keys/root
```

Create layout file
```
tee layout.yaml > /dev/null <<EOF
---
root_key: "<Key from root.pub >"
users:
  - grassets
chain_id: 23
EOF
```

Download Aptos Framework
```
wget -qO framework.zip https://github.com/aptos-labs/aptos-core/releases/download/aptos-framework-v0.1.0/framework.zip
unzip -o framework.zip
```

Compile genesis blob and waypoint
```
aptos genesis generate-genesis --local-repository-dir ~/$WORKSPACE --output-dir ~/$WORKSPACE
```

Start docker-compose
```
sudo docker-compose up -d
```

Node dashboard and logs
```
https://aptos-node.info/
sudo docker logs -f testnet_fullnode_1
sudo docker logs -f testnet_validator_1
```



