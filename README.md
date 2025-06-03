# dWeb Framework: Decentralized Web Communication

<p align="center">
  <img src="webs/desk/public/images/dweb.png" alt="dWeb Logo" width="96"/>
</p>

**Author:** Max  
**Contact:** [unitmax@proton.me](mailto:unitmax@proton.me)  

---

## What is dWeb?

dWeb is a framework that enables decentralized web applications through a multi-cluster architecture. It achieves ~500 transactions per second while maintaining complete privacy of messages through browser-level encryption. Built entirely in JavaScript for maximum developer accessibility.

### Key Features

- **High Performance:** ~500 transactions per second, outperforming traditional blockchains
- **End-to-End Encryption:** All communications are encrypted in the browser before reaching any server
- **Energy Efficient:** Powered by Delegated Proof of Stake (DPoS), eliminating wasteful mining
- **Multi-Cluster Architecture:** Independent, specialized service clusters working together seamlessly
- **JavaScript-Based:** Built for web developers, with no specialized languages to learn
- **Modular Design:** Easy to extend with new services and custom implementations

---

## System Architecture

dWeb operates on three primary components:

### 1. Clusters

Self-governing units that provide specialized decentralized services. Each cluster:
- Operates under its own consensus model (typically DPoS)
- Defines its own trusted peers for cross-cluster communication
- Can vary in size from small local networks to large distributed systems

### 2. Nodes

The infrastructure participants of the network. Each node:
- Has its own private key for secure identification
- Can participate in multiple clusters simultaneously
- Relays messages between clusters to maintain system resilience

### 3. Users

End-users of the decentralized applications. Each user:
- Has a private key for secure authentication
- Can interact with multiple clusters and services
- Maintains control of their data through client-side encryption

### Cross-Cluster Communication

Messages between clusters require 67% consensus from trusted peers of the originating cluster, ensuring security while enabling specialized services to work together.

---

## Getting Started

### Prerequisites

- Node.js (v14 or higher)
- npm (v6 or higher)

### Installation

1. **Clone the repository**

```bash
git clone github.com/maximilianco/dWeb-JS-blockchain
cd dWeb-JS-blockchain
```

2. **Install dependencies**

```bash
npm install
```

3. **Start the dWeb node**

```bash
node dweb.js
```

4. **Access the user interface**

Open your browser and navigate to:
```
http://127.0.0.1:1225/desk/
```

---

## Creating a Decentralized Service

One of dWeb's strengths is how easily developers can create new decentralized services. Below is a practical example of implementing a chat service.

### Example: Building a Chat Service

Creating a decentralized service in dWeb involves five main components:

1. **Network Extension** - Extending the base Network class
2. **Instruction Definition** - Defining the service's operations
3. **Validation** - Ensuring data integrity
4. **Processing** - Handling the business logic
5. **Configuration** - Setting up both system-wide and service-specific configs

#### 1. Create the Chat Network (chat.js)

First, extend the base Network class to create your specialized service:

```javascript
const Network = require('../../core/network/network.js');
const RPCMessageHandler = require('./network/rpc-message-handler.js');
const ChatMSGInstruction = require('./core/instructions/chatmsg/chatmsg.js');

class Chat extends Network {
    constructor(config) {
        super(config);
    }

    async initialize(node) {
        await super.initialize(node);
        this.actionManager.registerInstructionType('chatmsg', new ChatMSGInstruction(this));
    }

    Start(node) {
        super.Start(node);
        node.AddRPCMessageHandler(new RPCMessageHandler(this));
    }
}

module.exports = Chat;
```

#### 2. Define the Chat Message Instruction (chatmsg.js)

Create an instruction that defines what a chat message is and how it should be handled:

```javascript
const IInstruction = require('../../../../../core/system/interfaces/iinstruction.js');
const ChatMSGInstructionValidator = require('./validator.js');
const ChatMSGInstructionProcessor = require('./processor.js');
const PercentageFee = require('../../../../../core/system/instruction/fees/percentagefee.js');

class ChatMSGInstruction extends IInstruction {
    constructor(network) {
        super(network);
        this.validator = new ChatMSGInstructionValidator(network);
        this.processor = new ChatMSGInstructionProcessor(network, this.validator);
        
        // Set up percentage fee handler
        this.setFeeHandler(new PercentageFee(network));
    }

    async createInstruction(params) {
        const instruction = {
            type: 'chatmsg',
            toAccount: params.toAccount,
            amount: params.amount,
            message: params.message
        };
        
        return instruction;
    }

    async validateInstruction(validationData) {
        return await this.validator.validateInstruction(validationData);
    }

    async processInstruction(processData) {
        return await this.processor.processInstruction(processData);
    }
}

module.exports = ChatMSGInstruction;
```

