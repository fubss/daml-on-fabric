## Endorsement-policy changes testing

The channel contains two Orgs. Every Org has one peer.

### 1. Presetting Fabric network endorsement policy

You can change the endorsement policy for daml-on-fabric chaincode on the line 91-92 in the file `daml-on-fabric/config-local.yaml`. Default endorsement policy is "soft" - it takes one voice of any Org. (Default endorsement policy  is taken from `endorsement-policy.yaml`) Endorsement policy from `my-endorsement-policy.yaml` is "strict" - it takes voices from both Orgs.

### 2. Running the network

Do what they do in the [Readme](https://github.com/digital-asset/daml-on-fabric#running-java-quick-start-against-daml-on-fabric) untill **Step 6**.

### 3. Connecting to peers

Connect to docker containers peer0.Org1, peer0.Org2 in two new different terminals.
You can do it with this command:
```
docker exec -it CONTAINER_ID sh
```
You can get CONTAINER_ID from `docker ps` command.

After joining to peer you can check chaincode definition using command:
```
peer lifecycle chaincode querycommitted -o orderer.example.com:7050 --tls --cafile /etc/hyperledger/msp/tlsca.example.com-cert.pem --channelID mainchannel --name daml_on_fabric --output json
```

### 4. Changing endorsement policy

First, find and set PACKAGE_ID for commands below. You can get PACKAGE_ID from peers' logs.

#### 4.1 on Org2 peer

On the peer0.Org2 run this command to approve "strict" endorsement policy for this Org:
```
peer lifecycle chaincode approveformyorg --tls --cafile /etc/hyperledger/msp/tlsca.example.com-cert.pem --channelID mainchannel --name daml_on_fabric --version 1 --sequence 2 --waitForEvent --package-id daml_on_fabric:PACKAGE_ID --signature-policy "AND ('Org1MSP.member','Org2MSP.member')"
```
or to approve "soft" endorsement policy you can run this (the difference is the command after the `--signature-policy` flag):
```
peer lifecycle chaincode approveformyorg --tls --cafile /etc/hyperledger/msp/tlsca.example.com-cert.pem --channelID mainchannel --name daml_on_fabric --version 1 --sequence 2 --waitForEvent --package-id daml_on_fabric:PACKAGE_ID --signature-policy "OR ('Org1MSP.member','Org2MSP.member')"
```

#### 4.2 on Org1 peer

On the peer0.Org1 run the same command, which you ran on the peer0.Org2.

After approving for Org1 commit new endorsement policy running this command for setting "strict" endorsement policy:
```
peer lifecycle chaincode commit -o orderer.example.com:7050 --tls --cafile /etc/hyperledger/msp/tlsca.example.com-cert.pem --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /etc/hyperledger/msp/peer/tls/ca.crt --channelID mainchannel --name daml_on_fabric --version 1 --sequence 2 --signature-policy "AND ('Org1MSP.member','Org2MSP.member')"

```
or to commit "soft" endorsement policy you can run this:
```
peer lifecycle chaincode commit -o orderer.example.com:7050 --tls --cafile /etc/hyperledger/msp/tlsca.example.com-cert.pem --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /etc/hyperledger/msp/peer/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /etc/hyperledger/msp/ca.crt --channelID mainchannel --name daml_on_fabric --version 1 --sequence 2 --signature-policy "OR ('Org1MSP.member','Org2MSP.member')"
```
