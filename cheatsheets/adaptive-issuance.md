# Adaptive issuance tutorial - cheatsheet
Simply copy and paste a whole cheatsheet into your favourite text editor with MD files support. 

## Before you start
Before you start developing, please get the following values:
- build name (`BUILD_NAME`), eg. `tezos/tezos:master_6ac8e796_20240205215132`
- RPC endpoint (`RPC_ENDPOINT`), eg. `https://rpc.dailynet-2024-02-06.teztnets.com`
And fill the table below starting with the values from [Dailynet](~https://teztnets.com/dailynet-about~) and later in the next steps it will be useful to have the addresses for the `baker`, `delegator` and `staker`

| Parameter            | Value    | Source                                            |
|----------------------|----------|---------------------------------------------------|
| NAME_OF_THE_INSTANCE | tutorial | You can change it                                 |
| RPC_ENDPOINT         |          | [Dailynet](~https://teztnets.com/dailynet-about~) |
| BUILD_NAME           |          | [Dailynet](~https://teztnets.com/dailynet-about~) |
| BAKER_ADDRESS        |          | Step 1.2                                          |


### Step 1: Start the baker on Dailynet
### 1.1 Start the baker: Command template
Please replace the values from the table below above 
```
docker run -it --rm --name "NAME_OF_THE_INSTANCE" --entrypoint=/bin/sh BULD_NAME -c "octez-client --endpoint RPC_ENDPOINT gen keys baker0 && octez-baker-alpha --endpoint RPC_ENDPOINT run remotely baker0 --liquidity-baking-toggle-vote pass"

```

Once the container is running, use the following commands:
- get into the container in a second tab in your terminal:
```
docker exec  -it NAME_OF_THE_INSTANCE /bin/sh
```

- initiate the config and check the constants 
```
octez-client --endpoint RPC_ENDPOINT config init

octez-client rpc get /chains/main/blocks/head/context/constants
```
- get rid of the warning about the mainnet
```
export TEZOS_CLIENT_UNSAFE_DISABLE_DISCLAIMER=Y
```
### 1.2 Get the baker's details
```
octez-client show address baker0 -S

Hash: tzxxxxxxxxxxxxxxxxxxxAE4
Public Key: edpxxxxxxxxxxxxxxxxxxxxLAwHxKh
Secret Key: unencryxxxxxxxxxxxxxxxxxxxxRRscYobZi1XbyMEACzj
```
### 1.3 Create a stash
```
octez-client gen keys stash0
octez-client show address stash0 -S

Hash: tzxxxxxxxXdDFJ
Public Key: edpkuxxxxxxxDMCbiqc
Secret Key: unencryptxxxxxxxigeT4axjsZZ9gZ3i
```
### 1.4 Fund the baker and the stash
Fund 6001 tez each:
- [ ] baker0: 
- [ ] stash0: 

Verify 6001 tez each:
- [ ] baker0: `octez-client get balance for baker0`
- [ ] stash0:  `octez-client get balance for stash0`

### 1.5 Register the `baker0` as delegate and stake:
```
octez-client register key baker0  as delegate
octez-client stake 6000 for baker0 --burn-cap 0.06425
```
### 1.6 Accept 3rd party stakers
Set up the baker to accept the third-party stakers:
```
sudo apk add jq

octez-client rpc get chains/main/blocks/head/context/delegates/BAKER_ADDRESS/active_staking_parameters | jq 

octez-client set delegate parameters for baker0 --limit-of-staking-over-baking 5 --edge-of-baking-over-staking 1

sudo apk add jq

octez-client rpc get chains/main/blocks/head/context/delegates/BAKER_ADDRESS/pending_staking_parameters | jq 

octez-client rpc get chains/main/blocks/head | jq .metadata.level_info.cycle

```

If you have this error, it means your testnet is not adaptive inssuance ready, the feature needs to be enabled:
```
{ "id": "proto.alpha.operation.manual_staking_forbidden",
  "description":
    "Manual staking operations are forbidden because staking is currently automated.",
  "data": {} }
Fatal error:
  transfer simulation failed
```

### 1.7 Set Baking Edge
```
octez-client set delegate parameters for baker0 --limit-of-staking-over-baking 5 --edge-of-baking-over-staking 0.15
```

## Step 2: Setting up a delegator and staker
### 2.1 Create a new delegator
```
octez-client gen keys delegator0
octez-client transfer 1001 from stash0 to delegator0 --burn-cap 0.06425
octez-client set delegate for delegator0 to baker0
```
### 2.2 Create a staker
```
octez-client gen keys staker0
octez-client transfer 2002 from stash0 to staker0 --burn-cap 0.06425
octez-client set delegate for staker0 to baker0
```
## Step 3: Stake!
```
octez-client stake 1000 for staker0

octez-client rpc get chains/main/blocks/head/context/contracts/STAKER_ACCOUNT/staked_balance
```

Staking should work:
```
octez-client stake 1000 for staker0
octez-client get balance for staker0
```

## Step 4: Unstake
```
octez-client unstake everything for staker0
```

**## Finalize unstake**
```
octez-client finalize unstake for staker0
```

# Details of the commands

## Baker's Setup
Let's break down what's happening in these Docker commands step by step:

1. ****`docker run -it --rm --name "name_of_the_instance" --entrypoint=/bin/sh tezos_baker_build -c "octez-client --endpoint https://public-rpc-endopoint gen keys baker0 && octez-baker-alpha --endpoint https://public-rpc-endopoint run remotely baker0 --liquidity-baking-toggle-vote pass"`****

   - `docker run`: This command is used to run a new container instance from a specified image.
   - `-it`: This option stands for interactive terminal. It allows you to interact with the container through the terminal/command line.
   - `--rm`: This means that the container is removed automatically once it stops running. This is useful for not accumulating stopped containers on your system.
   - `--name "name_of_the_instance"`: Assigns a custom name to the running container for easier reference. In this case, the container will be named "name_of_the_instance".
   - `--entrypoint=/bin/sh`: Sets the entry point of the container to the shell (`/bin/sh`). This means that instead of running the default command specified in the Docker image, the shell is started.
   - `tezos_baker_build`: This is the name of the Docker image being used. It likely contains the Tezos baking software and dependencies.
   - `-c "octez-client --endpoint https://public-rpc-endopoint gen keys baker0 && octez-baker-alpha --endpoint https://public-rpc-endopoint run remotely baker0 --liquidity-baking-toggle-vote pass"`: This is a command passed to the shell started inside the container. It's doing two things:
     - `octez-client --endpoint https://public-rpc-endopoint gen keys baker0`: Generates a new pair of cryptographic keys for a baker called "baker0" using the Octez client, and specifies the endpoint of the Tezos node to connect to.
     - `octez-baker-alpha --endpoint https://public-rpc-endopoint run remotely baker0 --liquidity-baking-toggle-vote pass`: Starts the baking process for "baker0" on the alpha (test) network, connecting to the same Tezos node, and sets the liquidity baking toggle vote to "pass".

2. ****`docker exec -it name_of_the_instance /bin/sh`****
   - `docker exec`: This command is used to execute a new command in a running container.
   - `-it`: Again, this allows for interactive use of the terminal within the container.
   - `name_of_the_instance`: Specifies the container where the command should be executed; it should match the name given to the container in the first command.
   - `/bin/sh`: The command to be executed inside the container, which in this case is starting the shell.

In summary, the first command runs a Docker container named "name_of_the_instance" based on the `tezos_baker_build` image, and within that container, it generates a new keypair for a baker and starts the baker process with a specific configuration. The second command opens an interactive shell inside the already running container, allowing you to execute further commands manually within that container environment.
