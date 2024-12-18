# System requirements
Operating System: Ubuntu 22.04 LTS Recommended
CPU: 4 dedicated cores 
RAM: 16 GB
Storage: 4 TB (using snapDB) SSD Minimum,
NVMe: recommended Network 50Mb/s bandwidth

# Steps
## Step 1: Create the zippynode root directory
Choose the root directory for your installation, then goes into this directory.

For example:
```
cd ~
mkdir zippyroot
cd zippyroot
```

## Step 2: Download binaries
Download zpy-cli and zippynode. zpy-cli is used for linux terminal wallet. zippynode is used for zippy node program.

Download the zippy-cli:
```
curl -LO http://64.62.166.164:5800/zpy-cli && chmod 744 zpy-cli
```
Download the zippynode:
```
curl -LO http://64.62.166.164:5800/zippynode && chmod 744 zippynode
```

## Step 3: Generate bls key
Use the following command to generate your bls key. You need to keep this passphrase secret.
```
./zpy-cli keys generate-bls-keys --count 1 --shard 0 --passphrase
```
Note: count means number of the bls keys; shard means which shard you want to bind your BLS key to.

Example output:
```
node is  https://t.s0.n6.zippychain.ai  targetChain  
chainName id is  2
Cannot connect to node https://t.s0.n6.zippychain.ai, using Zippy mainnet endpoint https://t.s0.n6.zippychain.ai
Enter passphrase:
Repeat the passphrase:

[
  {
    "public-key": "<your public key string>",
    "private-key": "<your private key string>",
    "encrypted-private-key-path": "/home/zippyroot/<public key string>.key"
  }
]
```
After this step, you should see a <public key>.key file in your folder.

## Step 4: Creating A New Wallet
You need to provide a local account name of your choice and provide a passphrase. When creating an account, the CLI will ask you to provide a passphrase to encrypt the keystore file:
```
./zpy-cli keys add <your_wallet_name> --passphrase
```
Add a wallet named <your_wallet_name> and set a wallet password. Remember your passphrase. You will need it to decrypt the account keystore in order to send transactions & perform other actions. Also save your seed phrase (mnemonic) somewhere as well, in case you lose your keystore.

Example output:
```
node is  https://t.s0.n6.zippychain.ai  targetChain  
chainName id is  2
Enter passphrase:
Repeat the passphrase:

**Important** write this seed phrase in a safe place, it is the only way to recover your account if you ever forget your password

<a list of mnemonic words (seed phrase)>
ZPT Address: <zptxxxxx>
```

You can check the list of wallets (local accounts) with the following command:
```
./zpy-cli keys list
```
Example output:
```
node is  https://t.s0.n6.zippychain.ai  targetChain  
chainName id is  2
NAME                    		                ADDRESS

<your_wallet_name>                                   	 Bech32 Address: <zptxxxxx>
<your_wallet_name>                                   	 ERC20 Address: <0xxxxxx>
```

Note that ZippyChain uses both the ERC 20 format and Bech32 format for public addresses. But to setup a node, YOUR_WALLET_ADDRESS should use the Bech32 format in Next Step. 

