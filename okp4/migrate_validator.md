<strong><p style="font-size:14px" align="left">Founder :
<strong><p style="font-size:14px" align="left">Founder :
<a href="https://discord.gg/JqQNcwff2e" target="_blank">NodeX Capital Discord</a></p></strong>
<strong><p style="font-size:14px" align="left">Visit Our Website : 
<a href="https://nodex.codes/" target="_blank">https://nodex.codes</a></p></strong>
<strong><p style="font-size:14px" align="left">Follow Me :
<a href="https://twitter.com/nodexploit/" target="_blank">NodeX Twitter</a></p></strong>
<hr>

<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/44331529/197152847-749c938c-c385-4698-bfa5-3f159297f391.png">
</p>

# Migrate your validator to another machine

### 1. Run a new full node on a new machine
To setup full node you can follow my guide [Okp4 node setup for Testnet](https://github.com/nodesxploit/testnet/blob/main/okp4/README.md)

### 2. Confirm that you have the recovery seed phrase information for the active key running on the old machine

#### To backup your key
```
okp4d keys export mykey
```
> _This prints the private key that you can then paste into the file `mykey.backup`_

#### To get list of keys
```
okp4d keys list
```

### 3. Recover the active key of the old machine on the new machine

#### This can be done with the mnemonics
```
okp4d keys add mykey --recover
```

#### Or with the backup file `mykey.backup` from the previous step
```
okp4d keys import mykey mykey.backup
```

### 4. Wait for the new full node on the new machine to finish catching-up

#### To check synchronization status
```
okp4d status 2>&1 | jq .SyncInfo
```
> _`catching_up` should be equal to `false`_

### 5. After the new node has caught-up, stop the validator node

> _To prevent double signing, you should stop the validator node before stopping the new full node to ensure the new node is at a greater block height than the validator node_
> _If the new node is behind the old validator node, then you may double-sign blocks_

#### Stop and disable service on old machine
```
sudo systemctl stop okp4d
sudo systemctl disable okp4d
```
> _The validator should start missing blocks at this point_

### 6. Stop service on new machine
```
sudo systemctl stop okp4d
```

### 7. Move the validator's private key from the old machine to the new machine
#### Private key is located in: `~/.okp4d/config/priv_validator_key.json`

> _After being copied, the key `priv_validator_key.json` should then be removed from the old node's config directory to prevent double-signing if the node were to start back up_
```
sudo mv ~/.okp4d/config/priv_validator_key.json ~/.okp4d/bak_priv_validator_key.json
```

### 8. Start service on a new validator node
```
sudo systemctl start okp4d
```
> _The new node should start signing blocks once caught-up_

### 9. Make sure your validator is not jailed
#### To unjail your validator
```
okp4d tx slashing unjail --chain-id $OKP4D_CHAIN_ID --from mykey --gas=auto -y
```

### 10. After you ensure your validator is producing blocks and is healthy you can shut down old validator server
