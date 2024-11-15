# Demonstrate One Way Bridge from Bitcoin to Merlin Network 

## Scope

Demonstrate that by locking a specified amount of Bitcoin in a Taproot (P2TR) address on the Bitcoin Testnet, we can mint an equivalent ERC20 token ("BitSnark" - BIS) on the Merlin Testnet.

## PegIn Process

The PegIn process allows a user to lock Bitcoin (BTC) on the Bitcoin network (Layer 1) and mint an equivalent amount of ERC20 tokens on an EVM-compatible blockchain (Layer 2). 

- At the request of an agent initiating a PegIn, all agents collaborate to co-create a pre-signed Bitcoin Taproot (P2TR) transaction with a specified deposit amount.
- Using this pre-signed transaction, the requesting agent creates a PegIn request on the EVM network.
- This PegIn request must be confirmed by all agents, showing their commitment to the transaction.
- Once all confirmations are obtained, the agent completes the Bitcoin deposit into the Taproot address. 
- The requesting agent then generates a zk-SNARK proof verifying the Bitcoin deposit.
- Upon verification and proof of deposit, the PegIn request is finalized, triggering the minting of equivalent ERC20 tokens on Merlin, credited to the requester’s address.

## Demo Script

### Consideration

- For this demo, we assume that agents have already collaborated to generate the pre-signed Bitcoin transaction. 

- The BTC deposit to the Taproot (P2TR) address on Bitcoin Testnet has been completed, but the PegIn request and approval are still pending. Ideally, the deposit should occur just before the PegIn transaction and after the PegIn request approval for added security.

- The Bitcoin oracle has already been updated with the block containing this transaction, with a minimum confirmation depth of six blocks. This oracle is permissionless and has been updated only to streamline the demo. We could do the demo without this pre-update, with the prover updating it himself during the demo; however, it would just be annoying.

### Execution Details

We will run a script that performs the following steps:

#### Pre-PegIn Transaction

1. The requesting agent **creates the PegIn** request on the Merlin network.
2. All **agents confirm the PegIn request**, approving it for further processing. (2 agents total)

#### PegIn Transaction 

3. Fetch and **parse UTXO data from Blockstream**, gathering the necessary details for the deposit proof generation.
4. **Generate the PegIn proof** based on the parsed UTXO data, verifying the Bitcoin deposit.
5. **Execute the PegIn transaction**, minting the equivalent ERC20 tokens (BitSnark) on Merlin.

## Demo Configuration

### Scripts and Tools

For this demo, we utilize the following tools and scripts:

- **Agent CLI Local Test**: The `packages/agent-cli/.cli-merlin-demo.sh` script execute all steps of the PegIn process using the `agent-cli`
- **Contract Deployment**: Contracts have been deployed using `packages/contracts/ignition` `packages/contracts/ignition/params/grail-merlin-deployment-testnet.json`
- **Bitcoin Oracle Updates**: The `npx hardhat oracle-submit-headers ` task updates the Bitcoin Oracle with the required block header information using `./scripts/block-headers-merlin.json`
- **Agent Confirmation Script**: The `packages/agent-cli/tests/utils/all-agents-confirm.ts` script allows all configured agents to confirm the PegIn. Run using `npm run test:confirm`.

### Initial Data

#### Bitcoin Locked Transaction

We will executing the PegIn process using the following UTXO (Unspent Transaction Output) details:

