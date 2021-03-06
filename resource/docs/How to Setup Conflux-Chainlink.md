# How to Setup a Conflux - Chainlink

Polaris - Brian

# Anchors

[Setup Chainlink Node](#setup-chainlink-node)

[Setup an External Initiator](#setup-an-external-initiator)

[Setup Solidity Smart Contract](#setup-solidity-smart-contract)

[Setup External Adapters](#setup-external-adapters)

[Connecting Everything](#connecting-everything)



#### Main References

https://www.youtube.com/watch?v=yhLEYwdO9jw

https://github.com/Conflux-Network-Global/demo-cfx-chainlink



# Setup Chainlink Node

https://docs.chain.link/docs/running-a-chainlink-node

### Install Docker on Debian

```bash
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker $USER
exit
# log in again
```

### Create a Directory for Chainlink Data

```bash
mkdir ~/.chainlink
```

### Create ENV file for ChainLink

```bash
echo "ROOT=/chainlink
LOG_LEVEL=debug
ETH_CHAIN_ID=3
MIN_OUTGOING_CONFIRMATIONS=2
LINK_CONTRACT_ADDRESS=0x20fe562d797a42dcb3399062ae9546cd06f63280
CHAINLINK_TLS_PORT=0
SECURE_COOKIES=false
ALLOW_ORIGINS=*
DATABASE_TIMEOUT=0
ETH_DISABLED=true
FEATURE_EXTERNAL_INITIATORS=true
ENABLE_EXPERIMENTAL_ADAPTERS=true
CHAINLINK_DEV=true" > ~/.chainlink/.env
```

### Setup Remote Postgres Database

We will be using heroku postgress database, Here is the link of the [How to Setup Heroku Remote Postgress Database](https://dev.to/prisma/how-to-setup-a-free-postgresql-database-on-heroku-1dc1).

After the tutorial, you should get the URI of yoour postgresql. You need to replace the `postgresql....` with your postgressql URI

```bash
# Writing the DATABASE_URL to chainlink env setup file.
echo "DATABASE_URL=postgresql://$USERNAME:$PASSWORD@$SERVER:$PORT/$DATABASE" >> ~/.chainlink/.env

echo "DATABASE_URL=postgres://cbsqtfnvjryxeh:b6f26f17d40f8cb38ec29fb843cbedf1e307b1d94eaa6e09552c6f6258367517@ec2-52-71-153-228.compute-1.amazonaws.com:5432/deaarpo18a03ir" >> ~/.chainlink/.env
```

### Run the Chainlink Node

Currently Running Chainlink `0.9.4`

```bash
cd ~/.chainlink && docker run -p 6688:6688 -v ~/.chainlink:/chainlink -it --env-file=.env smartcontract/chainlink:0.9.4 local n
```



# Setup an External Initiator

You need to install the latest go to build the initiator. Here is the [link](https://golang.org/doc/install). Or just follow the following procedure, its for linux setup.

```bash
wget https://golang.org/dl/go1.15.5.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.15.5.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
go version # to check if go is installed successfully
```

### Clone and build external initiator repo

```
git clone https://github.com/Conflux-Network-Global/external-initiator
cd external-initiator
git checkout generalized-EI
go build
```

### Create a `.env` file in the `external-initiator` folder with the following contents

```bash
echo "EI_DATABASEURL=postgres://oafmwqxyizvobl:6a1a78bca748d98a4f0ef744d2b6eefb5376172361c946375b3fe29e0c665504@ec2-52-203-165-126.compute-1.amazonaws.com:5432/d29qhrfqsjnh2
EI_CHAINLINKURL=http://localhost:6688
EI_IC_ACCESSKEY=49cafabd90d44a3682e15ef937e6f9f0
EI_IC_SECRET=okPoScjKB2+GQXkiTm3WT+wDn9ARiS7OwjYEE/BMidsd16BRlgxL+PGUhnzOJ1QB
EI_CI_ACCESSKEY=TBjdcAXPNiOAx3KA9wZ7gw8VbM2mHGJMN9qafqovZ+Y0UleJi/amU4OmuTbR7mmQ
EI_CI_SECRET=72oxivjfbLiBmlt6ZtDHnPaPNW0R+XusXoNlMXVC9l1usDYO0ob5WvrlUqZuLVQQ" > ~/external-initiator/.env
```

1. The first line `EI_DATABASEURL`'s postgres url is created using Heroku. Here is the link of the [How to Setup Heroku Remote Postgress Database](https://dev.to/prisma/how-to-setup-a-free-postgresql-database-on-heroku-1dc1).

2. and the rest of the four keys are generated via following procedure: (or you can watch youtube tutorial [here](https://www.youtube.com/watch?v=yhLEYwdO9jw), starting from 3:20)

   The 4 keys in the `external-initiator/.env` file are generated by the Chainlink node with the following process. [Link](https://docs.chain.link/docs/miscellaneous) to more Chainlink/Docker documentation.

   1. Use `docker ps` and obtain the container ID for the Chainlink node. To access the command line within the container, use `docker exec -it <containerID> /bin/bash`.
   2. Once inside the container, log in using `chainlink admin login` and the username/password from when the container is created.
   3. To create the keys, `chainlink initiators create <NAME> <URL>`. Note that in this case the name is `cfx` and the url is `http://172.17.0.1:8080/jobs` to access the locally run external-initiator using the docker container gateway. (otherwise, they are on two different networks). The 4 keys are generated in the same order as listed above.

### Run Initiator

```bash
./external-initiator "{\"name\":\"cfx-testnet\",\"type\":\"conflux\",\"url\":\"http://test.confluxrpc.org\"}" --chainlinkurl "http://localhost:6688/"
```



# Setup Solidity Smart Contract

tutorials for Deploy Smart Contract Instance is [here](https://conflux-chain.github.io/conflux-doc/javascript-example/)

There are four `js` files in `/contractInteraction`, These files are just here to help us interact with the smart contract, we will just need to add these codes to our frontend so that we can interact with our smart contract.

```bash
.
├── contract # contract folder
│   ├── abi.json # Generated from Compiling .sol
│   ├── bytecode.json # Generated from Compiling .sol
│   └── test_oracle.sol # where we store our contract (not necessary, just here to demo)
├── deploy.js # used to deploy our smart contract
├── emitEvent.js # used to emit events to our External Initiator, and to Chainlink Node...
├── getInfo.js # getting the current status of Smart Contract
└── setValue.js # Setting the Value of the Smart Contract
```

# Setup External Adapters

#### Some of the Resources

https://github.com/Conflux-Chain/js-conflux-sdk

### First you need to have npm and yarn

1. Install [npm(install LTS version)](https://tecadmin.net/install-latest-nodejs-npm-on-debian/)

2. Install [yarn](https://tecadmin.net/install-yarn-debian/)



# Connecting Everything

In order to create the necessary connections between the various components (Conflux Network, Chainlink node, EI, EA, and NBA API), two job runs on the node need to be created. This can be done by accessing the node via the `localhost:6688` address and logging in.

The first job spec is for connecting the external initiator.

```json
{
  "initiators": [
    {
      "type": "external",
      "params": {
        "name": "cfx",
        "body": {
          "endpoint": "cfx-testnet",
          "addresses": ["<Our Lottery Smart Contract Address>"]
        }
      }
    }
  ],
  "tasks": [
    {"type": "NBABridge"}
  ]
}
```

The `name` in the initiator portion is the same name as entered when using the `chainlink initiators create <NAME> <URL>` command. The `endpoint` is the same name used in the external initiator startup command. The `addresses` are the contract addresses on Conflux Network. When this job is first created, it will trigger a test payload in the EI that is sent to Conflux Network. The test payload contains a `cfx_epochNumber`, and upon successful return to the node, the EI begins polling the Conflux Network endpoint periodically (~5 seconds). When it catches an event using `cfx_getLogs`, it sends information to the `TwilioBridge` external adapter via the Chainlink node.

The second job spec is for checking when NBA Score is out.

```
{
  "initiators": [
    {
      "type": "cron",
      "params": {
        "schedule": "CRON_TZ=UTC */30 * * * * *"
      }
    }
  ],
  "tasks": [{ "type": "TwilioCheck" }, { "type": "cfxSendTx" }]
}
```

This is achieved by using a CRON job where the Chainlink node will ask the SMS adapter to check the endpoint and any new messages are passed on to the CFX adapter to send to the chain.