To convert your ERC 20 address into the Bech32 address, please use the following [link](https://dev-utils.zippychain.ai/address).

## Step 5: Check account balance
This command shows the balance. We should keep the balance > 10001 in order to activate node validator. If you don't have enough balance, you may purchase zpt and transfer it to your wallet.
```
./zpy-cli --node="http://64.62.166.166:9500" balances <ERC20 Address>
```
Or you may use your own ip, once your node is synced with the blockchain.
```
./zpy-cli --node="http://<your ip>:19504" balances <ERC20 Address>
```

## Step 6: Create a Validator
For you to create a Validator successfully, it needs to have 10000 ZPT tokens plus the necessary fees to create the validator transaction on chain. For this reason, we recommend that you send at least 10001 but no more than 20000 ZPT tokens to your –validator-addr before you continue.
```
./zpy-cli --node="http://64.62.166.166:9500" staking create-validator --validator-addr zpt1ue80tt700e6egxs4yvxmyrv4rp55qtxyn3hx7q --amount 10000 --bls-pubkeys 71e2191adb9e07d73743062b3c543dd3f95bcdc41b71cf31bf71dab1061a5927185e8f1ee76dd344e3e52de2ac652290 --name "<wallet name>" --identity "<wallet name>" --details "<wallet name>" --security-contact "<wallet name>" --website "<wallet name>" --max-change-rate 0.1 --max-rate 0.1 --rate 0.1 --max-total-delegation 100000000 --min-self-delegation 10000 --passphrase
```
Upon the hint, input the bls passphrase, then Enter wallet keystore passphrase.


## Step 7: Run node
This command run your own node. It's suggested you run it in a separate nohup session. We're using tmux in the example:
```
tmux new -s zippysession

// then in zippysession screen:
./zippynode --log_folder ./stakenodetmp_log --min_peers 4 --bootnodes "/ip4/64.62.166.166/tcp/19876/p2p/QmdzLi9Mr3cHWnCBiz9hkK2VWVYvfpt5dDFEJUBWe5Tr4S" --network_type=localnet --verbosity=5 --p2p.security.max-conn-per-ip=200  --ip 0.0.0.0 --port 19004 --db_dir ./stakenodetestdb/ --broadcast_invalid_tx=false --http.ip=0.0.0.0 --ws.ip=0.0.0.0 --run.shard=0 --blskey_file ./<public key>.key
```
You may choose your own log_folder or db_dir, and remember to repleace the <public key>.key path in the above command from step 3.
Once started, the node will take some time to sync with the blockchain. Click [HERE](https://testnet-scan.zippychain.ai/#/) to see the current height of the blockchain. Use the journalctl command to check on the node's progress.

## Step 8: Check validator status
You can use this command to check your validator’s status.
```
./zpy-cli blockchain validator information <Bech32 Address> --node="http://64.62.166.166:9500"
```
You may also use your own node:
```
./zpy-cli blockchain validator information <Bech32 Address> --node="http://<your ip>:19504" | grep active-status
```
Example output:
```
node is  http://<your ip>:19504  targetChain  
chainName id is  2
{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "active-status": "inactive",
    "booted-status": "manually turned inactive or insufficient uptime",
    "current-epoch-performance": null,
    "currently-in-committee": false,
    "epos-status": "not eligible to be elected next epoch",
    "epos-winning-stake": null,
    "lifetime": {
      "apr": "0.000000000000000000",
      "blocks": {
        "signed": 21509,
        "to-sign": 22400
      },
      "epoch-apr": [
        {
          "apr": "1470.897264474969283975",
          "epoch": 57008
        },
        {
```
“active-status” means if you will take part in generating block processing.

## Step 9: Activate Validator and waiting for Election
If you want to join generaing block processing, you should enable your activation using the following command:
```
./zpy-cli --node="http://64.62.166.166:9500" staking edit-validator --active true --validator-addr <Bech32 Address> --passphrase
```
or use your own node:
```
./zpy-cli --node="http://<your node ip>:19504" staking edit-validator --active true --validator-addr <Bech32 Address> --passphrase
```
After running the command, checking Validator’s active-status again:
```
node is  http://64.62.166.171:19504  targetChain  
chainName id is  2
{
  "id": "0",
  "jsonrpc": "2.0",
  "result": {
    "active-status": "active",
    "booted-status": "manually turned inactive or insufficient uptime",
    "current-epoch-performance": null,
    "currently-in-committee": false,
    "epos-status": "eligible to be elected next epoch",
    "epos-winning-stake": null,
    "lifetime": {
    ...
```


> [!NOTE]
> The technical team advises that non-technical people should not participate in the validation process to avoid financial losses.
