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
  Executed AgentRegistryModule#AgentRegistry
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

AgentRegistryModule#AgentRegistry - 0xd3E3f2eD5fe3B0F3c25b0AdE87b1199E067F6487
BitcoinOracleModule#BitcoinOracle - 0xa45C6D8E183CD42CD88ffb0f3B9b5A0BAACE6C15
ContractRegistryModule#ContractRegistry - 0x285F58289e8B7c55F7133Eea03E68aC314D2dd50
PeginVerifierModule#PeginVerifier - 0xDBDaFb8a288F7e0F956146b0179f39b7aBC850c3
GrailBridgeDeployment#GrailBridge - 0xb4a29738862b26F537F1c2fc8A3706aD5606b58a
GrailBridgeDeployment#ERC20BitSnark - 0xc74B4EB3De0d103AABbCa92D98a3653C99420e65
```

### Block Headers Submission

```bash
$ npx hardhat oracle-submit-headers --file ./scripts/block-headers-merlin.json --network merlinTestnet
BitcoinOracle contract at address: 0xa45C6D8E183CD42CD88ffb0f3B9b5A0BAACE6C15
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
GRAIL_BRIDGE_ADDRESS=0xb4a29738862b26F537F1c2fc8A3706aD5606b58a
ERC20_ADDRESS=0xc74B4EB3De0d103AABbCa92D98a3653C99420e65
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
  "grail_bridge_address": "0xb4a29738862b26F537F1c2fc8A3706aD5606b58a",
  "erc20_address": "0xc74B4EB3De0d103AABbCa92D98a3653C99420e65",
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
⠼ Create PegIn request started...
PegIn request created successfully.
Request ID: 0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1
✔ Create PegIn request completed successfully.
Create PegIn request: 9.177s

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
Fetch and Parse transaction data from blockstream: 12.255ms

============================================================
Generating PegIn proof inputs file
============================================================
$ ./cli.sh generate-pegin-proof-inputs "0x00000000040afd9372ae26485bdaee67b391579bdd40663d5ca21636d0beabb0" "0xed470838b312d5984304b8ca4734e223235d115cfa5e0a368b907287732110b8" "0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1" "0"
⠋ Generate PegIn proof inputs file started...
PegIn inputs file generated successfully.
File path: /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1_inputs.json
✔ Generate PegIn proof inputs file completed successfully.
Generate PegIn proof inputs file: 24.658ms

============================================================
Generating PegIn proof file
============================================================
$ ./cli.sh generate-pegin-proof "0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1"
⠏ Generate PegIn proof started...
Proof file path: /home/elmol/Documents/wakeup/bitsnark/grail-contracts/packages/agent-cli/data/merlin-demo/0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1_proof.json
✔ Generate PegIn proof completed successfully.
Generate PegIn proof: 1:00.828 (m:ss.mmm)

============================================================
Performing the PegIn transaction on the blockchain
============================================================
$ ./cli.sh perform-pegin "0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1" "0x00000000040afd9372ae26485bdaee67b391579bdd40663d5ca21636d0beabb0"
⠹ Perform PegIn transaction started...
PegIn transaction performed successfully.
Result: 0xf37cbba1f066ec233d3070728cd894cc466263a79045b8f9ca198bea8b81d0a1,20000,0xAAfD7D37B78E245aC85a230e161b512b0140daF7,2,2
✔ Perform PegIn transaction completed successfully.
Perform PegIn transaction: 10.608s

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
[merlin-demo-execution.webm](https://github.com/user-attachments/assets/ddae1b23-519f-4a76-9c4c-03c2255af22b)


### Balance ERC20 check

Here, we can see that the balance has been updated to reflect 0.0002 BTC:

```
$ ./cli.sh show-balance
Account Address: 0xAAfD7D37B78E245aC85a230e161b512b0140daF7
ERC20 Balance: 0.0002 BTC
```

### Pegin Transaction

You can view the PegIn transaction on the Merlin chain explorer:

https://testnet-scan.merlinchain.io/tx/0x3603358a68ffc7f511f65a160428aa3e572ffa41a83c5fd15d91972184da5eab


## Conclusion

In this demo, we demonstrated the PegIn process from the Bitcoin testnet to the Merlin testnet blockchain. Assuming the agents had already co-created a pre-signed Bitcoin Taproot transaction and the BTC deposit to the Taproot address was completed, the requesting agent initiated the PegIn request, which was then confirmed by all agents. A zk-SNARK proof validated the transaction, finalizing the PegIn request and minting ERC20 tokens equivalent to the locked BTC.
