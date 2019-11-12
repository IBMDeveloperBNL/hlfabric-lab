# How blockchain can make a real difference
Imagine if every citizen had the confidence that philanthropic commitments to support societal challenges such as extreme poverty were being honored. That resources were in fact filtering down to the people with the greatest needs. That every dollar donated and spent was making a real impact. Wouldn’t a more transparent system motivate people to more readily champion worthy causes? Or better yet, inspire them to donate more themselves?
Complete this pattern to build a simple blockchain network using Hyperledger Fabric, on which cause-specific pledges and fund transfers are made by the government, registered with aid organizations, and validated by Global Citizen.

**Audience level**: Intermediate Developers

## Prerequisites

All necessary prereqs to sucessfully complete this workshop have been put together in a virtual image. A VMware based image and a VirtualBox based image have been prepared. Make your choice and select one of the options below to complete the prereqs. 

<details>
  <summary>Using VirtualBox</summary>
  
  1. The [VirtualBox image]() containing Hyperledger Fabric and all other pre-reqs to complete this workshop.
  
  2. VirtualBox 6.x: [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)

  Once VirtualBox 6.x is installed on your system, double-click the file `Hyperledger Fabric.ova` and accept all defaults to import it into VirtualBox.

</details>

<details>
  <summary>Using VMware</summary>

  1. The [VMware image]() containing Hyperledger Fabric and all other pre-reqs to complete this workshop.

  2. VMware player: [https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html)

Once VMware Player is installed on your system, double-click he file `Hyperledger Fabric (xenial based).vmx` in the folder `Hyperledger Fabric (xenial based).vmwarevm` and accept all defaults to import the image into VMware Player.

</details>

or when you prefer not to use a virtual image and wanna set up everything yourself...

<details>
  <summary>Install all prereqs on your own machine</summary>

  1. Follow the steps at the **Requirements** section of the IBM Blockchain extension for Visual Code at:

  <li>https://github.com/IBM-Blockchain/blockchain-vscode-extension</li>

</details>

#### IBM Cloud Account
All options require an IBM Cloud account, so if you don't already have one, you can create one for free at: https://ibm.biz/Bdz2vG

## Overview
The image below gives a high level overview of the scenario described above and will form the basis of our demo application. The original idea comes from the 'Tracking Donations with blockchain' code pattern and is modified so that it can also be deployed to the IBM Blockchain Platform V2.0 service.

![](./images/01-overview-1.png)  

The application to support the above process involves a frontend, a business service and a smart contract. The smart contract will have as asset the cause-specific pledge (called the project pledge) and the transactions: `createProjectPledge`, `sendPledgeToGlobalCitizen`, `sendPledgeToGovOrg`, `updatePledge` and `transferFunds`. The business service implements the logic to support the transactions and finally exposes them as RESTful endpoints. The business service communicates with the smart contract via the Hyperledger Fabric SDK. The transaction invoked on the smart contract updates the ledger. All this is depicted in the schematic overview below.

![](./images/02-overview-2.png)  

As you can see in the image, building this demo application consists of three major steps. The fourth and last step is connecting all parts.

