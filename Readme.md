# Morph's quick guide to getting Sentinel DVPN client setup on Ubuntu

## Install the dependencies

```bash
sudo apt-get update && \
sudo apt-get install golang make curl wireguard-tools && \
git clone https://github.com/sentinel-official/cli-client.git "${HOME}/sentinelcli"
```

## Install the Sentinel CLI

```bash
git clone https://github.com/sentinel-official/cli-client.git "${HOME}/sentinelcli"
cd "${HOME}/sentinelcli"
git checkout vX.X.X # always make sure it's the latest version
make install
sudo ln -s "${HOME}/sentinelcli" /usr/local/bin/sentinelcli
```

## Generate a new keypair

```bash
sentinelcli keys add \
    --home "${HOME}/.sentinelcli" \
    --keyring-backend file \
    default
```
You will get a pass phrase (DON'T LOSE IT) - and an address - you can send a _small_ test amount of DVPN to this address using Kelpr or another compatible wallet. I send 577 DVPN (which is about $0.70 at current rates) as proof of concept.

## Find a list of nodes you can subscribe to

```bash 
sentinelcli query nodes \
    --node https://rpc.sentinel.co:443 \
    --status Active \
    --page 1
```
This will print a table of active notes to connect to - choose one - you'll need their `sentnode` address.

You need to subscribe to them - the format is as follows:

```bash
sentinelcli tx node subscribe {id-of-node-to-subscribe-to} {gigabytes-to-subscribe} {hours-to-subscribe} udvpn \
  --chain-id sentinelhub-2 \
  --node https://rpc.sentinel.co:443 \
  --gas-prices 0.1udvpn \
  --from default
```

> **_NOTE:_**  you either put gigabytes OR hours - the other one needs to be zero  

A 'real world' example would look like this:

```bash
sentinelcli tx node subscribe sentnode1qmx0zyuqww9t0pvtl8glsnhzgmrn3lyjsteacy 1 0 udvpn \
  --chain-id sentinelhub-2 \
  --node https://rpc.sentinel.co:443 \
  --gas-prices 0.1udvpn \
  --from default
```
You now need your subscription number - which can be found like this (where `[MY-ADDRESS]` is your `sent` address):

```bash
sentinelcli query subscriptions \
    --home "${HOME}/.sentinelcli" \
    --node https://rpc.sentinel.co:443 \
    --address [MY-ADDRESS]
```

Use this subscription number in the connect command

sudo sentinelcli connect 513121 q   \
    --chain-id sentinelhub-2 \
    --node https://rpc.sentinel.co:443 \
    --gas-prices 0.1udvpn \
    --yes \
    --home "${HOME}/.sentinelcli" \
    --keyring-backend file \
	--from default

After a bunch of wireguard commands run - you should be on the VPN

You can check by running:

`curl ifconfig.co`

and checking the IP against your normal one.

To disconnect simply run:

```bash
sudo sentinelcli disconnect \
    --home "${HOME}/.sentinelcli"
```