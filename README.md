# Crossfi

### Crossfi node Installation Instructions.

[Official documentation](https://crossfi.org/documents)

System requirements:</br>
CPU: Quad Core or larger AMD or Intel (amd64) CPU
RAM:32GB RAM
SSD:1TB NVMe Storage
100MBps bidirectional internet connection
OS: Ubuntu 20.04 or 22.04</br>

You can take a weaker server

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>

### Installing the Babylon Node

1. Preparing the server/Required packages installation</br>
```
sudo apt update
sudo apt upgrade -y
sudo apt-get install libclang-dev
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```
### Go installation.
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd $HOME && mkdir -p $HOME/go/bin
curl -L https://github.com/crossfichain/crossfi-node/releases/download/v0.3.0-prebuild9/crossfi-node_0.3.0-prebuild9_linux_amd64.tar.gz > crossfi-node_0.3.0-prebuild9_linux_amd64.tar.gz
tar -xvzf crossfi-node_0.3.0-prebuild9_linux_amd64.tar.gz
chmod +x $HOME/bin/crossfid
mv $HOME/bin/crossfid $HOME/go/bin
rm -rf crossfi-node_0.3.0-prebuild9_linux_amd64.tar.gz readme.md $HOME/bin
```

# Config and init app
```
crossfid config chain-id crossfi-evm-testnet-1
crossfid config keyring-backend test
crossfid config node tcp://localhost:26057
crossfid init "your moniker" --chain-id crossfi-evm-testnet-1
```

# Download genesis and addrbook
```
curl -L https://snapshots-testnet.nodejumper.io/crossfi-testnet/genesis.json > $HOME/.mineplex-chain/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/crossfi-testnet/addrbook.json > $HOME/.mineplex-chain/config/addrbook.json
```

# Set seeds and peers
```
sed -i -e 's|^seeds *=.*|seeds = "89752fa7945a06e972d7d860222a5eeaeab5c357@128.140.70.97:26656,dd83e3c7c4e783f8a46dbb010ec8853135d29df0@crossfi-testnet-seed.itrocket.net:36656"|' $HOME/.mineplex-chain/config/config.toml
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "5000000000mpx"|' $HOME/.mineplex-chain/config/app.toml
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.mineplex-chain/config/app.toml
```

# Create service file
```
sudo tee /etc/systemd/system/crossfid.service > /dev/null << EOF
[Unit]
Description=CrossFi node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which crossfid) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable crossfid.service
```

# Reset and download snapshot
```
curl "https://snapshots-testnet.nodejumper.io/crossfi-testnet/crossfi-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.mineplex-chain"
```

# enable and start service
```
sudo systemctl start crossfid.service
sudo journalctl -u crossfid.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
crossfid keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
crossfid keys add wallet --recover
```

# check sync status, once your node is fully synced, the output from above will print "false"
```
crossfid status 2>&1 | jq -r '.SyncInfo.catching_up // .sync_info.catching_up'
```

### We receive tokens from the tap in the [discord](https://discord.gg/crossfi)
```
#testnetwallet /faucet adress
```

# before creating a validator, you need to fund your wallet and check balance
```
crossfid q bank balances $(crossfid keys show wallet -a)
```
# Create validator
```
crossfid tx staking create-validator \
--amount=1000000mpx \
--pubkey=$(crossfid tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO❤️" \
--chain-id=crossfi-evm-testnet-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=5000000000mpx \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
No update

Current network:zgtendermint_9000-1
Current version:v1.0.0-testnet
```

### Useful commands

Check balance
```
crossfid q bank balances $(crossfid keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -u crossfid -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart crossfid
```

GET VALIDATOR INFO
```
crossfid status 2>&1 | jq -r '.ValidatorInfo // .validator_info'
```

DELEGATE TOKENS TO YOURSELF
```
crossfid tx staking delegate $(crossfid keys show wallet --bech val -a) 1000000mpx --from wallet --chain-id crossfi-evm-testnet-1 --gas-prices 5000000000mpx --gas-adjustment 1.5 --gas auto -y  
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop crossfid && sudo systemctl disable crossfid && sudo rm /etc/systemd/system/crossfid.service && sudo systemctl daemon-reload && rm -rf $HOME/.mineplex-chain && rm -rf crossfi-node && sudo rm -rf $(which crossfid) 
```
