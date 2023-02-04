# Voi Node Installation and Participation Guide - Ubuntu 

## Install Algorand Binaries and Reconfigure for Voi

After installing Ubuntu, open a terminal and execute the following commands one at a time.

```
sudo apt-get update
sudo apt-get install -y gnupg2 curl software-properties-common
curl -o - https://releases.algorand.com/key.pub | sudo tee /etc/apt/trusted.gpg.d/algorand.asc
sudo add-apt-repository "deb [arch=amd64] https://releases.algorand.com/deb/ stable main"
sudo apt-get update

# To get both algorand and the devtools:
sudo apt-get install -y algorand-devtools

# Verify installation
algod -v
```

### Set environment variable for Algorand Data
Once the Algorand binaries are installed, set your terminal's profile to reference the Algorand data in the default location.

```
echo "export ALGORAND_DATA=/var/lib/algorand" >> .bashrc
source .bashrc
```

### Reconfigure node to Voi Network

```
systemctl stop algorand
sudo -u algorand algocfg set -p DNSBootstrapID -v "<network>.voi.network" -d /var/lib/algorand
sudo -u algorand algocfg set -p GossipFanout -v 9 -d /var/lib/algorand
sudo -u algorand algocfg set -p EnableCatchupFromArchiveServers -v true -d /var/lib/algorand
```

After entering, the commands, you can verify that they have been set properly by executing
```
cat /var/lib/algorand/config.json
```

This should produce the following output
```
{
	"GossipFanout": 9,
	"DNSBootstrapID": "<network>.voi.network",
	"EnableCatchupFromArchiveServers": true
}
```

### Replace the genesis file

```
sudo curl -s -o /var/lib/algorand/genesis.json https://voitest-api.algorpc.pro/genesis
```

### Enable Telemetry

You can enable telemetry by executing the following commands. You should replace `<name>` with your desired hostname
```
sudo -u algorand diagcfg telemetry name -n <name>
sudo -u algorand diagcfg telemetry enable
```


### Start your node
```
systemctl start algorand

# You can verify it's running with the following command. You should see voi-test-v1 for the Genesis ID. 
goal node status

# Catch your node up using the catchpoint
sudo apt install -y jq 
goal node catchup $(curl -s https://voitest-api.algorpc.pro/v2/status|jq -r '.["last-catchpoint"]')

# You can watch the status of the catchup with 
watch goal node status

# You can exit at any point with Ctrl + C.  The node is caught up once the sync time goes back to 0.0s
```

## Set up account to participate in consensus

### Create wallet
You can create a new wallet via the terminal with the command

```
goal wallet new VoiParticipation
```

If you want to import an account that you already have the mnemonic for, you can skip seeing the phrase for the new account. Otherwise, copy the account so that you can import it.

```
goal account import

# After pasting a mnemonic to import, you can verify it is there 
goal account list
```

At this point, you should see that the account is offline and has 0 microAlgos (microVoi).  To add participation keys to this account, execute the following commonds. You should replace the values in `<>` with actual values and no brackets.

### Create participation keys
```
goal node status

# Get the value for last committed block

goal account addpartkey --roundFirstValid <last block> --roundLastValid <last block + 3000000> --address <address>

# Once it's finish, you can see the information about your participation key
goal account listpartkeys
goal account partkeyinfo
```

Your participation keys are created, but you have to send a key registration transaction to the network before your node can particpate. To do this, you need to have some microVoi in your account. 

### Fund wallet from Faucet

You can fund your wallet with some microVoi from the [Testnet Faucet](https://testnet.voifaucet.com/)
Enter in the wallet address you imported earlier and submit. You can verify that you have some in the terminal

```
goal account list
```

### Register your account online

```
goal node status

# Copy the last committed block from the status and use it in the following command

goal account changeonlinestatus --address=<address> --fee=2000 --firstvalid=<last block> --lastvalid=<last block + 1000> --online=true
```

Once you see that the transaction has been committed in a block, you can verify that your node is online and participating in consensus

```
goal account list   
# You should see online now


goal account listpartkeys
# You should see registered set to yes
```

It may take a little while for your node to actually participate, you can occassionally check to see if your node has participated yet.

`goal node partkeyinfo`


