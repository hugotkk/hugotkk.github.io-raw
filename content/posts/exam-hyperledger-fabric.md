---
title: "Hyperledger fabric"
date: 2022-07-18
tags:
- blockchain
---

## Notes

* installation - [curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.7 1.5.3](https://hyperledger-fabric.readthedocs.io/en/release-2.2/install.html#:~:text=curl%20%2DsSL%20https%3A//bit.ly/2ysbOFE%20%7C%20bash%20%2Ds%20%2D%2D%202.2.7%201.5.3)
* to start a basic test network (two org, 1 application channel)
  * `network.sh up createChannel -c mychannel -s couchdb` without fabric-ca
  * `network.sh up createChannel -ca -c mychannel -s couchdb` with fabric-ca
* `organizations/cryptogen/crypto-config-org3.yaml` + `cryptogen` will be used to bootstrap all required crypto stuff like certificates / tls / user identity for you - this is what I call "without fabric-ca"
* in the with fabric-ca way, you have to cretae a fabric-ca-server and register the peer, generate the user identity and certificates manually with `fabric-ca-client`
* `configtx/configtx.yaml` + `configtxgen` will be used to generate the genesis block, channel transaction and org definition for you
* peer environment variables setup - useful when performing channel update and chaincode installation / execution
  * [peer0.org1.example.com](https://hyperledger-fabric.readthedocs.io/en/release-2.2/channel_update_tutorial.html#fetch-the-configuration:~:text=export%20CORE_PEER_LOCALMSPID%3D%22Org1MSP%22)
  * [peer0.org2.example.com](https://hyperledger-fabric.readthedocs.io/en/release-2.2/channel_update_tutorial.html#fetch-the-configuration:~:text=export%20CORE_PEER_LOCALMSPID%3D%22Org2MSP%22)
  * [peer1.org3.example.com](https://hyperledger-fabric.readthedocs.io/en/release-2.2/channel_update_tutorial.html#fetch-the-configuration:~:text=once%0A%0Aexport%20CORE_PEER_TLS_ENABLED%3Dtrue-,export%20CORE_PEER_LOCALMSPID%3D%22Org3MSP%22,-export%20CORE_PEER_TLS_ROOTCERT_FILE%3D%24%7BPWD%7D/organizations)
* [tls setup for orderer, peer and fabric-ca](https://hyperledger-fabric.readthedocs.io/en/release-2.2/enable_tls.html)
* [configtx.yaml reference](https://hyperledger-fabric.readthedocs.io/en/release-2.2/create_channel/create_channel_config.html)

## Leader, Anchor (Gossip)
* [leader peer](https://hyperledger-fabric.readthedocs.io/en/release-2.2/channel_update_tutorial.html#configuring-leader-election)
    * it will always receive the block from orderer
    * it will then send the block to others peers
    * it can be set to keep the peer always up-to-date
    * it can be auto-elected or config statically
* [anchor peer](https://hyperledger-fabric.readthedocs.io/en/release-2.2/config_update.html#:~:text=%3A%20%22Admins%22%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22value%22%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22-,anchor_peers,-%22%3A%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%7B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%22host%22%3A%20%22peer0.org1)
    * The network uses the GOSSIP mechanism to know peers from other org with anchor peer. 
    * For example: org1 peer1 has to communicate with org2 peer1. it can know org2 peer1 from org1 anchor peer. 
    * an anchor peer can talk with another anchor peer from different org
    * to enable this in peer, define `CORE_PEER_GOSSIP_EXTERNALENDPOINT` to expose peer to the GOSSIP channel

## create channels

* orderer system channel
  * use [configtxgen -outputBlock](https://hyperledger-fabric.readthedocs.io/en/release-2.2/create_channel/create_channel.html#creating-an-application-channel:~:text=in%20your%20logs%3A-,configtxgen%20%2Dprofile,-TwoOrgsOrdererGenesis%20%2DchannelID) to generate genesis block
  * config `ORDERER_GENERAL_GENESISFILE` with genesis.block in orderer
* application channel
  * use [configtxgen -outputCreateChannelTx](https://hyperledger-fabric.readthedocs.io/en/release-2.2/create_channel/create_channel.html#creating-an-application-channel:~:text=for%20channel1%3A-,configtxgen%20%2Dprofile,-TwoOrgsChannel%20%2DoutputCreateChannelTx) to create transaction
  * create block with [peer channel create](https://hyperledger-fabric.readthedocs.io/en/release-2.2/create_channel/create_channel.html#creating-an-application-channel:~:text=the%20following%20command%3A-,peer%20channel%20create,-%2Do%20localhost) 
  * add peer to the channel with [peer channel join -b](https://hyperledger-fabric.readthedocs.io/en/release-2.2/create_channel/create_channel.html#creating-an-application-channel:~:text=the%20command%20below.-,peer%20channel%20join%20%2Db,-./channel%2D)

## add org3

* update `organizations/cryptogen/crypto-config-org3.yaml`
* [generate certificates for the org3 peer](https://hyperledger-fabric.readthedocs.io/en/release-2.2/channel_update_tutorial.html#generate-the-org3-crypto-material:~:text=../../bin/cryptogen%20generate)
* update configtx/config.yaml with org3 definition
* [print out the definition](https://hyperledger-fabric.readthedocs.io/en/release-2.2/channel_update_tutorial.html#generate-the-org3-crypto-material:~:text=../../bin/configtxgen%20%2DprintOrg)

* [update channel config](https://hyperledger-fabric.readthedocs.io/en/release-2.2/channel_update_tutorial.html#fetch-the-configuration:~:text=latest%20config%20block%3A-,peer%20channel%20fetch%20config,-channel%2Dartifacts)
  * fetch channel config
  * make a change request for adding new org definition to the channel config
  * sign the change request by org
  * commit the change
* [bring up new org's peer](https://hyperledger-fabric.readthedocs.io/en/release-2.2/channel_update_tutorial.html#bring-up-org3-components)
  
## add peer

* same as `add org` but 
* no need to update channel config

## add orderer

* same as `add org` but 
* need to update both `orderer` and `application` channel config (update the `Addresses` and `Consensers` list)
* bring up the new orderer with the latest orderer config block (don't use the original genesis.block)

## service discovery

- [configure external endpoints](https://hyperledger-fabric.readthedocs.io/en/release-2.2/discovery-cli.html#configuring-external-endpoints)
- add anchor peers to channel config 
- [setup persistent config](https://hyperledger-fabric.readthedocs.io/en/release-2.2/discovery-cli.html#persisting-configuration)
- [query peers](https://hyperledger-fabric.readthedocs.io/en/release-2.2/discovery-cli.html#peer-membership-query)
- [query channel config](https://hyperledger-fabric.readthedocs.io/en/release-2.2/discovery-cli.html#configuration-query)
- [query endorsers](https://hyperledger-fabric.readthedocs.io/en/release-2.2/discovery-cli.html#endorsers-query)
    - when the chaincode wasn't installed on the peer. it will return an error: chaincode definition wasn't found.

## chaincode

* use [peer lifecycle chaincode package](https://hyperledger-fabric.readthedocs.io/en/release-2.2/commands/peerlifecycle.html#peer-lifecycle-chaincode-package-example) to package the chaincode source code. 
* no need to call `npm install` / `go mod vendor` before packaging because the package command will exclude the dependencies folders (vendors / node_modules) for you
* use [peer lifecycle chaincode install](https://hyperledger-fabric.readthedocs.io/en/release-2.2/commands/peerlifecycle.html#peer-lifecycle-chaincode-install-example) to install the chaincode. a chaincode container will be started in each installed peers

To commit a chaincode, suggests to follow the Using Private Data in Fabric tutorial
* [peer lifecycle chaincode approveformyorg](https://hyperledger-fabric.readthedocs.io/en/release-2.2/private_data_tutorial.html#:~:text=in%20your%20terminal%3A-,peer%20lifecycle%20chaincode%20approveformyorg,-%2Do%20localhost%3A7050)
* [peer lifecycle chaincode commit](https://hyperledger-fabric.readthedocs.io/en/release-2.2/private_data_tutorial.html#:~:text=the%20collection%20definition.-,peer%20lifecycle%20chaincode%20commit,-%2Do%20localhost%3A7050)

It is a good example as it show how to add the extra params to the chaincode definition

For example,
* `--signature-policy`: define which peers need to endorse during the invoke
* `--collections-config`: config the private data definition
* `--init-required`: need to add `-I` when first time invoke the chaincode

To check which org has approved the chaincode, use
* [peer lifecycle chaincode checkcommitreadiness](https://hyperledger-fabric.readthedocs.io/en/release-2.2/commands/peerlifecycle.html#peer-lifecycle-chaincode-checkcommitreadiness-example)

Notice that those extra params have to be kept in the whole commit process. 

For example if you set `--init-required` in approveformyorg. you have to keep this in approveformyorg for other org, commit and checkcommitreadiness.

Some information need to know to commit a new version of chaincode:
* cc_package_id - [peer lifecycle chaincode queryinstalled](https://hyperledger-fabric.readthedocs.io/en/release-2.2/commands/peerlifecycle.html#peer-lifecycle-chaincode-queryinstalled-example)
* channel_name - `peer channel list`
* chaincode version and sequence - [peer lifecycle chaincode querycommitted](https://hyperledger-fabric.readthedocs.io/en/release-2.2/commands/peerlifecycle.html#peer-lifecycle-chaincode-querycommitted-example) or [peer lifecycle chaincode queryapproved](https://hyperledger-fabric.readthedocs.io/en/release-2.2/commands/peerlifecycle.html#peer-lifecycle-chaincode-queryapproved-example)

However, there is no way to know what extra parameters are used in approveformyorg by other orgs. 

It is hard to approve a chaincode without knowing the extra params being used in other org.

To execute an approved chaincode,
* use [peer chaincode invoke](https://hyperledger-fabric.readthedocs.io/en/release-2.2/private_data_tutorial.html#:~:text=20%2C%5C%22appraisedValue%5C%22%3A100%7D%22%20%7C%20base64%20%7C%20tr%20%2Dd%20%5C%5Cn\)-,peer%20chaincode%20invoke,-%2Do%20localhost%3A7050%20%2D%2DordererTLSHostnameOverride%20orderer.example.com)
* `--waitForEvent` and `peerAddresses` are needed in `peer chaincode invoke`
* `--transient` can be used to pass an object to the chaincode
* `--tls` means use tls to connect to the orderer but not peers
* if tls is enabled in peers, you must connect peer with tls by `export CORE_PEER_TLS_ENABLED=true`

To define which / how many org are required in `peer lifecycle chaincode commit` 
* change "LifecycleEndorsement" policy in channel config

To define which / how many org are required to endorse in `peer chaincode invoke`
* change "Endorsements" policy in channel config
* use `--signature-policy` to approve and commit the chaincode. this will override the channel config

## fabric-ca-client

* fabric-ca-client will look at the `fabric-ca-client-config.yml` and msp (user identity) at the `$FABRIC_CA_CLIENT_HOME` folder. 
* by setting up this environment variable, you could run commands with that user identity directly without enrollment
* `fabric-ca-client enroll` will generate the `fabric-ca-client-config.yml` and msp (user identity) at the `$FABRIC_CA_CLIENT_HOME` folder. 
* you have to specify `-M` when enrolling a user so you won't override the user identity config.
* everytime when you call `fabric-ca-client enroll`. a new pair of cert and key will be generated in the `$FABRIC_CA_CLIENT_HOME/msp` folder
* [Using Private Data in Fabric](https://hyperledger-fabric.readthedocs.io/en/release-2.2/private_data_tutorial.html#:~:text=export%20FABRIC_CA_CLIENT_HOME%3D%24%7BPWD%7D/organizations/peerOrganizations/org1.example.com/) has a good example to show how to create a user in fabric-ca
  * login as org1.example.com admin (`$FABRIC_CA_CLIENT_HOME/msp`)
    ```
    export FABRIC_CA_CLIENT_HOME=${PWD}/organizations/peerOrganizations/org1.example.com/
    ```
  * create new user `owner`
    ```
    fabric-ca-client register --caname ca-org1 --id.name owner --id.secret ownerpw --id.type client --tls.certfiles "${PWD}/organizations/fabric-ca/org1/tls-cert.pem"
    ```
  * generate owner's certificate in `${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp`
    ```
    fabric-ca-client enroll -u https://owner:ownerpw@localhost:7054 --caname ca-org1 -M "${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp" --tls.certfiles "${PWD}/organizations/fabric-ca/org1/tls-cert.pem"
    ```
  * if remove -M, `$FABRIC_CA_CLIENT_HOME/msp` will be overridden so the user identity will change to `owner`. If you don't have any backup for the msp folder, you have to enroll in an admin account to re-generate the config again.
    ```
    fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-org1 --tls.certfiles "${PWD}/organizations/fabric-ca/org1/tls-cert.pem"
    ```

* [Fabric CA User's Guide](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#revoking-a-certificate-or-identity) recorded many useful examples for daily operations

  * create a user with type: client - [fabric-ca-client register](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#enrolling-a-peer-identity:~:text=certificate%20by%20default.-,fabric%2Dca%2Dclient%20register,-%2D%2Did.)
  * login & generate the certificate - [fabric-ca-client enroll](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#enrolling-a-peer-identity:~:text=fabric%2Dca%2Dclient%20enroll%20%2Du%20http%3A//peer1%3Apeer1pw%40localhost%3A7054%20%2DM%20%24FABRIC_CA_CLIENT_HOME/msp)
  * list out the certs in ca - [fabric-ca-client certificate list](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#revoking-a-certificate-or-identity:~:text=List%20all%20certificates%3A-,fabric%2Dca%2Dclient%20certificate%20list,-List%20all%20certificates)
  * set user's affiliation - [--affiliation org2](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#revoking-a-certificate-or-identity:~:text=identity%20modify%20user1-,%2D%2Daffiliation,-org2) 
  * set user as revoker of peer and client - [--id.attrs '"hf.Registrar.Roles=peer,client",hf.Revoker=true'](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#revoking-a-certificate-or-identity:~:text=%2D%2Did.attrs%20%27%22hf.Registrar.Roles%3Dpeer%2Cclient%22%2Chf.Revoker%3Dtrue%27)
  * revoke a user identity - [fabric-ca-client revoke -e](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#revoking-a-certificate-or-identity:~:text=will%20be%20rejected.-,fabric%2Dca%2Dclient%20revoke%20%2De,-%3Cenrollment_id%3E)
  * update the crl - [fabric-ca-client gencrl](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#revoking-a-certificate-or-identity:~:text=fabric%2Dca%2Dclient%20revoke%20%2De%20peer1%20%2D%2Dgencrl)
  * renew the cert - [fabric-ca-client reenroll](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#reenrolling-an-identity)

## fabric-ca-server

* use `fabric-ca-server init -b` to generate the dummy server config. They will be stored in `$FABRIC_CA_HOME`
* update `csr.cn`, `csr.names`, `csr.hosts`, `ca.name`, `tls.enabled` in `$FABRIC_CA_HOME/fabric-ca-server-config.yaml`
* before starting the server, make sure to unset all conflict fabric-ca-server environment variables. They will override the setting in config.yaml. I spent lots of time debugging because of this.
* `fabric-ca-server start -b` to start the ca server
* enable hsm
  * [this page tells how to enable hsm in fabric-ca-server](https://hyperledger-fabric.readthedocs.io/en/release-2.2/hsm.html#configuring-an-hsm). 
  * basically, we need the `libsofthsm2.so` `pin` and `label` to config the `bccsp` session in `fabric-ca-server-config.yaml`. 
  * then the private keys will be stored in hsm instead of the msp folder
  * however they don't provide any fabric-ca binary with pscs11 enabled
  * to play with fabric-ca-server with hsm. we have to compile [softhsm2](https://github.com/opendnssec/SoftHSMv2) and [fabric-ca](https://github.com/hyperledger/fabric-ca) by ourselves

* enable mysql
  * by default, the user identities will be stored in a sqlite file
  * if you plan to create a ca in cluster - build multiple ca servers and use haproxy to load balancer them - you need a mysql server to store the user identities globally (among the cluster)
  * the config can be found in `fabric-ca-server-config.yaml` `db:type`
  * you may need to `set sql_mode=''` in the mysql server to fix the incapability

* intermediate ca
  * start ca with `fabric-ca-server start -b admin:adminpw -u http://<enrollmentID>:<secret>@<parentserver>:<parentport>`
  * to enroll a peer with intermediate ca. you need to concat root ca + intermediate certs for the org definition and msp (user identity)

## Note for exam

* [remote desktop environment](https://itnext.io/cks-cka-ckad-changed-terminal-to-remote-desktop-157a26c1d5e)
  * XFCE 4.14
  * Guacamole 1.4.0
  * XFCE Terminal Emulator (black background, white font)
  * Ubuntu 20.04
  * Firefox Browser
* [PSI secure browser interface](https://docs.linuxfoundation.org/tc-docs/certification/lf-handbook2/exam-user-interface#lf_remote_desktop69fKd8/s/-M5QaeeC1mG9VndIpgJe/certification/lf-handbook2/exam-user-interface#lf_remote_desktop)
* [retake policy](https://training.linuxfoundation.org/about/policies/certification-exam-retake-policy/) - one free retake per Exam purchase
* [Update on Certification Exam Proctoring Migration](https://training.linuxfoundation.org/blog/update-on-certification-exam-proctoring-migration/) mentioned a few important things about the exam:
    * no personal bookmarks anymore - it's very stupid...
    * copying and pasting the yaml in vim will cause incorrect indentation - fix it by `:set paste`!
    * copy and paste from the terminal will be `Copy = CTRL+SHIFT+C`, `Paste = CTRL+SHIFT+V` for Paste - you may need to get used to it
* External monitor: only 1 active monitor is allowed - If you are using macbook, you will see 2 monitors in "About This Macs > Displays". There is no way for me to disable the built-in monitor unless I close the lid. So I can't use the monitor during the exam.