## Steps
1. [Build a frontend in Node-RED](#1-build-a-front-end-in-node-red)
2. [Build the Business Service](#2-build-the-business-service)
3. [Build the Smart Contract](#3-build-the-smart-contract)
4. [Connecting all parts](#4-connecting-all-parts)

## 1. Build a frontend in Node-RED
Logon to IBM Cloud using the free account you just created. Next, click the 'Catalog' on the top bar (1) and then search for Node-RED (2). Finally, click on the starter kit Node-RED Starter.

![](./images/03-nodered-1.png)  

Give the service a unique name (1) and select a nearby region to deploy to (e.g. London or Frankfurt). Click 'Create' (2) to create the service. 

![](./images/04-nodered-2.png)

The deployment takes a couple of minutes to complete, so you might want to move on to [Part 2](#2-build-the-business-service) and start the creation of your business service and return later to complete the setup of your frontend.

Once the deployment is successfully completed, visit the app URL by clicking on the 'Visit App URL' (1) on the image below.

![](./images/05-nodered-3.png)

Next, complete the Node-RED initial set-up wizard. Click Next on the first screen and choose a userid/password on the second one to secure your Node-RED environment. Click Next (1) and Next again to go to the final overview page of the wizard. Click Finish to complete it. 

![](./images/06-nodered-4.png)

On the welcome page click on the 'Go to your Node-RED flow editor' button.

![](./images/07-nodered-5.png)

Login with your Node-RED credentials (the userid/password you specified in the wizard). You are presented with the drawing canvas, that you'll need to create your Node-RED flows. As we are developing a frontend, we need UI nodes. Unfortunately, these are not present by default so we need to add them. For this, click the menu on the top right (1) and select 'Manage palette' (2).

![](./images/08-nodered-6.png)

Next, click Install (1) and type 'dashboard' in the search bar (2). In the search results look for `node-red-dashboard` and click Install (3) to start the installation of the Node-RED dashboard nodes (). 

![](./images/09-nodered-7.png)

Once the installation successfully completes, click 'Close' (4) to close the palette. The dashboard nodes are available to you as a separate section on the left-hand side. You might need to scroll down to see them.

![](./images/10-nodered-8.png)

For now, this completes the first step. You just created a Node-RED application on IBM Cloud that serves as frontend and that is ready to consume some REST services...

## 2. Build the Business Service

The business service is generated using the LoopBack framework. Two generators are used, the application generator and the OpenAPI generator. The latter generates the models and controllers that are needed by the business service and that are defined in the included `openapi.json` file. For this part of the pattern you need to start the VirtualBox image. Once the image is completely up and running, open a terminal session (1). 

![](./images/11-build-service-1.png)

The first step is to change directory to the `Development/blockchain` folder and to clone the following git repository. 

```
cd ~/Development/blockchain
git clone https://github.com/IBMDeveloperBNL/hlfabric-lab-code
```

This will download the necessary code snippets (needed to complete this guide) to your VirtualBox environment. At the command prompt of the terminal session, make sure your current directory is the `Development/blockchain` folder and type:

```
lb4 my-bc-app
```

This invokes the application generator and generates a basic LoopBack application with the name `my-bc-app`. Hit \<Enter> to accept the defaults. Once the basic application has been generated, change directory into the `my-bc-app` folder and invoke the OpeAPI generator to complete the set-up of our business service.

```
cd ~/Development/blockchain/my-bc-app
lb4 openapi --url ../hlfabric-lab-code/business-service/openapi.json --validate true
```

Again, accept all defaults. The second command generates the models and controllers needed for the business service. They are defined in the `openapi.json` file. Browse to the `hlfabric-lab-code` folder to manually inspect the JSON file. You'll see definitions for the project pledge asset as well as transactions like `createProjectPledge` and `transferFunds`.

Finally, the skeleton for the business service can be tested by typing `npm start` (current directory should be `my-bc-app`).

```
npm start
```

Now open a browser session and browse to [http://localhost:3000/explorer](http://localhost:3000/explorer). This gives an overview of all available REST endpoints of our business service. 

Try to invoke one of the endpoints (e.g. createProjectPledge) by clicking on the service (1). Next, click 'Try it out' (2) and provide some sample values (3). Finally, click 'Execute' (4) to submit the request and test the service. 

![](./images/12-build-service-2.png)

The response will be a '500 Internal Server Error'. The reason for this is, that this is the current implementation of the service. It throws exactly this error message.

```ts
export class CreateProjectPledgeController {
  constructor() {}
  /*
   * @param _requestBody 
   * @returns ResponseMessage model instance
   */
  @operation('post', '/CreateProjectPledge')
  async createProjectPledgeCreate(@requestBody() _requestBody: CreateProjectPledge): Promise<ResponseMessage> {
    throw new Error('Not implemented');
  }
}
```

It is our job to implement the business service in a meaningful way. This is done in Part 4 of this pattern, but first we have to develop the smart contract.

## 3. Build the Smart Contract

Typically, business rules that need to be executed --- before a transaction can be recorded on the blockchain --- are captured in a so-called smart contract, also referred to as chaincode in Hyperledger Fabric. As this is an important part of our application, it is good to get familiar with the basics of smart contract development in Hyperledger Fabric. 

For this, we will follow the beginner tutorial that is part of the IBM Blockchain extension in Visual Code. After you completed the tutorial you should have a better understanding of how to develop and debug smart contracts as well as of how to deploy them to your network. The IBM blockchain extension itself is built to provide developers an integrated workflow for building blockchain applictions. 

To get started with the tutorial, click on the Visual Code launcher (1) in your VirtualBox image.

![](./images/13-smart-contract-1.png)

Next, click the IBM Blockchain extension on the left-hand side (1), followed by the blockchain icon on the top right (2). This opens the IBM Blockchain Platform welcome page. On this page, click the 'Follow tutorials' link (3) to open the tutorials gallery. 

![](./images/14-smart-contract-2.png)

Next click the 'Local smart contract development' tutorial. This one takes you through the basics of smart contract development for Fabric. It is highly recommended to complete this tutorial in order to understand the rest of this section.

![](./images/40-smart-contract-10.png)

At this point, you have completed the tutorial and gained basic knowledge on smart contract development. The next step is to build the smart contract for the tracking donations use case. For this, change directory to `smart-contract` folder in the `hlfabric-lab-code` repo and run `npm install`.

```
cd ~/Development/blockchain/hlfabric-lab-code/smart-contract
npm install
```

Next, add the `smart-contract` sub-folder to your workspace in Visual Code. Your workspace should look similar to:

![](./images/15-smart-contract-3.png)

Now inspect this smart contract project. The structure of it is similar to the one from the tutorial. In this case, there are 2 assets and 5 transactions. The transactions are defined and implemented in the file `src/my-projectpledge-contract.ts`. Take a look at the `createProjectPledge` transaction.

```ts
@Transaction()
async createProjectPledge(ctx: Context, aidOrg: string, pledgeNumber: string, name: string, description: string, fundsRequired: number): Promise<void> {
    const pledgeId = aidOrg + ':' + pledgeNumber;
    const exists = await this.projectPledgeExists(ctx, pledgeId);

    if (exists) {
        throw new Error(`The project pledge ${pledgeId} already exists`);
    }

    // create an instance of the project pledge
    let pledge = new ProjectPledge();

    pledge.pledgeNumber = pledgeNumber;
    pledge.aidOrg = aidOrg;
    pledge.name = name;
    pledge.description = description;
    pledge.fundsRequired = fundsRequired;
    pledge.status = ppState.INITIALSTATE;

    const buffer = Buffer.from(JSON.stringify(pledge));
    await ctx.stub.putState(pledgeId, buffer);
}
```

The `@Transaction()` method decorator is used to indicate that the function actually transacts on the blockchain. Functions that are only reading from the ledger have the decorator `@Transaction(false)`. Inside the function is checked whether a project pledge with the given pledgeId already exists. If not, a new project pledge asset is constructed based on the values from the input parameters. Finally, the project pledge is converted to a buffer and submitted to the blockchain by calling `ctx.stub.putState`. Now, have a close look at the other functions in `src/my-projectpledge-contract.ts` so that you understand the different transactions involved. 

The above transactions are always related to a given project pledge -- the asset. So, in our contract we have one main asset -- which is defined in `src/projectpledge.ts` -- and a supporting funding asset (`src/funding.ts`). Have a close look at both files. You'll see that they are both decorated with the `@Object()` method decorator and that they define the attributes (properties) of the asset.

```ts
/*
 * SPDX-License-Identifier: Apache-2.0
 */

import { Object, Property } from 'fabric-contract-api';

@Object()
export class Funding {
    /**
     *
     */
    @Property()
    fundingType: 'WEEKLY' | 'MONTHLY' | 'SEMIANNUALY' | 'ANNUALY';
  
    /**
     *
     */
    @Property()
    nextFundingDueInDays: number;

    :
    :

```

The last part of this section is to deploy the smart contract to our blockchain network. For this, open the IBM Blockchain extension in your Visual Code editor by clicking the blockchain icon on the left-hand side (1). Next, in the Smart Contract Packages panel, select the Options (2) and click 'Package Open Project' (3).

![](./images/16-smart-contract-4.png)

Next, choose your `smart-contract` folder as workspace folder to package (1). 

![](./images/17-smart-contract-5.png)

The result should be a `global-citizen@0.0.1` package under the 'Smart Contract Packages' palette. Next, make sure your local fabric is started by checking the 'Local Fabric Ops' palette. If you see the message 'Local Fabric (click to start)', click that message to start your local blockchain network. If the network is started, click instantiate (1) and hit \<Enter\> to accept 'mychannel' (2) as channel to instantiate the contract on. Make sure the global-citizen@0.0.1 package is selected when asked which package to select. Hit \<Enter\> to accept the defaults on all other questions asked.

![](./images/39-smart-contract-9.png)

The final step in this section is to test whether the smart contract was successfully deployed. For this, click on the contract in the Fabric Gateways panel (expand local_fabric, then click mychannel -> global-citizen@0.0.1). Now, click the createProjectPledge (1) method and select 'Submit Transaction' (2).

![](./images/18-smart-contract-6.png)

Pass the argument string `["aid1","001","test","test desc","100000"]` to the function and hit Enter.

![](./images/19-smart-contract-7.png)

The transaction should complete successfully. Finally, test whether you can read the project pledge by invoking the `readProjectPledge` function. Pass `["aid1:001"]` as pledgeId and check if the response matches the screenshot below.

![](./images/20-smart-contract-8.png)

## 4. Connecting all parts

### Connect the Business Service to the smart contract

The business service communicates with the smart contract via the Hyperledger Fabric SDK. We concentrated all the communication with the blockchain network in the file `blockchainClient.ts`. To include this file in your project as well, copy it from the `hlfabric-lab-code` repo to the `src` folder of your business service.

```
cd ~/Development/blockchain
cp ./hlfabric-lab-code/business-service/src/blockchainClient.ts ./my-bc-app/src
```

Next, move the `local_fabric` folder from the `hlfabric-lab-code` repo to the root folder of your business service. This folder contains the connection information to your local Hyperledger Fabric network, as well as some JavaScript files to enroll an admin user and a normal user.

```
cd ~/Development/blockchain
mv ./hlfabric-lab-code/business-service/local_fabric ./my-bc-app
```

Now open the blockchain client in Visual Code. An important function in this class is the function `connectToNetwork()`.

```ts
    async connectToNetwork() {

      try {
        await gateway.connect(ccp, { wallet, identity: appAdmin, discovery: gatewayDiscovery });

        // Connect to our local fabric
        const network = await gateway.getNetwork('mychannel');

        console.log('Connected to mychannel. ');

        // Get the contract we have installed on the peer
        const contract = await network.getContract('global-citizen');

        let networkObj = {
          contract: contract,
          network: network
        };

        return networkObj;

      } catch (error) {
        console.log(`Error processing transaction. ${error}`);
        console.log(error.stack);

        return error;
      }
    }
```

First a connection is made to the gateway using the admin identity of the network (line 30 of the file). Next, an instance of the `mychannel` network is retrieved in line 33. An instance of the contract object is obtained in line 38. Note that in this demo we explicitly look for `global-citizen` as contract name and that this name matches the name of the smart contract that was deployed to the network in part 3 of this pattern. The contract and network are stored in a -- for this purpose created -- object called networkObj. This object is returned to the caller in line 45. 

Once connected to the network, every transaction has its own function in the client. All functions have the same layout. They expect one generic input object, called `args`. This object needs to have an attribute called `contract` and one called `function`. Besides these two args contain all attributes needed to successfully invoke the smart contract transaction. As an example, the `createProjectPledge` function is listed below.

```ts
    async createProjectPledge(args: any) {
      // Calling transaction1 transaction in the smart contract
      let response = await args.contract.submitTransaction(args.function, args.aidOrg, args.pledgeNumber, args.name, args.desc, args.fundsRequired);

      return response;
    }
```

Now all transactions can be invoked via the client, it is time to implement the controller logic. For this, add the `my-bc-app` folder to your workspace in Visual Code and expand the `src/controllers` folder. Double-click the first controller `create-project-pledge.controller.ts` in the list. Now replace line 23

```ts
throw new Error('Not implemented');
```

with the following code:

```ts
     try {
      let networkObj = await blockchainClient.connectToNetwork();

      if (networkObj && !(networkObj.stack)) {
        // Construct data object that serves as
        let inputObj = {
          function: 'createProjectPledge',
          contract: networkObj.contract,
          pledgeNumber: _requestBody.pledgeNumber,
          name: _requestBody.name,
          desc: _requestBody.description,
          fundsRequired: _requestBody.fundsRequired,
          aidOrg: _requestBody.aidOrg
        };

        await blockchainClient.createProjectPledge(inputObj);
      } else {
        // Couldn't connect to network, so passing this object on to catch clause...
        throw new Error(networkObj.message);
      }

      let responseMessage: ResponseMessage = new ResponseMessage({ message: 'added project pledge' });
      return responseMessage;

    } catch (error) {
      let responseMessage: ResponseMessage = new ResponseMessage({ message: error, statusCode: '400' });
      return responseMessage;
    } finally {
      blockchainClient.disconnectFromNetwork();
    }
```

You'll probably notice that the blockchainClient variable cannot be resolved. This is because there is no blockchainClient defined in the controller. To define one, type the following line of code above the class definition in the controller.

```ts
let blockchainClient = new BlockChainModule.BlockchainClient();
```

The code should now look like:

```ts
/* tslint:disable:no-any */
import { operation, param, requestBody } from '@loopback/rest';
import { CreateProjectPledge } from '../models/create-project-pledge.model';
import { ResponseMessage } from '../models/response-message.model';
import { BlockChainModule } from '../blockchainClient';

let blockchainClient = new BlockChainModule.BlockchainClient();

/**
 * The controller class is generated from OpenAPI spec with operations tagged
 * by CreateProjectPledgeController
 *
 */
export class CreateProjectPledgeController {
  constructor() { }

  :
  :
```

and the errors should have disappeared. Make sure your changes to the controller code are saved. Now repeat this for all other controllers in the folder. Below  the code snippets are listed for each of the controllers.

**project-pledge.controller.ts**
```ts
    try {
      let networkObj = await blockchainClient.connectToNetwork();

      if (networkObj && !(networkObj.stack)) {
        let inputObj = {
          function: 'readProjectPledge',
          contract: networkObj.contract,
          id: id
        };

        return await blockchainClient.queryProject(inputObj);
      } else {
        // Couldn't connect to network, so passing this object on to catch clause...
        throw new Error(networkObj.message);
      }

    } catch (error) {
      return error;
    } finally {
      blockchainClient.disconnectFromNetwork();
    }
```

**send-pledge-to-global-citizen.controller.ts**
```ts
    try {
      let networkObj = await blockchainClient.connectToNetwork();

      if (networkObj && !(networkObj.stack)) {
        // Construct data object that serves as
        let inputObj = {
          function: 'sendPledgeToGlobalCitizen',
          contract: networkObj.contract,
          pledgeId: _requestBody.pledgeId
        };

        await blockchainClient.sendPledgeToGlobalCitizen(inputObj);
      } else {
        // Couldn't connect to network, so passing this object on to catch clause...
        throw new Error(networkObj.message);
      }

      let responseMessage: ResponseMessage = new ResponseMessage({ message: 'Sent project pledge ' + _requestBody.pledgeId + ' to citizen org for review...' });
      return responseMessage;

    } catch (error) {
      let responseMessage: ResponseMessage = new ResponseMessage({ message: error, statusCode: '400' });
      return responseMessage;
    } finally {
      blockchainClient.disconnectFromNetwork();
    }
```

**send-pledge-to-gov-org.controller.ts**
```ts
    try {
      let networkObj = await blockchainClient.connectToNetwork();

      if (networkObj && !(networkObj.stack)) {
        // Construct data object that serves as
        let inputObj = {
          function: 'sendPledgeToGovOrg',
          contract: networkObj.contract,
          pledgeId: _requestBody.pledgeId
        };

        await blockchainClient.sendPledgeToGovOrg(inputObj);
      } else {
        // Couldn't connect to network, so passing this object on to catch clause...
        throw new Error(networkObj.message);
      }

      let responseMessage: ResponseMessage = new ResponseMessage({ message: 'Sent project pledge ' + _requestBody.pledgeId + ' to government org for review...' });
      return responseMessage;

    } catch (error) {
      let responseMessage: ResponseMessage = new ResponseMessage({ message: error, statusCode: '400' });
      return responseMessage;
    } finally {
      blockchainClient.disconnectFromNetwork();
    }
```

**transfer-funds.controller.ts**
```ts
    try {
      let networkObj = await blockchainClient.connectToNetwork();

      if (networkObj && !(networkObj.stack)) {
        // Construct data object that serves as
        let inputObj = {
          function: 'transferFunds',
          contract: networkObj.contract,
          pledgeId: _requestBody.pledgeId
        };

        await blockchainClient.transferFunds(inputObj);
      } else {
        // Couldn't connect to network, so passing this object on to catch clause...
        throw new Error(networkObj.message);
      }

      let responseMessage: ResponseMessage = new ResponseMessage({ message: 'Total funds received incremented for project pledge ' + _requestBody.pledgeId });
      return responseMessage;

    } catch (error) {
      let responseMessage: ResponseMessage = new ResponseMessage({ message: error, statusCode: '400' });
      return responseMessage;
    } finally {
      blockchainClient.disconnectFromNetwork();
    }
```

**update-pledge.controller.ts**
```ts
    try {
      let networkObj = await blockchainClient.connectToNetwork();

      if (networkObj && !(networkObj.stack)) {
        // Construct data object that serves as
        let inputObj = {
          function: 'updatePledge',
          contract: networkObj.contract,
          pledgeId: _requestBody.pledgeId,
          fundingType: _requestBody.fundingType,
          approvedFunding: _requestBody.approvedFunding,
          fundsPerInstallment: _requestBody.fundsPerInstallment
        };

        await blockchainClient.updatePledge(inputObj);
      } else {
        // Couldn't connect to network, so passing this object on to catch clause...
        throw new Error(networkObj.message);
      }

      let responseMessage: ResponseMessage = new ResponseMessage({ message: 'Updated project pledge ' + _requestBody.pledgeId });
      return responseMessage;

    } catch (error) {
      let responseMessage: ResponseMessage = new ResponseMessage({ message: error.message, statusCode: '400' });
      return responseMessage;
    } finally {
      blockchainClient.disconnectFromNetwork();
    }
```
Now, try to run your business service by running `npm start` from the root directory of your service.

```
cd ~/Development/blockchain/my-bc-app
npm start
```

This will fail with the following error message:

![](./images/21-connecting-parts-1.png)

The `fabric-network` module is not added as a dependency yet. To add this module, open the `package.json` file in Visual Code and add the following line to the `devDependencies` section.

```
"fabric-network": "^1.4.2"
```

Now, run `npm install` in the same directory (`~/Development/blockchain/my-bc-app`) as before.

When the blockchain network was started for the first time (in Part 3 of this pattern), an admin user was registered with our Certificate Authority. Now we need to send an enroll call to the CA server and retrieve the enrollment certificate (eCert) for this user. We won’t delve into enrollment details here, but suffice by saying that the SDK --- and by extension our applications --- need this cert in order to form a user object for the admin. This user object (identity) is needed to connect to the Hyperledger Fabric gateway (line 30 in the `blockchainClient.ts` file). 

To enroll the admin user, type the following in the `local_fabric` folder of your business service.

```
cd ~/Development/blockchain/my-bc-app/local_fabric
node enrollAdmin.js
```

It is time to test the connected business service. For this, start the application by running `npm start` in the root folder of your project.

```
cd ~/Development/blockchain/my-bc-app
npm start
```

The business service can be accessed on [http://localhost:3000/explorer](http://localhost:3000/explorer). Now, test all transactions to make sure the controllers are properly implemented and can access the smart contract.

We start with the CreateProjectPledgeController. We create a new project pledge using the parameters as shown in the screenshot below. Remember, to get to the form where you can provide the project pledge details, first click the `CreateProjectPledgeController`. Then, click the POST request `CreateProjectPledge` and finally the 'Try it out' button on the right. The POST request should return with a HTTP 200 OK response message.

![](./images/22-connecting-parts-2.png)

Next, send this project pledge to the citizen organization for review by invoking the `SendPledgeToGlobalCitizenController` function. If you used the parameters from the screenshot above, you should use 'aid2:002' as plegeId.

![](./images/23-connecting-parts-3.png)

Next, send the project pledge to the government organization. Use the same pledgeId as above.

![](./images/24-connecting-parts-4.png)

The government organization can update the pledge with the total fund approved, funding type and the fund installment per transfer. To update the project pledge with these details, invoke the controller with the following parameters...

![](./images/25-connecting-parts-5.png)

Finally, you can start transferring funds by invoking the `TransferFundsController` controller with `aid2:002` as pledgeId.

![](./images/26-connecting-parts-6.png)

Now query the project pledge by invoking the `ProjectPledgeController` controller. Use the same pledgeId as above. 

![](./images/27-connecting-parts-7.png)

You should see an output similar to:

![](./images/28-connecting-parts-8.png)

Note the funds object that is part of the project pledge now. If you invoke transferFunds again, followed by the above project pledge query, you should see an increase of the `totalFundsReceived` attribute.

### Connecting the business service to the frontend

As the business service exposes the transactions as RESTful endpoints, it is relatively easy to connect the frontend to it. The most important part is to open up the local running business sevice to the outside world. This is done via a tool called `ngrok`. To start this, open a separate tab in the terminal that you opened in your Virtual Box environment and type 

```
ngrok http 3000
```

This will create a secure tunnel that allows communication (HTTP) from the outside world towards your locahost (the VirtualBox image) listening on port 3000.

![](./images/29-connecting-parts-9.png)

The result should look like:

![](./images/30-connecting-parts-10.png)

At this point your running business service should also be accessible via the URL above (1). Try this in your browser to verify this is true for you as well.

To show that the frontend can connect to the business service, open a new tab in your browser and look up the app URL of the Node-RED application that you deployed in Part 1. Paste this URL. On the Node-RED welcome page that is shown, click the 'Go to your Node-RED editor' button to open your flow editor. Logon with your Node-RED userid / password when needed (also created in Part 1). On the empty canvas, we will create a very simple flow that, on request, reads a project pledge with a given pledgeId and then, sends the output to the debug node. 

From the node pallet drag the inject node (1) and the debug node (2) to the canvas. 

![](./images/31-connecting-parts-11.png)

Now scroll down in the node pallet till you see the 'network' section and drag the HTTP request node (1) onto the canvas as well. Connect the three nodes together as shown at (2) in the screenshot below. 

![](./images/32-connecting-parts-12.png)

Next, double-click the 'http request' node and fill in the URL (1) as shown below. Replace the secure base URL with your secure ngrok URL and click 'Done' (2). Finally, click 'Deploy' (3) to deploy the flow. 

![](./images/33-connecting-parts-13.png)

To test the flow click the small box (1) left from the timestamp inspect node. Click the debug tab on the right to see the result. This should be a string representing the project pledge with pledgeId `aid2:002`.

![](./images/34-connecting-parts-14.png)

Congratulations, you connected all three parts of the demo application!! The frontend (Node-RED) invokes via a REST call the business service. The business service uses the Hyperledger Fabric SDK to invoke a transaction of the smart contract and transact with the blockchain network.

### Build a basic dashboard to create project pledges
Okay, agreed, this still is a very minimalistic UI...Therefore, let's extend it a bit and create a web UI that can be used to create new project pledges. For this, copy the content of the `create-project-pledge.json` in the `frontend` folder of the `hlfabric-lab-code` repo to your clipboard. Next, click the menu (1) on the top-right. Select Import (2), followed by Clipboard (3). 

![](./images/35-connecting-parts-15.png)

Now, use CTRL-V to paste the clipboard and click 'Import' (1) to complete the import. 

![](./images/36-connecting-parts-16.png)

The result should be similar to:

![](./images/37-connecting-parts-17.png)

Double-click on both HTTP request nodes and make sure the base URL (https://<something>.ngrok.io) points to your secure ngrok URL. Finally, click 'Deploy' (1) to deploy the flow. You can now test the dashboard by going to the same base URL as your Node-RED application only suffixed with `/ui` instead of `/red`. So the URL for the dashboard should look like:

```
http://<node-red-base-url>/ui
```

The result should be similar to: 

![](./images/38-connecting-parts-18.png)

**CONGRATULATIONS!!** :smiley: :+1: 

You successfully created a blockchain application that is capable of tracking donations according to the simplified business scenario mentioned at the top of this pattern. Next, play around with the dashboard and create some additional project pledges and query them. Have a look at the 'Blockchain' log in Visual Code to confirm that your dashboard activities actually transact with the blockchain network.

