# Adaptive issuance tutorial - cheatsheet
Simply copy and paste a whole cheatsheet into your favourite text editor with MD files support. 

## Setup
   Start a dailynet baker, based on [Dailynet](https://teztnets.com/dailynet-about)

Before you start developing, please get the following values:
- name of the instance (`NAME_OF_THE_INSTANCE`), eg. `dailynet_walkthrough`
- build name (`BUILD_NAME`), eg. `tezos/tezos:master_6ac8e796_20240205215132`
- RPC endpoint (`RPC_ENDPOINT`), eg. `https://rpc.dailynet-2024-02-06.teztnets.com`

| Parameter            | Value |
|----------------------|-------|
| NAME_OF_THE_INSTANCE |       |
| BUILD_NAME           |       |
| RPC_ENDPOINT         |       |

### Command template
== Please replace the values from above with the up-to-date onces. ==
```
docker run -it --rm --name "NAME_OF_THE_INSTANCE" --entrypoint=/bin/sh BULD_NAME -c "octez-client --endpoint RPC_ENDPOINT gen keys baker0 && octez-baker-alpha --endpoint RPC_ENDPOINT run remotely baker0 --liquidity-baking-toggle-vote pass"

docker exec  -it NAME_OF_THE_INSTANCE /bin/sh

```

### Example command
```
docker run -it --rm --name "dailynet_walkthrough" --entrypoint=/bin/sh tezos/tezos:master_6ac8e796_20240205215132 -c "octez-client --endpoint https://rpc.dailynet-2024-02-06.teztnets.com gen keys baker0 && octez-baker-alpha --endpoint https://rpc.dailynet-2024-02-06.teztnets.com run remotely baker0 --liquidity-baking-toggle-vote pass"

docker exec  -it dailynet_walkthrough /bin/sh

```
Once the container is running, use the following commands. 


```
octez-client --endpoint https://rpc.dailynet-2024-02-06.teztnets.com config init

octez-client rpc get /chains/main/blocks/head/context/constants
```

```
export TEZOS_CLIENT_UNSAFE_DISABLE_DISCLAIMER=Y
```
### Get the baker's details
```
octez-client show address baker0 -S

Hash: tzxxxxxxxxxxxxxxxxxxxAE4
Public Key: edpxxxxxxxxxxxxxxxxxxxxLAwHxKh
Secret Key: unencryxxxxxxxxxxxxxxxxxxxxRRscYobZi1XbyMEACzj
```
### Create a stash
```
octez-client gen keys stash0
octez-client show address stash0 -S

Hash: tzxxxxxxxXdDFJ
Public Key: edpkuxxxxxxxDMCbiqc
Secret Key: unencryptxxxxxxxigeT4axjsZZ9gZ3i
```
### Fund the baker and the stash
Fund 6001 tez each:
- [ ] Baker0: 
- [ ] Stash0

Verify 6001 tez each:
- [ ] Backer0: `octez-client get balance for baker0`
- [ ] Stash0:  `octez-client get balance for stash0`

Register the `baker0` as delegate and stake:
```
octez-client register key baker0  as delegate
octez-client stake 6000 for baker0 --burn-cap 0.06425
```
### Accept 3rd party stakers
Set up the baker to accept the third-party stakers:
```
sudo apk add jq

octez-client rpc get chains/main/blocks/head/context/delegates/tz1Yj2pi1s3NK6xakM8zNBPSLjvdRRkroAE4/active_staking_parameters | jq 

octez-client set delegate parameters for baker0 --limit-of-staking-over-baking 5 --edge-of-baking-over-staking 1

sudo apk add jq

octez-client rpc get chains/main/blocks/head/context/delegates/tz1Yj2pi1s3NK6xakM8zNBPSLjvdRRkroAE4/pending_staking_parameters | jq 

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

### Set Baking Edge
```
octez-client set delegate parameters for baker0 --limit-of-staking-over-baking 5 --edge-of-baking-over-staking 0.15
```

## Setting up a delegator and staker
1. Create a new delegator
```
octez-client gen keys delegator0
octez-client transfer 1001 from stash0 to delegator0 --burn-cap 0.06425
octez-client set delegate for delegator0 to baker0
```
2. Create a staker
```
octez-client gen keys staker0
octez-client transfer 2002 from stash0 to staker0 --burn-cap 0.06425
octez-client set delegate for staker0 to baker0
```
3. Stake!
```
octez-client stake 1000 for staker0

octez-client rpc get chains/main/blocks/head/context/contracts/tz1XP1fBFnoQiq8yrxLkJ1ihKTk22uugkMzY/staked_balance
```

4. Staking should work:
```
octez-client stake 1000 for staker0
octez-client get balance for staker0
```

## Unstake
```
octez-client unstake everything for staker0
```

## Finalize unstake
```
octez-client finalize unstake for staker0
```

# Details of the commands

## Baker's Setup

Let's break down what's happening in these Docker commands step by step:

1. **`docker run -it --rm --name "name_of_the_instance" --entrypoint=/bin/sh tezos_baker_build -c "octez-client --endpoint https://public-rpc-endopoint gen keys baker0 && octez-baker-alpha --endpoint https://public-rpc-endopoint run remotely baker0 --liquidity-baking-toggle-vote pass"`**

   - `docker run`: This command is used to run a new container instance from a specified image.
   - `-it`: This option stands for interactive terminal. It allows you to interact with the container through the terminal/command line.
   - `--rm`: This means that the container is removed automatically once it stops running. This is useful for not accumulating stopped containers on your system.
   - `--name "name_of_the_instance"`: Assigns a custom name to the running container for easier reference. In this case, the container will be named "name_of_the_instance".
   - `--entrypoint=/bin/sh`: Sets the entry point of the container to the shell (`/bin/sh`). This means that instead of running the default command specified in the Docker image, the shell is started.
   - `tezos_baker_build`: This is the name of the Docker image being used. It likely contains the Tezos baking software and dependencies.
   - `-c "octez-client --endpoint https://public-rpc-endopoint gen keys baker0 && octez-baker-alpha --endpoint https://public-rpc-endopoint run remotely baker0 --liquidity-baking-toggle-vote pass"`: This is a command passed to the shell started inside the container. It's doing two things:
     - `octez-client --endpoint https://public-rpc-endopoint gen keys baker0`: Generates a new pair of cryptographic keys for a baker called "baker0" using the Octez client, and specifies the endpoint of the Tezos node to connect to.
     - `octez-baker-alpha --endpoint https://public-rpc-endopoint run remotely baker0 --liquidity-baking-toggle-vote pass`: Starts the baking process for "baker0" on the alpha (test) network, connecting to the same Tezos node, and sets the liquidity baking toggle vote to "pass".

2. **`docker exec -it name_of_the_instance /bin/sh`**
   - `docker exec`: This command is used to execute a new command in a running container.
   - `-it`: Again, this allows for interactive use of the terminal within the container.
   - `name_of_the_instance`: Specifies the container where the command should be executed; it should match the name given to the container in the first command.
   - `/bin/sh`: The command to be executed inside the container, which in this case is starting the shell.

In summary, the first command runs a Docker container named "name_of_the_instance" based on the `tezos_baker_build` image, and within that container, it generates a new keypair for a baker and starts the baker process with a specific configuration. The second command opens an interactive shell inside the already running container, allowing you to execute further commands manually within that container environment.