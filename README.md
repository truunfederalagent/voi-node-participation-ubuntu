# Voi Node Installation and Participation Guide - Ubuntu 

## Install Algorand Binaries

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

## Set environment variable for Algorand Data
Once the Algorand binaries are installed, set your terminal's profile to reference the Algorand data in the default location.

```
echo "export ALGORAND_DATA=/var/lib/algorand" >> .bashrc
source .bashrc
```

## Reconfigure node to Voi Network

```
systemctl stop algorand
sudo -u algorand algocfg set -p DNSBootstrapID -v "<network>.voi.network" -d /var/lib/algorand
sudo -u algorand algocfg set -p GossipFanout -v 9 -d /var/lib/algorand
sudo -u algorand algocfg set -p EnableCatchupFromArchiveServers -v true -d /var/lib/algorand
```

After entering, the commands, you can verify that they have been set properly by executing
```
/var/lib/algorand/config.json
```

This should produce the following output
```
{
	"GossipFanout": 9,
	"DNSBootstrapID": "<network>.voi.network",
	"EnableCatchupFromArchiveServers": true
}
```

## Replace the genesis file

```
sudo curl -s -o /var/lib/algorand/genesis.json https://voitest-api.algorpc.pro/genesis
```

## Enable Telemetry

You can enable telemetry by executing the following commands. You should replace `<name>` with your desired hostname
```
sudo -u algorand diagcfg telemetry name -n <name>
sudo -u algorand diagcfg telemetry enable
```


## Start your node
```
systemctl start algorand

# You can verify it's running with the following command
goal node status

# Catch your node up using the catchpoint
sudo apt install -y jq 
goal node catchup $(curl -s https://voitest-api.algorpc.pro/v2/status|jq -r '.["last-catchpoint"]')

# You can watch the status of the catchup with 
watch goal node status

# You can exit at any point with Ctrl + C.  The node is caught up once the sync time goes back to 0.0s
```


