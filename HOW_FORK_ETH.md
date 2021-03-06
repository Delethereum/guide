# Hitchhiker's guide to Fork Ethereum
In the beginning was the Block, the Block was with Ethereum, and the Block was Ethereum.


## Build the Genesis
As Bitcoin, also Ethereum start from a genesis block. On opposite of Bitcoin, I can define the initial constants and the property of our blockchain without modify the source-code. I write properties in json format into genesis.json file.
```
$ cat genesis.json 
{
    "config": {
        "chainId": 666,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "difficulty": "0x40",
    "gasLimit": "0x8000000",
    "alloc": {
    }
}
```
The list of all parameters 
In the following, I The main config options:
* chainId:
* difficulty:
* gasLimit:
* alloc: list of address with an initial value (PREMIED as each best scam)
Note that I don't define or mine the initial block. On opposite of Bitcoin, Ethereum makes all the job with a simple command:
```
$ geth --datadir "/opt/ethereum/ethdata" init genesis.json
```
Geth generates the genesis block and I can start to play!
In this example, I choose to don't use the default Ethereum folder, so I can start more than one peer in the same machine. I will use this data folder in the next sections.

### Premine Account
Add as many premine accounts as you wish! 
1. create a new account
```
$ geth --datadir "/opt/ethereum/ethdata" account new
```
2. edit genesis.json
```
"alloc": {
    "your_friend_address" : { "balance" : "amount"}
    }
```
Note: 10^18 = 1 ether
3. clear the genesis 
```
$ rm /opt/ethereum/ethdata/geth
```
4. build the genesis
```
$ geth --datadir "/opt/ethereum/ethdata" init genesis.json
```

## Master Node
Start the first peer of the network!
```
$ geth --datadir "/opt/ethereum/ethdata" --networkid 666 --identity "My Ethereum Node" --rpccorsdomain "*" --nodiscover --rpc --verbosity 5 --nat "none" --rpcaddr MASTER_IP
```
When the peer is open to public, a lots of other peers try to connect to my node. These peers have different genesis block, so the commucation fails, as the follow example:
```
TRACE[07-04|02:23:29] Accepted connection                      addr=201.231.140.128:33214
DEBUG[07-04|02:23:30] Adding p2p peer                          id=4178fe0c77e647ba name=Parity/v1.6.7-beta-e...                                         addr=201.231.140.128:33214 peers=2
TRACE[07-04|02:23:30] Starting protocol eth/63                 id=4178fe0c77e647ba conn=inbound
DEBUG[07-04|02:23:30] Ethereum peer connected                  id=4178fe0c77e647ba conn=inbound name=Parity/v1.6.7-beta-e128418-20170518/x86_64-linux-gnu/rustc1.17.0
DEBUG[07-04|02:23:30] Ethereum handshake failed                id=4178fe0c77e647ba conn=inbound err="Genesis block mismatch - a3c565fc15c74788 (!= db8b372432b35224)"
```
I try different combination before run successfully a node. 
Most common error:

* No Connection : set options to disable nat port mapping mechanism (--nat "none")
```
DEBUG[07-04|01:51:42] Couldn't add port mapping                proto=tcp extport=30303 intport=30303 interface="UPnP or NAT-PMP" err="no UPnP or NAT-PMP router discovered"
```
* Too many connections : set options to restrict the inbound connections (--nodiscover, --maxpeers, --maxpendpeers, --netrestrict)

* No rpc connection : set the public ip as the external interface (--rpcaddr <PUBLIC_IP>)


## Attach to Master Node
I want connect to my master node to test everything works fine! 
I only need the public ip of the master node to connect.
```
$ geth attach http://<PUBLIC_IP>:8545
Welcome to the Geth JavaScript console!

> eth.blockNumber
0
> eth.syncing
false
```
The most common error is the connection refused. In this case, I make sure to have no firewall and set the network/rpc options on my server as showed in the previous section.
```
Fatal: Failed to start the JavaScript console: api modules: Post http://<PUBLIC_IP>:8545: dial tcp <PUBLIC_IP>:8545: getsockopt: connection refused
```

