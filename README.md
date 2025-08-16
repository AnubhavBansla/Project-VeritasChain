# Project-VeritasChain
#step 1
docker-compose.yaml
version: '3.7'

networks:
  veritasnet:

services:

  # --------------------- Orderer Service ---------------------
  orderer.example.com:
    container_name: orderer.example.com
    image: hyperledger/fabric-orderer:2.5
    environment:
      - FABRIC_LOGGING_SPEC=INFO
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      - ORDERER_GENERAL_TLS_ENABLED=true
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
      - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/:/var/hyperledger/orderer/tls
    ports:
      - 7050:7050
    networks:
      - veritasnet

  # --------------------- FarmerOrg Services ---------------------
  peer0.farmer.example.com:
    container_name: peer0.farmer.example.com
    image: hyperledger/fabric-peer:2.5
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=veritaschain_veritasnet
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      - CORE_PEER_ID=peer0.farmer.example.com
      - CORE_PEER_ADDRESS=peer0.farmer.example.com:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer0.farmer.example.com:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.farmer.example.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.farmer.example.com:7051
      - CORE_PEER_LOCALMSPID=FarmerOrgMSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/fabric/msp
      - CORE_PEER_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_PEER_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0:5984
      - CORE_PEER_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin
      - CORE_PEER_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=password
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/farmer.example.com/peers/peer0.farmer.example.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/farmer.example.com/peers/peer0.farmer.example.com/tls:/etc/hyperledger/fabric/tls
    ports:
      - 7051:7051
    depends_on:
      - couchdb0
    networks:
      - veritasnet

  couchdb0:
    container_name: couchdb0
    image: couchdb:3.1.1
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=password
    ports:
      - 5984:5984
    networks:
      - veritasnet

  # --------------------- ProcessorOrg Services ---------------------
  peer0.processor.example.com:
    container_name: peer0.processor.example.com
    image: hyperledger/fabric-peer:2.5
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=veritaschain_veritasnet
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      - CORE_PEER_ID=peer0.processor.example.com
      - CORE_PEER_ADDRESS=peer0.processor.example.com:9051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:9051
      - CORE_PEER_CHAINCODEADDRESS=peer0.processor.example.com:9052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:9052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.processor.example.com:9051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.processor.example.com:9051
      - CORE_PEER_LOCALMSPID=ProcessorOrgMSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/fabric/msp
      - CORE_PEER_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_PEER_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1:5984
      - CORE_PEER_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin
      - CORE_PEER_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=password
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/processor.example.com/peers/peer0.processor.example.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/processor.example.com/peers/peer0.processor.example.com/tls:/etc/hyperledger/fabric/tls
    ports:
      - 9051:9051
    depends_on:
      - couchdb1
    networks:
      - veritasnet

  couchdb1:
    container_name: couchdb1
    image: couchdb:3.1.1
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=password
    ports:
      - 6984:5984
    networks:
      - veritasnet

  # --------------------- RetailerOrg Services ---------------------
  peer0.retailer.example.com:
    container_name: peer0.retailer.example.com
    image: hyperledger/fabric-peer:2.5
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=veritaschain_veritasnet
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      - CORE_PEER_ID=peer0.retailer.example.com
      - CORE_PEER_ADDRESS=peer0.retailer.example.com:11051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:11051
      - CORE_PEER_CHAINCODEADDRESS=peer0.retailer.example.com:11052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:11052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.retailer.example.com:11051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.retailer.example.com:11051
      - CORE_PEER_LOCALMSPID=RetailerOrgMSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/fabric/msp
      - CORE_PEER_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_PEER_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb2:5984
      - CORE_PEER_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin
      - CORE_PEER_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=password
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/retailer.example.com/peers/peer0.retailer.example.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/retailer.example.com/peers/peer0.retailer.example.com/tls:/etc/hyperledger/fabric/tls
    ports:
      - 11051:11051
    depends_on:
      - couchdb2
    networks:
      - veritasnet

  couchdb2:
    container_name: couchdb2
    image: couchdb:3.1.1
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=password
    ports:
      - 7984:5984
    networks:
      - veritasnet

  # --------------------- CLI Service ---------------------
  cli:
    container_name: cli
    image: hyperledger/fabric-tools:2.5
    tty: true
    stdin_open: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - FABRIC_LOGGING_SPEC=INFO
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
      - /var/run/:/host/var/run/
      - ./chaincode/:/opt/gopath/src/github.com/chaincode
      - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
      - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - orderer.example.com
      - peer0.farmer.example.com
      - peer0.processor.example.com
      - peer0.retailer.example.com
    networks:
      - veritasnet
