[//]: # (SPDX-License-Identifier: CC-BY-4.0)

## Hyperledger Fabric Samples

Please visit the [installation instructions](http://hyperledger-fabric.readthedocs.io/en/latest/install.html)
to ensure you have the correct prerequisites installed. Please use the
version of the documentation that matches the version of the software you
intend to use to ensure alignment.

## Download Binaries and Docker Images

The installation instructions will utilize `scripts/bootstrap.sh` (available in the fabric repository)
script to download all of the requisite Hyperledger Fabric binaries and docker
images, and tag the images with the 'latest' tag. Optionally,
specify a version for fabric, fabric-ca and thirdparty images. If versions
are not passed, the latest available versions will be downloaded.

The script will also clone fabric-samples repository using the version tag that
is aligned with the Fabric version.

You can also download the script and execute locally:

```bash
# Fetch bootstrap.sh from fabric repository using
curl -sS https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh -o ./scripts/bootstrap.sh
# Change file mode to executable
chmod +x ./scripts/bootstrap.sh
# Download binaries and docker images
./scripts/bootstrap.sh [version] [ca version] [thirdparty_version]
```

# To interact with test network, Start here
## Bring up the test network
1. Navigate to test network directory by using the following command
```
cd fabric-samples/test-network
```
2. Run `./network.sh -h` to print the script help text
3. Run the following command to remove any containers or artifacts from any previous runs: `./network.sh up`
4. Then we can bring up the network by issuing the following command: `./network.sh up`
This command creates a Fabric network that consists of two peer nodes, one ordering node.

## Components of the test network
> Run the following command to list all of Docker containers that are running on your machine: `docker ps -a`

Each node and user that interacts with a Fabric network needs to belong to an organization that is a network member. The group of organizations that are members of a Fabric network are often referred to as the consortium. The test network has two consortium members, Org1 and Org2. The network also includes one orderer organization that maintains the ordering service of the network.

Peers are the fundamental components of any Fabric network. Peers store the blockchain ledger and validate transactions before they are committed to the ledger. Peers run the smart contracts that contain the business logic that is used to manage the assets on the blockchain ledger.

Every peer in the network needs to belong to a member of the consortium. In the test network, each organization operates one peer each, peer0.org1.example.com and peer0.org2.example.com.

Every Fabric network also includes an ordering service. While peers validate transactions and add blocks of transactions to the blockchain ledger, they do not decide on the order of transactions or include them into new blocks. On a distributed network, peers may be running far away from each other and not have a common view of when a transaction was created. Coming to consensus on the order of transactions is a costly process that would create overhead for the peers.

An ordering service allows peers to focus on validating transactions and committing them to the ledger. After ordering nodes receive endorsed transactions from clients, they come to consensus on the order of transactions and then add them to blocks. The blocks are then distributed to peer nodes, which add the blocks the blockchain ledger. Ordering nodes also operate the system channel that defines the capabilities of a Fabric network, such as how blocks are made and which version of Fabric that nodes can use. The system channel defines which organizations are members of the consortium.

The sample network uses a single node Raft ordering service that is operated by the ordering organization. You can see the ordering node running on your machine as orderer.example.com. While the test network only uses a single node ordering service, a real network would have multiple ordering nodes, operated by one or multiple orderer organizations. The different ordering nodes would use the Raft consensus algorithm to come to agreement on the order of transactions across the network.

## Creating a channel
> Run the following command to create a channel with the default name of mychannel: `./network.sh createChannel`

Now that we have peer and orderer nodes running on our machine, we can use the script to create a Fabric channel for transactions between Org1 and Org2. Channels are a private layer of communication between specific network members. Channels can be used only by organizations that are invited to the channel, and are invisible to other members of the network. Each channel has a separate blockchain ledger. Organizations that have been invited “join” their peers to the channel to store the channel ledger and validate the transactions on the channel.

Can also use the channel flag to create a channel with custom name. As an example, the following command would create a channel named channel1: `./network.sh createChannel -c channel1`

## Starting a chaincode on the channel
Start a chaincode on the channel using the following command: `./network.sh deployCC`

The deployCC subcommand will install the fabcar chaincode on peer0.org1.example.com and peer0.org2.example.com and then deploy the chaincode on the channel specified using the channel flag (or mychannel if no channel is specified). If you are deploying a chaincode for the first time, the script will install the chaincode dependencies. By default, The script installs the Go version of the fabcar chaincode. However, you can use the language flag, -l, to install the Java or javascript versions of the chaincode. You can find the Fabcar chaincode in the chaincode folder of the fabric-samples directory.

After the fabcar chaincode definition has been committed to the channel, the script initializes the chaincode by invoking the init function and then invokes the chaincode to put an initial list of cars on the ledger. The script then queries the chaincode to verify the that the data was added.

After you have created a channel, you can start using smart contracts to interact with the channel ledger. Smart contracts contain the business logic that governs assets on the blockchain ledger. Applications run by members of the network can invoke smart contracts to create assets on the ledger, as well as change and transfer those assets. Applications also query smart contracts to read data on the ledger.

To ensure that transactions are valid, transactions created using smart contracts typically need to be signed by multiple organizations to be committed to the channel ledger. Multiple signatures are integral to the trust model of Fabric. Requiring multiple endorsements for a transaction prevents one organization on a channel from tampering with the ledger on their peer or using business logic that was not agreed to. To sign a transaction, each organization needs to invoke and execute the smart contract on their peer, which then signs the output of the transaction. If the output is consistent and has been signed by enough organizations, the transaction can be committed to the ledger. The policy that specifies the set organizations on the channel that need to execute the smart contract is referred to as the endorsement policy, which is set for each chaincode as part of the chaincode definition.

In Fabric, smart contracts are deployed on the network in packages referred to as chaincode. A Chaincode is installed on the peers of an organization and then deployed to a channel, where it can then be used to endorse transactions and interact with the blockchain ledger. Before a chaincode can be deployed to a channel, the members of the channel need to agree on a chaincode definition that establishes chaincode governance. When the required number of organizations agree, the chaincode definition can be committed to the channel, and the chaincode is ready to be used.

## Interacting with the network
After you bring up the test network, you can use the peer CLI to interact with your network. The peer CLI allows you to invoke deployed smart contracts, update channels, or install and deploy new smart contracts from the CLI.

Make sure that you are operating from the test-network directory. If you followed the instructions to install the Samples, Binaries and Docker Images, You can find the peer binaries in the bin folder of the fabric-samples repository. Use the following command to add those binaries to your CLI Path:
```
export PATH=${PWD}/../bin:${PWD}:$PATH
```
You also need to set the FABRIC_CFG_PATH to point to the core.yaml file in the fabric-samples repository:
```
export FABRIC_CFG_PATH=$PWD/../config/
```
You can now set the environment variables that allow you to operate the peer CLI as Org1:
```
# Environment variables for Org1

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```
The `CORE_PEER_TLS_ROOTCERT_FILE` and `CORE_PEER_MSPCONFIGPATH` environment variables point to the Org1 crypto material in the `organizations` folder.

If you used `./network.sh deployCC` to install and start the fabcar chaincode, you can now query the ledger from your CLI. Run the following command to get the list of cars that were added to your channel ledger:
```
peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'
```

If the command is successful, you can see the same list of cars that were printed in the logs when you ran the script:
```
[{"Key":"CAR0", "Record":{"make":"Toyota","model":"Prius","colour":"blue","owner":"Tomoko"}},
{"Key":"CAR1", "Record":{"make":"Ford","model":"Mustang","colour":"red","owner":"Brad"}},
{"Key":"CAR2", "Record":{"make":"Hyundai","model":"Tucson","colour":"green","owner":"Jin Soo"}},
{"Key":"CAR3", "Record":{"make":"Volkswagen","model":"Passat","colour":"yellow","owner":"Max"}},
{"Key":"CAR4", "Record":{"make":"Tesla","model":"S","colour":"black","owner":"Adriana"}},
{"Key":"CAR5", "Record":{"make":"Peugeot","model":"205","colour":"purple","owner":"Michel"}},
{"Key":"CAR6", "Record":{"make":"Chery","model":"S22L","colour":"white","owner":"Aarav"}},
{"Key":"CAR7", "Record":{"make":"Fiat","model":"Punto","colour":"violet","owner":"Pari"}},
{"Key":"CAR8", "Record":{"make":"Tata","model":"Nano","colour":"indigo","owner":"Valeria"}},
{"Key":"CAR9", "Record":{"make":"Holden","model":"Barina","colour":"brown","owner":"Shotaro"}}]
```
Chaincodes are invoked when a network member wants to transfer or change an asset on the ledger. Use the following command to change the owner of a car on the ledger by invoking the fabcar chaincode:
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls true --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n fabcar --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"changeCarOwner","Args":["CAR9","Dave"]}'
```
If the command is successful, you should see the following response:
```
2019-12-04 17:38:21.048 EST [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
```
Note: If you deployed the Java chaincode, run the invoke command with the following arguments instead: `'{"function":"changeCarOwner","Args":["CAR009","Dave"]}'` The Fabcar chaincode written in Java uses a different index than the chaincode written in Javascipt or Go.

Because the endorsement policy for the fabcar chaincode requires the transaction to be signed by Org1 and Org2, the chaincode invoke command needs to target both `peer0.org1.example.com` and `peer0.org2.example.com` using the `--peerAddresses` flag. Because TLS is enabled for the network, the command also needs to reference the TLS certificate for each peer using the `--tlsRootCertFiles` flag.

After we invoke the chaincode, we can use another query to see how the invoke changed the assets on the blockchain ledger. Since we already queried the Org1 peer, we can take this opportunity to query the chaincode running on the Org2 peer. Set the following environment variables to operate as Org2:
```
# Environment variables for Org2

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```
You can now query the fabcar chaincode running on `peer0.org2.example.com`:
```
peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryCar","CAR9"]}'
```
The result will show that "CAR9" was transferred to Dave:
```
Tou can find he {"make":"Holden","model":"Barina","colour":"brown","owner":"Dave"}
```

## Bring down the network
When you are finished using the test network, you can bring down the network with the following command:
```
./network.sh down
```
The command will stop and remove the node and chaincode containers, delete the organization crypto material, and remove the chaincode images from your Docker Registry. The command also removes the channel artifacts and docker volumes from previous runs, allowing you to run `./network.sh up` again if you encountered any problems.

## License <a name="license"></a>

Hyperledger Project source code files are made available under the Apache
License, Version 2.0 (Apache-2.0), located in the [LICENSE](LICENSE) file.
Hyperledger Project documentation files are made available under the Creative
Commons Attribution 4.0 International License (CC-BY-4.0), available at http://creativecommons.org/licenses/by/4.0/.