## Second peer node
Now, I run a second node that should connect to my previous master node in order to grow up my network.
The precondition is to start a node with the same genesis block of the master node; else, the connection fails with the error: "Genesis block mismatch". So I need to propagate the genesis block to the current node and the run the peer.
```
$ cat /opt/ethereum/ethdata/static-nodes.json
[
"enode:///<HEX_STRING>@[<PUBLIC_IP>]:30303?discport=0"
]

$ geth --datadir "/opt/ethereum/ethdata" init genesis.json
$ geth --datadir "/opt/ethereum/ethdata" --networkid 666 --identity "My Ethereum Node" --rpc --nodiscover  --verbosity 5 --nat "none"
```
Note : the option bootnodes not work for me. Define the list of peers as static-nodes.json or attach manually the peer with the console command: admin.addPeer("enode:///<HEX_STRING>@[<PUBLIC_IP>]:30303?discport=0");
I can retrieve the master address from the log in the following format:
```
"enode://<HEX_STRING>@[::]:30301"
```
Remember this address works only for localhost, in order to access from another machine, I need to change the "[::]" into "[<PUBLIC_IP>]".

To test the connection, check the log into master node:
```
DEBUG[07-04|02:18:38] Adding p2p peer                          id=4396b22722383e3f name=Geth/Ethereum
ode...                                              addr=<PUBLIC_IP>:18688    peers=1 
TRACE[07-04|02:18:38] Starting protocol eth/63                 id=4396b22722383e3f conn=inbound 
DEBUG[07-04|02:18:38] Ethereum peer connected                  id=4396b22722383e3f conn=inbound name=Get
/EthereumNode/v1.6.7-unstable-a0aa071c/linux-386/go1.7.1 
TRACE[07-04|02:18:38] Registering sync peer                    peer=4396b22722383e3f 
```

## Grow up the network
Now, I want start 10/100/1000 new peers! It is very tedious define to each peer the list of all the others. So I need a different solution to manage the list of peers.
I need to bootstrap node that others can use to find each other in your network and/or over the internet.
```
bootnode --genkey=boot.key
bootnode --nodekey=boot.key
```
In the output is showed the current enode address to pass as bootnodes option in the geth command.

## Mining and be happy


1. generate miner account on local node
```
$ geth --datadir "/opt/ethereum/ethdata" account new
WARN [07-04|02:58:30] No etherbase set and no accounts found as default 
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase: 
Repeat passphrase: 
Address: {cabb18d3d18c7fe74ec21d1670127abf66ca1cc8}
```

2. restart the localnode with miner options (--mine, --etherbase). Wait the complete of generating DAG process.
```
$ geth --datadir "/opt/ethereum/ethdata" --networkid 666 --identity "My Ethereum Node" --rpc --nodiscover --verbosity 5 --mine --etherbase 0xcabb18d3d18c7fe74ec21d1670127abf66ca1cc8
```
Mining requires a lots of memory. I min in a virtual machine with  <1024MB free ram, and geth crash.
```
ERROR[07-04|03:15:27] Failed to generate mapped ethash dataset epoch=1 err="cannot allocate memory"
runtime: out of memory: cannot allocate 2164260864-byte block (136445952 in use)
```

3. Use a real miner like ethminer : ethereum cpu mining is really inefficient.
https://github.com/ethereum-mining/ethminer


## Live longer


## Wallet
Download and install Mist: https://github.com/ethereum/mist .

Run a local peer:
```
$ geth --datadir "/opt/ethereum/ethdata" --networkid 666 --identity "My Ethereum Node" --rpc --nodiscover  --verbosity 5 --nat "none"
```
In the rootdir of Mist:
```
$ cd interface && meteor --no-release-check
```
In another console:
```
$ node_modules/electron/dist/electron . --rpc /opt/ethereum/ethdata/geth.ipc
```
Note:
* Popup freezed "Connecting to 1 peer": when your local peer node is not runned or it is not available, electron runs a new instance of geth with default parameters. Elecron support also some commandline options to pass to geth (--node option) but I can't make them work correctly. 

