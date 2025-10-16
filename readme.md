<picture>
  <img alt="Rocket Pool - Decentralised Ethereum Staking Protocol" src="https://raw.githubusercontent.com/rocket-pool/.github/main/assets/logo.svg" width="auto" height="120">
</picture>

# Private Testnet

---

## Prerequisites

Git/Docker/Jq (skip if already installed)
```bash
sudo apt update && sudo apt install -y git curl jq ca-certificates apt-transport-https software-properties-common && \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null && \
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker $USER && newgrp docker
sudo reboot
```

Install Kurtosis CLI
```bash
echo "deb [trusted=yes] https://apt.fury.io/kurtosis-tech/ /" | sudo tee /etc/apt/sources.list.d/kurtosis.list
sudo apt update
sudo apt install kurtosis-cli
```

---


## Bootnode

Get the artifacts from the bootnode
```bash
mkdir -p ./artifacts/el_cl_genesis_data
kurtosis files download rocketpool el_cl_genesis_data ./artifacts/el_cl_genesis_data
```

---

## Join your node

Start up compose
```bash
cd node
cp env.example .env
docker compose up --force-recreate -d
```

Check your Genesis tx against the bootnode (should be as below)
```bash
curl -s -X POST http://localhost:8545 \
      -H 'content-type: application/json' \
      --data '{"jsonrpc":"2.0","id":1,"method":"eth_getBlockByNumber","params":["0x0",false]}' \
      | jq -r '.result.hash'
0x53baf2a08b050f2eea446792d9f94de592023177ccb0881e14ac8a42c2553e83
```
---

## Helpful commands

Get the ENR from the Bootnode
```bash
curl -s http://127.0.0.1:33001/eth/v1/node/identity | jq -r '.data.enr'
```

Get the Enode value from the Bootnode
```bash
kurtosis service exec rocketpool el-1-geth-lighthouse "geth --exec admin.nodeInfo.enode attach ipc:/data/geth/execution-data/geth.ipc"
```

Generate JWT
```bash
mkdir -p node/jwt
openssl rand -hex 32 > node/jwt/jwtsecret
chmod 600 node/jwt/jwtsecret
```

Get the Genesis TX from the Bootnode
```bash
kurtosis service exec rocketpool el-1-geth-lighthouse "geth --exec 'eth.getBlock(0).hash' attach ipc:/data/geth/execution-data/geth.ipc"
```

Check the Genesis TX of your node against the Bootnode
```bash
curl -s -X POST http://localhost:8545 \
-H 'content-type: application/json' \
--data '{"jsonrpc":"2.0","id":1,"method":"eth_getBlockByNumber","params":["0x0",false]}' \
| jq -r '.result.hash'
0x53baf2a08b050f2eea446792d9f94de592023177ccb0881e14ac8a42c2553e83
```

Bootnode UFW settings
```bash
sudo ufw allow 32000/tcp comment 'RP Testnet: EL Peer TCP'
sudo ufw allow 32000/udp comment 'RP Testnet: EL Peer UDP'
sudo ufw allow 32001/tcp comment 'RP Testnet: EL Engine API (JWT)'
sudo ufw allow 32003/tcp comment 'RP Testnet: EL JSON-RPC HTTP'
sudo ufw allow 32004/tcp comment 'RP Testnet: EL JSON-RPC WS'
sudo ufw allow 33000/tcp comment 'RP Testnet: CL Peer TCP'
sudo ufw allow 33000/udp comment 'RP Testnet: CL Peer UDP'
sudo ufw allow 33001/tcp comment 'RP Testnet: CL Beacon REST'
sudo ufw allow 33002/tcp comment 'RP Testnet: CL Metrics'
sudo ufw allow 33003/udp comment 'RP Testnet: CL QUIC Discovery'
sudo ufw allow 34000/tcp comment 'RP Testnet: VC Metrics / API'
sudo ufw allow 37000/tcp comment 'RP Testnet: MEV-Boost / Relay'
sudo ufw allow 36000/tcp comment 'RP Testnet: Additional Services (Grafana, etc.)'
```


