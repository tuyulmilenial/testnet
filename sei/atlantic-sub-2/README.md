<strong><p style="font-size:14px" align="left">Founder :
<a href="https://discord.gg/JqQNcwff2e" target="_blank">NodeX Capital Discord</a></p></strong>
<strong><p style="font-size:14px" align="left">Visit Our Website : 
<a href="https://nodex.codes/" target="_blank">https://nodex.codes</a></p></strong>
<strong><p style="font-size:14px" align="left">Follow Me :
<a href="https://twitter.com/nodexploit/" target="_blank">NodeX Twitter</a></p></strong>
<hr>

<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/169664551-39020c2e-fa95-483b-916b-c52ce4cb907c.png">
</p>

# sei node setup for Testnet — atlantic-sub-2

Chain explorer:
>- [Explorer from Nodes.Guru](https://sei.explorers.guru/)

## Usefull tools and references
>
> To migrate your validator to another machine read [Migrate your validator to another machine](https://github.com/nodesxploit/testnet/blob/main/sei/migrate_validator.md)

## Hardware Requirements
Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

### Minimum Hardware Requirements
 - 3x CPUs; the faster clock speed the better
 - 4GB RAM
 - 80GB Disk
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

### Recommended Hardware Requirements 
 - 4x CPUs; the faster clock speed the better
 - 8GB RAM
 - 200GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

## Set up your sei fullnode
### Option 1 (automatic)
You can setup your sei fullnode in few minutes by using automated script below. It will prompt you to input your validator node name!
```
wget -O sei.sh https://raw.githubusercontent.com/nodesxploit/testnet/main/sei/atlantic-sub-2/sei.sh && chmod +x sei.sh && ./sei.sh
```

### Option 2 (manual)
You can follow [manual guide](https://github.com/nodesxploit/testnet/blob/main/sei/atlantic-sub-2/manual_install.md) if you better prefer setting up node manually

## Post installation

When installation is finished please load variables into system
```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
seid status 2>&1 | jq .SyncInfo
```

### Data snapshot
```
N/A
```

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
seid keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
seid keys add $WALLET --recover
```

To get current list of wallets
```
seid keys list
```

### Save wallet info
Add wallet and valoper address and load variables into the system
```
SEI_WALLET_ADDRESS=$(seid keys show $WALLET -a)
SEI_VALOPER_ADDRESS=$(seid keys show $WALLET --bech val -a)
echo 'export SEI_WALLET_ADDRESS='${SEI_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export SEI_VALOPER_ADDRESS='${SEI_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Fund your wallet
To top up your wallet join [Sei discord server](https://discord.gg/sPsUN6ay) and navigate to **#atlantic-sub-2-faucet** channel

To request a faucet grant:
```
!faucet <YOUR_WALLET_ADDRESS>
```

### Create validator
Before creating validator please make sure that you have at least 1 sei (1 sei is equal to 1000000 usei) and your node is synchronized

To check your wallet balance:
```
seid query bank balances $SEI_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
seid tx staking create-validator \
  --amount 1000000usei \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(seid tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $SEI_CHAIN_ID
```

## Security
To protect you keys please make sure you follow basic security rules

### Set up ssh keys for authentication
Good tutorial on how to set up ssh keys for authentication to your server can be found [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

### Basic Firewall security
Start by checking the status of ufw.
```
sudo ufw status
```

Sets the default to allow outgoing connections, deny all incoming except ssh and 26656. Limit SSH login attempts
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow ${SEI_PORT}656,${SEI_PORT}660/tcp
sudo ufw enable
```


## Calculate synchronization time
This script will help you to estimate how much time it will take to fully synchronize your node\
It measures average blocks per minute that are being synchronized for period of 5 minutes and then gives you results
```
wget -O synctime.py https://raw.githubusercontent.com/nodesxploit/testnet/main/sei/tools/synctime.py && python3 ./synctime.py
```

### Get list of validators
```
seid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${SEI_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Usefull commands
### Service management
Check logs
```
journalctl -fu seid -o cat
```

Start service
```
sudo systemctl start seid
```

Stop service
```
sudo systemctl stop seid
```

Restart service
```
sudo systemctl restart seid
```

### Node info
Synchronization info
```
seid status 2>&1 | jq .SyncInfo
```

Validator info
```
seid status 2>&1 | jq .ValidatorInfo
```

Node info
```
seid status 2>&1 | jq .NodeInfo
```

Show node id
```
seid tendermint show-node-id
```

### Wallet operations
List of wallets
```
seid keys list
```

Recover wallet
```
seid keys add $WALLET --recover
```

Delete wallet
```
seid keys delete $WALLET
```

Get wallet balance
```
seid query bank balances $SEI_WALLET_ADDRESS
```

Transfer funds
```
seid tx bank send $SEI_WALLET_ADDRESS <TO_SEI_WALLET_ADDRESS> 10000000usei
```

### Voting
```
seid tx gov vote 1 yes --from $WALLET --chain-id=$SEI_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
seid tx staking delegate $SEI_VALOPER_ADDRESS 10000000usei --from=$WALLET --chain-id=$SEI_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
seid tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000usei --from=$WALLET --chain-id=$SEI_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
seid tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$SEI_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
seid tx distribution withdraw-rewards $SEI_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$SEI_CHAIN_ID
```

### Validator management
Edit validator
```
seid tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$SEI_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```
seid tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$SEI_CHAIN_ID \
  --gas=auto
```

### Delete node
This commands will completely remove node from server. Use at your own risk!
```
sudo systemctl stop seid
sudo systemctl disable seid
sudo rm /etc/systemd/system/sei* -rf
sudo rm $(which seid) -rf
sudo rm $HOME/.sei -rf
sudo rm $HOME/sei-chain -rf
sed -i '/SEI_/d' ~/.bash_profile
```