## Mining Pool
Open Source Ethereum Mining Pool: https://github.com/sammy007/open-ethereum-pool .
1. Generate new address
```
$ geth --datadir "/opt/ethereum/ethdata" account new
WARN [07-09|03:00:36] No etherbase set and no accounts found as default 
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase: 
Repeat passphrase: 
Address: {433de02034e1e651f66085dbf4e3bac0b78c9938}
```
2. Start local peer with mine options activated
```
$ geth --datadir "/opt/ethereum/ethdata" --networkid 666 --identity "My Ethereum Node" --rpc --nodiscover -verbosity 5 --mine --etherbase 433de02034e1e651f66085dbf4e3bac0b78c9938
```
3. Edit Api config.json file:
* if the peers is not localhost edit the remote ip url: "upstream"->"url", "unlocker"->"daemon", "payouts"->"daemon" 
* set pool fee and address: "unlocker"->"poolFee", "unlocker"->"poolFeeAddress"
* set payouts address: "payouts"->"address"
3. Running Api Pool:
```
$ ./build/bin/open-ethereum-pool config.json
```
4. Edit Frontend config file:
```
$ vim www/config/environment.js:
...
APP: {
      // API host and port
      ApiUrl: '//domain.com/',

      // HTTP mining endpoint
      HttpHost: 'http://domain.com',
      HttpPort: 8888,

      // Stratum mining endpoint
      StratumHost: 'domain.com',
      StratumPort: 8008,

      // Fee and payout details
      PoolFee: '100%',
      PayoutThreshold: '0.5 Ether',

      // For network hashrate (change for your favourite fork)
      BlockTime: 14.4
    }
...
```
5. Building Frontend:
```
$ cd www && ./build.sh
```
6. Configure Nginx
```
server {
        listen 80;
        server_name www.domain.com domain.com;
        root /opt/open-ethereum-pool/www/dist/;
        index index.html;
        location / {
                try_files $uri $uri/ =404;
        }
        location /api {
                proxy_pass http://api;
        }
}
```
## NetStat.io
Ethereum Network Stats : https://github.com/cubedro/eth-netstats .


1. Cloning utils
```
$ git clone https://github.com/ethersphere/eth-utils
cd eth-utils
```
2. Buid config
```
bash netstatconf.sh <number_of_clusters> <name_prefix> <ws_server> <ws_secret> 
$ bash netstatconf.sh 2 center http://localhost:3301 password > /opt/delethereum/ethstats.json
```
3. Installing eth-net-intelligence-api
```
$ git clone https://github.com/cubedro/eth-net-intelligence-api
$ cd eth-net-intelligence-api
$ npm install
$ sudo npm install -g pm2
$ pm2 start app.json /opt/delethereum/ethstats.json
```
4. Installing eth-nets
```
$ git clone https://github.com/cubedro/eth-netstats
$ cd eth-netstats
$ npm install
```
5. Edif peernode list:
```
$ vim lib/utils/config.js
var trusted = [
        '127.0.0.1'];
...
```
6. Build & Run the full version run
```
grunt
PORT=3301 WS_SECRET=432432 npm start
```

## Explorer
EthereumClassic Block Explorer: https://github.com/ethereumproject/explorer
1. Download
```
git clone https://github.com/ethereumproject/explorer
cd explorer
```
2. Edit configuration file: /tools/config.json
```
{
    "gethPort": 8545, 
    "blocks": [ {"start": 0, "end": "latest"}],
    "quiet": false,
    "terminateAtExistingDB": true,
    "listenOnly": false
}
```
3. Run
```
nohup node ./tools/grabber.js &
nohup ./tools/stats.js &
```

## Resources
* https://github.com/ethereum/go-ethereum/wiki/Private-network
* https://souptacular.gitbooks.io/ethereum-tutorials-and-tips-by-hudson/content/private-chain.html
* https://github.com/ethereum/mist
* https://github.com/sammy007/open-ethereum-pool
bash netstatconf.sh <number_of_clusters> <name_prefix> <ws_server> <ws_secret> 

bash netstatconf.sh 2 center http://localhost:3301 432432 > /opt/delethereum/ethstats.json