#### 3. Create a Validator (validator.js)

Define validation rules to ensure message integrity:

```javascript
const BaseInstructionValidator = require('../../../../../core/system/instruction/base/baseinstructionvalidator');

class ChatMSGInstructionValidator extends BaseInstructionValidator {
    constructor(network) {
        super(network);

        this.addInstructionProperties({
            type: { type: 'string', enum: ['chatmsg'] },
            message: { type: 'string' }
        }, [
            'type',
            'message'
        ]);
    }
}

module.exports = ChatMSGInstructionValidator;
```

#### 4. Implement a Processor (processor.js)

Create the business logic for processing chat messages:

```javascript
const BaseInstructionProcessor = require('../../../../../core/system/instruction/base/baseinstructionprocessor');

class ChatMSGInstructionProcessor extends BaseInstructionProcessor {
    constructor(network) {
        super(network);
    }
    
    // Add custom processing logic here
}

module.exports = ChatMSGInstructionProcessor;
```

#### 5. Configure Your Service

dWeb requires two configuration files to properly set up a service:

##### Main System Configuration (config/config.js)

Add your service to the list of enabled networks in the main configuration:

```javascript
module.exports = {
    "enabledNetworks": ['desk', 'chat', 'finance', 'exchange', 'thumbnail', 'search', 'call', 'email', 'file', 'social', 'governance', 'name'],
    "logLevel": 'debug',
    "useSSL": true
};
```

##### Service-Specific Configuration (webs/chat/config/config.js)

Create a configuration file for your service with network-specific settings:

```javascript
module.exports = {
    "networks": {
        "chat-testnet": {
            "peerPort": 1224,
            "rpcPort": 1225,
            "subscriptionPort": 1226,
            "peers": [
                "185.196.8.90:1224"
            ],
            "dbPath": "data/chat-testnet"
            // networkId is automatically generated
        }
    }
};
```

This configuration specifies:
- Network name (`chat-testnet`)
- Port numbers for peer communication, RPC calls, and subscriptions
- Initial peers to connect to
- Database storage path
- Unique network identifier

Once both configurations are in place, your service will be automatically loaded when the dWeb node starts. The service will be accessible via RPC calls to the specified port, allowing frontends to interact with it.

That's it! You've created a decentralized chat service that:
- Validates message structure
- Processes messages according to your business logic
- Applies a percentage-based fee system
- Integrates with the broader dWeb ecosystem
- Is properly configured for network communication

This modular approach allows you to focus on your service's unique features while leveraging dWeb's infrastructure for consensus, security, and cross-cluster communication.

---

## Service Integration Flow

When a dWeb node starts:

1. The main configuration (`config/config.js`) is loaded to determine which services to enable
2. For each enabled service, its specific configuration is loaded (e.g., `webs/chat/config/config.js`)
3. The service's network class is instantiated with its configuration
4. The service registers its instruction types during initialization
5. The service starts and registers its RPC message handlers
6. The service is now available for RPC calls from frontends

Frontends can then connect to the service via its RPC port and send instructions that will be validated, processed, and propagated through the network according to the consensus rules.

---

## Development Guidelines

Simplicity, inclusivity, and flexibility is priotized in the development process:

1. **JavaScript Only** - No TypeScript, keeping the codebase accessible to developers of all levels
2. **Modular Design** - Break down complex functionality into manageable classes or helper files
3. **Cluster Independence** - Each cluster maintains complete autonomy in governance and operation
4. **Flexible Consensus** - Clusters can choose their own consensus mechanisms (only DPoS supported at the moment)
5. **Clear, Simple Code** - Prioritize readability and simplicity over complex abstractions

---

## Contributing

Contributions from developers of all skill levels are welcomed.

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

<p align="center">
  <em>dWeb - step towards a more open, free and privacy supported internet</em>
</p>