- **Block Height**: 3433935
- **Bitcoin Transaction ID**: [`ed470838b312d5984304b8ca4734e223235d115cfa5e0a368b907287732110b8`](https://blockstream.info/testnet/tx/ed470838b312d5984304b8ca4734e223235d115cfa5e0a368b907287732110b8)
- **Pegin Address**: `f37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1`
- **Amount (Sats)**: 20000 satoshis (0.00020000 BTC)
- **Bitcoin Block Hash**: `00000000040afd9372ae26485bdaee67b391579bdd40663d5ca21636d0beabb0`

This transaction locks BTC in the Bitcoin testnet network, and we will execute the PegIn operation to mint ERC20 tokens equivalent to the locked BTC on the **Merlin** testnet blockchain.

#### Configured Agents

- __Agent 0__  `0xAAfD7D37B78E245aC85a230e161b512b0140daF7` *requesting agent*
- __Agent 1__  `0xcdF9Ba6507D7da93Efbc4dB795c56C9dFcf91C72`
- __Agent 2__  `0xc2fff2D5eDafd3bBE93f3deD0d1630A38A3baAE1`


### Contracts Deployment
```bash
$ npx hardhat ignition deploy ignition/modules/GrailBridgeDeployment.ts  --parameters ignition/params/grail-merlin-deployment-testnet.json --network merlinTestnet 
✔ Confirm deploy to network merlinTestnet (686868)? … yes
Hardhat Ignition 🚀

Deploying [ GrailBridgeDeployment ]

Batch #1
  Executed BitcoinOracleModule#BitcoinOracle
  Executed ContractRegistryModule#ContractRegistry
  Executed PeginVerifierModule#PeginVerifier

Batch #2
  Executed GrailBridgeDeployment#GrailBridge

Batch #3
  Executed GrailBridgeDeployment#ERC20BitSnark

Batch #4
  Executed GrailBridgeDeployment#ContractRegistryModule~ContractRegistry.initialize

[ GrailBridgeDeployment ] successfully deployed 🚀

Deployed Addresses

AgentRegistryModule#AgentRegistry - 0x2751421Bb6Be92Df3c1c71b22fBda6fe4f6d809F
BitcoinOracleModule#BitcoinOracle - 0x454ef0F8D285FCB61E6e53954d078410C006EdA7
ContractRegistryModule#ContractRegistry - 0xe26731E32043ad0b538174D0372b91B695c5F262
PeginVerifierModule#PeginVerifier - 0xFCFF9d87681DFf476482F4BaE4DE0Ae2b916eD48
GrailBridgeDeployment#GrailBridge - 0xB00dAB5C8b4b561cCdb7F43f4853144A05A099B0
GrailBridgeDeployment#ERC20BitSnark - 0xB86D9E9D575C8E27f63e8572D9F21Ffa8efE6950
```

### Block Headers Submission

```bash
$ npx hardhat oracle-submit-headers --file ./scripts/block-headers-merlin.json --network merlinTestnet
BitcoinOracle contract at address: 0x454ef0F8D285FCB61E6e53954d078410C006EdA7
Transaction submitted. Waiting for confirmation...
Transaction confirmed.
```

### Agent CLI Environment Configuration

In the `packages/agent-cli/` directory, you'll find all the configuration files needed for running the demo script.

1. **Pegin request file `pegin_request.json`**

`packages/agent-cli/data/merlin-demo/pegin_request.json`

```json
{
    "peginAddress": "0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1",
    "amount": "20000"
}
```

2. **`cli-merlin-demo.sh` script**

`packages/agent-cli/cli-merlin-demo.sh`

``` bash
PEGIN_REQUEST_FILE="./data/merlin-demo/pegin_request.json"
AGENTS_CONFIG_FILE="./data/merlin-demo/agents_local.json"

BLOCK_HASH="0x00000000040afd9372ae26485bdaee67b391579bdd40663d5ca21636d0beabb0"
TX_ID="0xed470838b312d5984304b8ca4734e223235d115cfa5e0a368b907287732110b8"
PEGIN_ADDRESS="0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1"
OUTPUT_INDEX=0
```

3. **Agents configuration**

`packages/agent-cli/data/merlin-demo/agents_local.json`

```json
[
    {
        "address": "0xcdF9Ba6507D7da93Efbc4dB795c56C9dFcf91C72",
        "privateKey": "xxxxx"
    },
    {
        "address": "0xc2fff2D5eDafd3bBE93f3deD0d1630A38A3baAE1",
        "privateKey": "xxxxx"
    }
]
```

4. **Agent CLI `.env` Configuration**
```bash
## MERLIN TESTNET
RPC_HOST='https://testnet-rpc.merlinchain.io'
PRIVATE_KEY=xxxxx
GRAIL_BRIDGE_ADDRESS=0xB00dAB5C8b4b561cCdb7F43f4853144A05A099B0
ERC20_ADDRESS=0xB86D9E9D575C8E27f63e8572D9F21Ffa8efE6950
DATA_PATH=data/merlin-demo
BLOCKSTREAM_URL=https://blockstream.info/testnet/api/ #TESTNET
```

## Demo Execution
```bash
  $cd packages/agent-cli/
```  

### Check Agent CLI Configuration
``` bash
$./cli.sh show-config
Environment Configuration: {
  "private_key": "xxxx",
  "rpc_host": "https://testnet-rpc.merlinchain.io",
  "grail_bridge_address": "0xB00dAB5C8b4b561cCdb7F43f4853144A05A099B0",
  "erc20_address": "0xB86D9E9D575C8E27f63e8572D9F21Ffa8efE6950",
  "default_path": "data/merlin-demo",
  "blockstream_url": "https://blockstream.info/testnet/api/"
}
```
### Check Requesting Agent Balance
```
$./cli.sh show-balance
Account Address: 0xAAfD7D37B78E245aC85a230e161b512b0140daF7
ERC20 Balance: 0.0 BTC
```

### Check Bitcoin Locked Transaction

**Bitcoin Testnet Transaction ID**: [`ed470838b312d5984304b8ca4734e223235d115cfa5e0a368b907287732110b8`](https://blockstream.info/testnet/tx/ed470838b312d5984304b8ca4734e223235d115cfa5e0a368b907287732110b8)

![bitcoin-tx](https://github.com/user-attachments/assets/09c4608a-c124-4814-966c-ae0829868fcb)


### Demo Script Execution
```
$./cli-merlin-demo.sh 

============================================================
Showing the ERC20 balance and account address
============================================================
$ ./cli.sh show-balance
Account Address: 0xAAfD7D37B78E245aC85a230e161b512b0140daF7
ERC20 Balance: 0.0 BTC

============================================================
Creating a PegIn request using the JSON file
============================================================
$ ./cli.sh create-pegin -f "./data/merlin-demo/pegin_request.json"
⠏ Create PegIn request started...
PegIn request created successfully.
Request ID: 0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1
✔ Create PegIn request completed successfully.
Create PegIn request: 10.408s

============================================================
Confirming the PegIn request with all agents
============================================================
$ npm run test:confirm "0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1" "./data/merlin-demo/agents_local.json" --silent
Pegin request confirmed

============================================================
Fetch and Parsing data from blockstream
============================================================
$ ./cli.sh fetch-data-from-blockstream "0xed470838b312d5984304b8ca4734e223235d115cfa5e0a368b907287732110b8"
⠋ Fetch and Parse transaction data from blockstream started...
Blockstream data fetched and parsed successfully.
JSON files generated at: /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0x00000000040afd9372ae26485bdaee67b391579bdd40663d5ca21636d0beabb0_block.json , /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0xed470838b312d5984304b8ca4734e223235d115cfa5e0a368b907287732110b8_transaction.json 
✔ Fetch and Parse transaction data from blockstream completed successfully.
Fetch and Parse transaction data from blockstream: 12.73ms

============================================================
Generating PegIn proof inputs file
============================================================
$ ./cli.sh generate-pegin-proof-inputs "0x00000000040afd9372ae26485bdaee67b391579bdd40663d5ca21636d0beabb0" "0xed470838b312d5984304b8ca4734e223235d115cfa5e0a368b907287732110b8" "0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1" "0"
⠋ Generate PegIn proof inputs file started...
PegIn inputs file generated successfully.
File path: /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1_inputs.json
✔ Generate PegIn proof inputs file completed successfully.
Generate PegIn proof inputs file: 24.109ms

============================================================
Generating PegIn proof file
============================================================
$ ./cli.sh generate-pegin-proof "0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1"
⠏ Generate PegIn proof started...
Proof file path: /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1_proof.json
✔ Generate PegIn proof completed successfully.
Generate PegIn proof: 1:00.193 (m:ss.mmm)

============================================================
Performing the PegIn transaction on the blockchain
============================================================
$ ./cli.sh perform-pegin "0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1" "0x00000000040afd9372ae26485bdaee67b391579bdd40663d5ca21636d0beabb0"
⠴ Perform PegIn transaction started...
PegIn transaction performed successfully.
Result: 0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1,20000,0xAAfD7D37B78E245aC85a230e161b512b0140daF7,2,2
✔ Perform PegIn transaction completed successfully.
Perform PegIn transaction: 15.742s

============================================================
Showing the ERC20 balance after PegIn transaction
============================================================
$ ./cli.sh show-balance
Account Address: 0xAAfD7D37B78E245aC85a230e161b512b0140daF7
ERC20 Balance: 0.0002 BTC

============================================================
Local CLI integration test completed successfully!
============================================================
```

#### Screencast Record
[merlin-demo-execution.webm](https://github.com/user-attachments/assets/c0cfc1b4-c05a-433a-a67f-3630b8de9c23)

### Balance ERC20 check

Here, we can see that the balance has been updated to reflect 0.0002 BTC:

```
$ ./cli.sh show-balance
Account Address: 0xAAfD7D37B78E245aC85a230e161b512b0140daF7
ERC20 Balance: 0.0002 BTC
```

### Pegin Transaction

You can view the PegIn transaction on the Merlin chain explorer:

https://testnet-scan.merlinchain.io/tx/0x872dbe30829dd704a25269be2e579d48df4245354dfaa405e084463be26a3eb1
![merlin-tx](https://github.com/user-attachments/assets/6526ed77-2631-4354-9575-61b168124587)

## Ensuring Secure PegIn Validations  

The bridge relies on a network of operators / agents who ensure the creation, configuration, and confirmation of PegIn requests, backed by the Bitsnark protocol.  

Additionally, the PegIn process leverages zk-SNARK proofs and a Bitcoin oracle to prevent invalid transactions.  

zk-SNARK proofs validate the Bitcoin transaction by verifying its inclusion in a valid block. These proofs ensure the PegIn address, deposited amount, block header, transaction Merkle tree, and outputs meet all criteria for Bitcoin blockchain inclusion.  

The onchain Bitcoin oracle maintains the canonical Bitcoin blockchain and verifies that the block containing the transaction has been finalized with sufficient confirmations, ensuring the Bitcoin deposit is irreversible before minting ERC20 tokens on the EVM side.

### Fake PegIn Demo
We assume that the agents have already created and confirmed a PegIn request for a specific address. As shown below:

PegIn request created and confirmed

```
{
    "peginAddress": "0x27ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b",
    "amount": "5000000"
}
```

```
============================================================
Creating a PegIn request using the JSON file
============================================================
$ ./cli.sh create-pegin -f "./data/merlin-demo/pegin_request.json"
⠹ Create PegIn request started...
PegIn request created successfully.
Request ID: 0x27ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b
✔ Create PegIn request completed successfully.
Create PegIn request: 12.259s
```

```
============================================================
Confirming the PegIn request with all agents
============================================================
$ npm run test:confirm "0x27ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b" "./data/merlin-demo/agents_local.json" --silent
Pegin request confirmed
```

Now, the requesting agent is now attempting to falsify the PegIn by providing a fake deposit.

#### Invalid or fake zk-proof  

We assume that the requesting agent has generated a zk-proof for a different PegIn or an invalid one and is attempting to mint ERC20 tokens using this proof.  

Generate a proof for `0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1` valid pegin.

```
$ ./cli.sh generate-pegin-proof "0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1"
⠙ Generate PegIn proof started...
Proof file path: /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1_proof.json
✔ Generate PegIn proof completed successfully.
Generate PegIn proof: 53.681s
```
Then rename the file `0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1_proof` to  `0x27ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b` pegin.

And try to mint ERC20 using this proof.

```
$ ./cli.sh perform-pegin "0x27ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b" "0x00000000000000000001b4b9f79123df985e564b11843c52e1262740f7cc732a"
⠧ Perform PegIn transaction started...Error performing PegIn transaction

Error: Exception while processing PegIn on blockchain, Error: Error: execution reverted (unknown custom error) (action="estimateGas", data="0x8355494700000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000019496e76616c696420506567696e20536e61726b2050726f6f6600000000000000", reason=null, transaction={ "data": "0xb37e111027ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b00000000000000000001b4b9f79123df985e564b11843c52e1262740f7cc732a065397d6034e1def93e00b3dade5253658f0fcc9bda95bce01e8ea0f1d58f3180d96cb89ae616ebacb542f5cb9b9bdb3f74b445eb8a7b57844686e4c75eb2626291f88623bec6d493e78a5e377c3bd856e0bfb191e11c5bd1af7e44a1f5d025104b3d586777232f1235a302e02a3ad63516c46c06fed9428fde56c7df8dfbb992ade0a2dbf9a2e87b8a1c5ed59aa1c7c9003117cbede044c1c1be70ea8daa87302efbeef6c773814dd79da55acb929c9c15d7b4c4aa0747a38574f828da5e6d5228207c59fc264f34f633018af9d4eb9eb70d053faf802f9496de9ee96c8e5cd177450ca07679eff9c434e497a3b65f05762d7fe2842106d0be77efa57b0aa92", "from": "0xAAfD7D37B78E245aC85a230e161b512b0140daF7", "to": "0xB00dAB5C8b4b561cCdb7F43f4853144A05A099B0" }, invocation=null, revert=null, code=CALL_EXCEPTION, version=6.13.3)
```

**ERROR:** `UIG Invalid Pegin Snark Proof`
 
In this case, the bridge will reject the proof because it does not correspond to a valid PegIn transaction. The zk-SNARK verification ensures that the proof matches the specific PegIn address, deposited amount, and block hash, preventing the minting of tokens based on invalid or mismatched data.

#### Block not included in the oracle
Although it is extremely difficult to generate a valid proof for a PegIn that has already been approved, as it would require achieving proof of work (PoW) for the block generation, we assume that this has been accomplished. 

For this test, we created and confirmed a valid PegIn on the mainnet. However, the PegIn is valid only on the Bitcoin mainnet, not on the testnet. To achieve this, the proof is generated in the following way:

```
===========================================================
Fetch and Parsing data from blockstream
============================================================
$ ./cli.sh fetch-data-from-blockstream "0x84228c55b75a7241481fc0c7313084848fe5e7540483ad800f6a240ed4130737"
⠋ Fetch and Parse transaction data from blockstream started...Key 0x84228c55b75a7241481fc0c7313084848fe5e7540483ad800f6a240ed4130737 not found in repository. Downloading data...
⠏ Fetch and Parse transaction data from blockstream started...Key 0x00000000000000000001b4b9f79123df985e564b11843c52e1262740f7cc732a not found in repository. Downloading data...
⠼ Fetch and Parse transaction data from blockstream started...Key 0x000000000000000000001da061256cfff70b179946be82e6b1c395a3a51a7072 not found in repository. Downloading data...
⠇ Fetch and Parse transaction data from blockstream started...Key 0x00000000000000000001b4b9f79123df985e564b11843c52e1262740f7cc732a not found in repository. Downloading data...
⠼ Fetch and Parse transaction data from blockstream started...Key 0x84228c55b75a7241481fc0c7313084848fe5e7540483ad800f6a240ed4130737 not found in repository. Downloading data...
⠇ Fetch and Parse transaction data from blockstream started...
Blockstream data fetched and parsed successfully.
JSON files generated at: /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0x00000000000000000001b4b9f79123df985e564b11843c52e1262740f7cc732a_block.json , /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0x84228c55b75a7241481fc0c7313084848fe5e7540483ad800f6a240ed4130737_transaction.json 
✔ Fetch and Parse transaction data from blockstream completed successfully.
Fetch and Parse transaction data from blockstream: 2.308s

============================================================
Generating PegIn proof inputs file
============================================================
$ ./cli.sh generate-pegin-proof-inputs "0x00000000000000000001b4b9f79123df985e564b11843c52e1262740f7cc732a" "0x84228c55b75a7241481fc0c7313084848fe5e7540483ad800f6a240ed4130737" "0x27ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b" "0"
⠋ Generate PegIn proof inputs file started...
PegIn inputs file generated successfully.
File path: /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0x27ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b_inputs.json
✔ Generate PegIn proof inputs file completed successfully.
Generate PegIn proof inputs file: 22.529ms

============================================================
Generating PegIn proof file
============================================================
$ ./cli.sh generate-pegin-proof "0x27ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b"
⠙ Generate PegIn proof started...
Proof file path: /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0x27ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b_proof.json
✔ Generate PegIn proof completed successfully.
Generate PegIn proof: 58.967s
```

Then, it will attempt to mint using the mainnet valid proof for a valid PegIn, but on the testnet.

```
$ ./cli.sh perform-pegin "0x27ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b" "0x00000000000000000001b4b9f79123df985e564b11843c52e1262740f7cc732a"
⠦ Perform PegIn transaction started...Error performing PegIn transaction: Error: Exception while processing PegIn on blockchain, Error: Error: execution reverted (unknown custom error) (action="estimateGas", data="0x835549470000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000f426c6f636b206e6f7420666f756e640000000000000000000000000000000000", reason=null, transaction={ "data": "0xb37e111027ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b00000000000000000001b4b9f79123df985e564b11843c52e1262740f7cc732a26e5a2e2b8eaa5c78b03921d0e2770703cf3b177aed50b5d57e39d7c88ec5e04187623762b653972007eb8a76935f64f151b26794d7cfa37967f2885ff2b97b806e4dbe7c064789dead616707efda64da08f4b9a278513f155253d17c551ac121eacfc130128c3a3180328011045f0b8b8b09421bfb4f28b175dfbdae6b6fedc0f8a426f7694f3561e839ea42d2ef5d763b5f8e2c04cae71e77619725ca658bf17d0f96b7a48c584e8d9cd3b96ca4e997652d4e153115f9971b4e71dba5d2c6e2980e46b722bee854f99a338b5ebec0ee0c078744de1d3958d62158cc0cae3ed0d36d6878946e147a9eec1a9545f1e125adacbc375a6ca09df6fabe417207583", "from": "0xAAfD7D37B78E245aC85a230e161b512b0140daF7", "to": "0xB00dAB5C8b4b561cCdb7F43f4853144A05A099B0" }, invocation=null, revert=null, code=CALL_EXCEPTION, version=6.13.3)
    at AgentClient.pegin (/home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/src/agent-client.ts:42:19)
    at processTicksAndRejections (node:internal/process/task_queues:95:5)
    at async AgentClientApp.performPegIn (/home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/src/agent-client-app.ts:232:20)
    at async /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/bin/cli.ts:137:26
    at async executeWithSpinnerAndTiming (/home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/bin/cli.ts:14:5)
    at async Command.<anonymous> (/home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/bin/cli.ts:135:7)
✖ Perform PegIn transaction failed.
Error: Exception while processing PegIn on blockchain, Error: Error: execution reverted (unknown custom error) (action="estimateGas", data="0x835549470000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000f426c6f636b206e6f7420666f756e640000000000000000000000000000000000", reason=null, transaction={ "data": "0xb37e111027ace467dafd4ba039039b7f735f4a92d502adf1fd6b425d36f9bb60a177854b00000000000000000001b4b9f79123df985e564b11843c52e1262740f7cc732a26e5a2e2b8eaa5c78b03921d0e2770703cf3b177aed50b5d57e39d7c88ec5e04187623762b653972007eb8a76935f64f151b26794d7cfa37967f2885ff2b97b806e4dbe7c064789dead616707efda64da08f4b9a278513f155253d17c551ac121eacfc130128c3a3180328011045f0b8b8b09421bfb4f28b175dfbdae6b6fedc0f8a426f7694f3561e839ea42d2ef5d763b5f8e2c04cae71e77619725ca658bf17d0f96b7a48c584e8d9cd3b96ca4e997652d4e153115f9971b4e71dba5d2c6e2980e46b722bee854f99a338b5ebec0ee0c078744de1d3958d62158cc0cae3ed0d36d6878946e147a9eec1a9545f1e125adacbc375a6ca09df6fabe417207583", "from": "0xAAfD7D37B78E245aC85a230e161b512b0140daF7", "to": "0xB00dAB5C8b4b561cCdb7F43f4853144A05A099B0" }, invocation=null, revert=null, code=CALL_EXCEPTION, version=6.13.3)
    at AgentClient.pegin (/home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/src/agent-client.ts:42:19)
    at processTicksAndRejections (node:internal/process/task_queues:95:5)
    at async AgentClientApp.performPegIn (/home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/src/agent-client-app.ts:232:20)
    at async /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/bin/cli.ts:137:26
    at async executeWithSpinnerAndTiming (/home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/bin/cli.ts:14:5)
    at async Command.<anonymous> (/home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/bin/cli.ts:135:7)
Perform PegIn transaction: 1.371s
```

**ERROR:** `UIG Block not found`

When attempting to execute the PegIn transaction on the testnet, the Bitcoin oracle rejects it because the block containing the transaction is not included in the oracle’s records for the testnet. This prevents the transaction from being processed and ERC20 tokens from being minted.

## Conclusion

In this demo, we demonstrated the PegIn process from the Bitcoin testnet to the Merlin testnet blockchain. Assuming the agents had already co-created a pre-signed Bitcoin Taproot transaction and the BTC deposit to the Taproot address was completed, the requesting agent initiated the PegIn request, which was then confirmed by all agents. A zk-SNARK proof validated the transaction, finalizing the PegIn request and minting ERC20 tokens equivalent to the locked BTC.

We also tested and demonstrated the failure scenarios to ensure the security of the PegIn process. This includes cases like an invalid zk-proof, where a user attempts to use a proof for another PegIn, and a transaction for a block not included in the Bitcoin oracle, such as when the PegIn was confirmed on the mainnet but attempted on the testnet. These checks ensure that only valid PegIn transactions can be processed and that security is maintained throughout the process.