#step2 
# promyt 1
# ---------------------------------------------------------------------------
# "OrdererOrgs" - This section defines the orderer organization.
# The orderer manages the communication channel and consensus.
# ---------------------------------------------------------------------------
OrdererOrgs:
  - Name: OrdererOrg
    Domain: example.com
    # "Specs" defines the orderer nodes to be generated.
    Specs:
      - Hostname: orderer

# ---------------------------------------------------------------------------
# "PeerOrgs" - This section defines the peer organizations.
# Each peer organization represents a distinct entity in the network.
# ---------------------------------------------------------------------------
PeerOrgs:
  - Name: FarmerOrg
    Domain: farmerorg.example.com
    # "Template" defines the peers to be generated within this organization.
    Template:
      Count: 1
    # "Users" defines additional user identities to be generated
    # beyond the admin user.
    Users:
      Count: 1

  - Name: ProcessorOrg
    Domain: processororg.example.com
    Template:
      Count: 1
    Users:
      Count: 1

  - Name: RetailerOrg
    Domain: retailerorg.example.com
    Template:
      Count: 1
    Users:
      Count: 1
# prompt 2
# ---------------------------------------------------------------------------
# "Organizations" - Defines the organizations in the network
# ---------------------------------------------------------------------------
Organizations:
    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: crypto-config/ordererOrganizations/example.com/msp
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"

    - &FarmerOrg
        Name: FarmerOrg
        ID: FarmerMSP
        MSPDir: crypto-config/peerOrganizations/farmerorg.example.com/msp
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('FarmerMSP.admin', 'FarmerMSP.peer', 'FarmerMSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('FarmerMSP.admin', 'FarmerMSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('FarmerMSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('FarmerMSP.peer')"

    - &ProcessorOrg
        Name: ProcessorOrg
        ID: ProcessorMSP
        MSPDir: crypto-config/peerOrganizations/processororg.example.com/msp
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('ProcessorMSP.admin', 'ProcessorMSP.peer', 'ProcessorMSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('ProcessorMSP.admin', 'ProcessorMSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('ProcessorMSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('ProcessorMSP.peer')"

    - &RetailerOrg
        Name: RetailerOrg
        ID: RetailerMSP
        MSPDir: crypto-config/peerOrganizations/retailerorg.example.com/msp
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('RetailerMSP.admin', 'RetailerMSP.peer', 'RetailerMSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('RetailerMSP.admin', 'RetailerMSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('RetailerMSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('RetailerMSP.peer')"

# ---------------------------------------------------------------------------
# "Capabilities" - Defines the capabilities for various network components
# ---------------------------------------------------------------------------
Capabilities:
    Channel: &ChannelCapabilities
        V2_0: true
    Orderer: &OrdererCapabilities
        V2_0: true
    Application: &ApplicationCapabilities
        V2_0: true

# ---------------------------------------------------------------------------
# "ApplicationDefaults" - Defines the default application policies and capabilities
# ---------------------------------------------------------------------------
Application: &ApplicationDefaults
    Organizations:
    Policies:
        Readers:
            Type: Signature
            Rule: "OR('FarmerMSP.peer', 'ProcessorMSP.peer', 'RetailerMSP.peer')"
        Writers:
            Type: Signature
            Rule: "OR('FarmerMSP.client', 'ProcessorMSP.client', 'RetailerMSP.client')"
        Admins:
            Type: Signature
            Rule: "OR('FarmerMSP.admin', 'ProcessorMSP.admin', 'RetailerMSP.admin')"
        LifecycleEndorsement:
            Type: Signature
            Rule: "OR('FarmerMSP.peer', 'ProcessorMSP.peer', 'RetailerMSP.peer')"
        Endorsement:
            Type: Signature
            Rule: "OR('FarmerMSP.peer', 'ProcessorMSP.peer', 'RetailerMSP.peer')"
    Capabilities:
        <<: *ApplicationCapabilities

# ---------------------------------------------------------------------------
# "Profiles" - Defines various profiles for channel and genesis block creation
# ---------------------------------------------------------------------------
Profiles:
    # This profile is used to generate the system channel's genesis block.
    ThreeOrgsOrdererGenesis:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            ThreeOrgsConsortium:
                Organizations:
                    - *FarmerOrg
                    - *ProcessorOrg
                    - *RetailerOrg
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *FarmerOrg
                - *ProcessorOrg
                - *RetailerOrg
            Capabilities:
                <<: *ApplicationCapabilities

    # This profile is used to generate the application channel transaction.
    ThreeOrgsChannel:
        Consortium: ThreeOrgsConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *FarmerOrg
                - *ProcessorOrg
                - *RetailerOrg
            Capabilities:
                <<: *ApplicationCapabilities
#step 3
#!/bin/bash

# This script automates the setup of a Hyperledger Fabric network for a
# three-organization supply chain. It performs the following actions:
# 1. Removes old Docker containers and cryptographic material.
# 2. Generates new cryptographic assets using the cryptogen tool.
# 3. Creates the system channel's genesis block.
# 4. Creates the application channel transaction.
# 5. Creates anchor peer update transactions for each organization.
# 6. Starts the Docker containers.
# 7. Creates and joins the channel.
# 8. Sets the anchor peers for each organization.

# Stop and remove old containers, as well as remove old cryptographic material
function networkDown() {
    echo "Stopping and removing existing Docker containers..."
    docker-compose down --volumes --remove-orphans
    echo "Removing old cryptographic material and channel artifacts..."
    rm -rf crypto-config
    rm -rf channel-artifacts
    echo "Cleanup complete."
}

# --- Main Script Execution ---

# 1. Clean up old environment
networkDown

# Define channel and system channel names
export SYS_CHANNEL="syschannel"
export CHANNEL_NAME="veritaschannel"

# Set the path to the configtx.yaml file
export FABRIC_CFG_PATH=$PWD

# 2. Generate cryptographic material using cryptogen
echo "Generating cryptographic material..."
cryptogen generate --config=./crypto-config.yaml --output=./crypto-config

if [ $? -ne 0 ]; then
    echo "ERROR: cryptogen tool failed to generate crypto material."
    exit 1
fi
echo "Cryptographic material generation complete."

# 3. Generate the system channel genesis block
echo "Generating the system channel genesis block..."
mkdir channel-artifacts
configtxgen -profile ThreeOrgsOrdererGenesis -channelID $SYS_CHANNEL -outputBlock ./channel-artifacts/genesis.block
if [ $? -ne 0 ]; then
    echo "ERROR: configtxgen tool failed to generate genesis block."
    exit 1
fi
echo "Genesis block generation complete."

# 4. Generate the application channel transaction
echo "Generating the application channel transaction..."
configtxgen -profile ThreeOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
if [ $? -ne 0 ]; then
    echo "ERROR: configtxgen tool failed to generate channel transaction."
    exit 1
fi
echo "Channel transaction generation complete."

# 5. Generate anchor peer update transactions for each organization
echo "Generating anchor peer update transactions..."
configtxgen -profile ThreeOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/FarmerOrgAnchors.tx -channelID $CHANNEL_NAME -asOrg FarmerOrg
configtxgen -profile ThreeOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/ProcessorOrgAnchors.tx -channelID $CHANNEL_NAME -asOrg ProcessorOrg
configtxgen -profile ThreeOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/RetailerOrgAnchors.tx -channelID $CHANNEL_NAME -asOrg RetailerOrg
echo "Anchor peer updates generated."

# 6. Start the Docker containers
echo "Starting Docker containers..."
docker-compose -f docker-compose.yaml up -d
if [ $? -ne 0 ]; then
    echo "ERROR: Docker containers failed to start."
    exit 1
fi
echo "All Docker containers are up and running."

# Wait for containers to be ready
echo "Waiting for 10 seconds for containers to initialize..."
sleep 10

# The following commands will be executed inside the 'cli' container
# to interact with the network.

# 7. Create the channel on the orderer
echo "Creating channel '$CHANNEL_NAME'..."
docker exec cli bash -c 'peer channel create -o orderer.example.com:7050 -c veritaschannel -f /etc/hyperledger/configtx/channel.tx --tls --cafile /etc/hyperledger/crypto/orderer/tls/ca.pem'
if [ $? -ne 0 ]; then
    echo "ERROR: Failed to create channel."
    exit 1
fi
echo "Channel created successfully."

# 8. Join peers from each organization to the channel
echo "Joining peers to the channel..."

# Join FarmerOrg peer
docker exec -e CORE_PEER_LOCALMSPID="FarmerMSP" -e CORE_PEER_MSPCONFIGPATH="/etc/hyperledger/crypto/peers/farmerorg/users/Admin@farmerorg.example.com/msp" -e CORE_PEER_ADDRESS=peer0.farmerorg.example.com:7051 cli peer channel join -b /etc/hyperledger/channel-artifacts/veritaschannel.block
if [ $? -ne 0 ]; then
    echo "ERROR: FarmerOrg peer failed to join channel."
    exit 1
fi

# Join ProcessorOrg peer
docker exec -e CORE_PEER_LOCALMSPID="ProcessorMSP" -e CORE_PEER_MSPCONFIGPATH="/etc/hyperledger/crypto/peers/processororg/users/Admin@processororg.example.com/msp" -e CORE_PEER_ADDRESS=peer0.processororg.example.com:7051 cli peer channel join -b /etc/hyperledger/channel-artifacts/veritaschannel.block
if [ $? -ne 0 ]; then
    echo "ERROR: ProcessorOrg peer failed to join channel."
    exit 1
fi

# Join RetailerOrg peer
docker exec -e CORE_PEER_LOCALMSPID="RetailerMSP" -e CORE_PEER_MSPCONFIGPATH="/etc/hyperledger/crypto/peers/retailerorg/users/Admin@retailerorg.example.com/msp" -e CORE_PEER_ADDRESS=peer0.retailerorg.example.com:7051 cli peer channel join -b /etc/hyperledger/channel-artifacts/veritaschannel.block
if [ $? -ne 0 ]; then
    echo "ERROR: RetailerOrg peer failed to join channel."
    exit 1
fi
echo "All peers joined the channel successfully."

# 9. Set anchor peers for each organization
echo "Setting anchor peers for each organization..."

# Set FarmerOrg anchor peer
docker exec -e CORE_PEER_LOCALMSPID="FarmerMSP" -e CORE_PEER_MSPCONFIGPATH="/etc/hyperledger/crypto/peers/farmerorg/users/Admin@farmerorg.example.com/msp" cli peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f /etc/hyperledger/configtx/FarmerOrgAnchors.tx --tls --cafile /etc/hyperledger/crypto/orderer/tls/ca.pem
if [ $? -ne 0 ]; then
    echo "ERROR: Failed to set FarmerOrg anchor peer."
    exit 1
fi

# Set ProcessorOrg anchor peer
docker exec -e CORE_PEER_LOCALMSPID="ProcessorMSP" -e CORE_PEER_MSPCONFIGPATH="/etc/hyperledger/crypto/peers/processororg/users/Admin@processororg.example.com/msp" cli peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f /etc/hyperledger/configtx/ProcessorOrgAnchors.tx --tls --cafile /etc/hyperledger/crypto/orderer/tls/ca.pem
if [ $? -ne 0 ]; then
    echo "ERROR: Failed to set ProcessorOrg anchor peer."
    exit 1
fi

# Set RetailerOrg anchor peer
docker exec -e CORE_PEER_LOCALMSPID="RetailerMSP" -e CORE_PEER_MSPCONFIGPATH="/etc/hyperledger/crypto/peers/retailerorg/users/Admin@retailerorg.example.com/msp" cli peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f /etc/hyperledger/configtx/RetailerOrgAnchors.tx --tls --cafile /etc/hyperledger/crypto/orderer/tls/ca.pem
if [ $? -ne 0 ]; then
    echo "ERROR: Failed to set RetailerOrg anchor peer."
    exit 1
fi
echo "Anchor peers updated successfully."

echo "Network setup is complete! You can now access the CLI container to interact with the channel."

