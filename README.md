# How blockchain can make a real difference
Imagine if every citizen had the confidence that philanthropic commitments to support societal challenges such as extreme poverty were being honored. That resources were in fact filtering down to the people with the greatest needs. That every dollar donated and spent was making a real impact. Wouldn’t a more transparent system motivate people to more readily champion worthy causes? Or better yet, inspire them to donate more themselves?
Complete this pattern to build a simple blockchain network using Hyperledger Fabric, on which cause-specific pledges and fund transfers are made by the government, registered with aid organizations, and validated by Global Citizen.

Audience level: Intermediate Developers

## Prerequisites

1. Free [IBM Cloud account](https://ibm.biz/Bd2t6h)
2. The VirtualBox image containing Hyperledger Fabric and all other pre-reqs to complete this pattern
3. VirtualBox 6.x

Once VirtualBox 6.x is installed on your environment, double-click on the `Hyperledger Fabric.ova` file and accept the defaults to import it into VirtualBox.

## Overview
The image below gives a high level overview of the scenario described above and will form the basis of our demo application. This idea is taken from the 'Tracking Donations with blockchain' code pattern and modified so that it can also be deployed to the IBM Blockchain Platform V2.0 service

![](./images/01-overview-1.png)  

The application to support the above process involves a frontend, a business service and a smart contract. The smart contract will have as asset the cause-specific pledge (called the project pledge) and the transactions: `createProjectPledge`, `sendPledgeToGlobalCitizen`, `sendPledgeToGovOrg`, `updatePledge` and `transferFunds`. The business service implements the logic to support the transactions and finally exposes them as RESTful endpoints. The business service communicates with the smart contract via the Hyperledger Fabric SDK. The transaction invoked on the smart contract updates the ledger. All this is depicted in the schematic overview below.

![](./images/02-overview-2.png)  

As you can see in the image, building this demo application consists of three major steps. The fourth and last step is connecting all parts.

## Steps
1. [Build a frontend in Node-RED](#1-build-a-front-end-in-node-red)
2. [Build the Business Service](#2-build-the-business-service)
3. [Build the Smart Contract](#3-build-the-smart-contract)
4. [Connecting all parts](#4-connecting-all-parts)

## 1. Build a front-end in Node-RED
Logon to IBM Cloud using the free account you just created. Next, click the 'Catalog' on the top bar (1) and then search for Node-RED (2). Finally, click on the starter kit Node-RED Starter.

![](./images/03-nodered-1.png)  

Give the service a unique name (1) and select a nearby region to deploy to (e.g. London or Frankfurt). Click 'Create' (2) to create the service. 

![](./images/04-nodered-2.png)

The deployment takes a couple of minutes to complete, so you might want to move on to part 2 and start the creation of your business service and return later to complete the setup of your frontend.

Once the deployment is successfully completed, visit the app URL by clicking (1) on the image below.

![](./images/05-nodered-3.png)

Next, complete the Node-RED initial set-up wizard. Click Next on the first screen and choose a userid/password on the second one to secure your Node-RED environment. Click Next (2) and Next again to go to the final overview page of the wizard. Click Finish to complete it. 

![](./images/06-nodered-4.png)

On the welcome page click on the 'Go to your Node-RED flow editor' button.

![](./images/07-nodered-5.png)

Login with your Node-RED credentials (the userid/password you specified in the wizard). You are presented with the drawing canvas, that you'll need to create your Node-RED flows. As we are developing a frontend, we need UI nodes. Unfortunately, these are not present by default so we need to add them. For this, click the menu on the top right (1) and select 'Manage palette' (2).

![](./images/08-nodered-6.png)

Next, click Install (1) and type 'dashboard' in the search bar. Click Install (3) to start the installation of the Node-RED dashboard nodes. 

![](./images/09-nodered-7.png)

Once the installation successfully completes, click 'Close' to close the palette. The dashboard nodes are available to you as a separate section on the left-hand side. You might need to scroll down to see them.

![](./images/10-nodered-8.png)

For now, this completes the first step. You just created a Node-RED application on IBM Cloud that serves as frontend and that is ready to consume some REST services...

## 2. Build the Business Service

The business service is generated using the LoopBack framework. Two generators are used, the application generator and the OpenAPI generator. The latter generates the models and controllers that are needed by the business service and that are defined in the included `openapi.json` file. For this part of the pattern you need to start the VirtualBox image. Once the image is completely up and running, open a terminal session. 

![](./images/11-build-service-1.png)

The first step is to change directory to the `Development` folder and to clone the following git repository. 

```
cd ~/Development
git clone https://github.com/eciggaar/hlfabric-lab-code
```

This will download the necessary code snippets (needed to complete this guide) to your VirtualBox environment. At the command prompt of the terminal session, make sure your current directory is the `Development` folder and type:

```
lb4 ibm-tnw2019-bc
```

This invokes the application generator and generates a basic LoopBack application with the name `ibm-tnw2019-bc`. Hit <Enter> to accept the defaults. Once the basic application has been generated, change directory into the `ibm-tnw2019-bc` folder and invoke the OpeAPI generator to complete the set-up of our business service.

```
cd ~/Development/ibm-tnw2019-bc
lb4 openapi --url ../hlfabric-lab-code/business-service/openapi.json --validate true
```

Again, accept all defaults. The second command generates the models and controllers needed for the business service. They are defined in the `openapi.json` file. Browse to the `hlfabric-lab-code` folder to manually inspect the JSON file. You'll see definitions for the project pledge asset as well as transactions like `createProjectPledge` and `transferFunds`.

Finally, the skeleton for the business service can be tested by typing `npm start` (current directory should be `ibm-tnw2019-bc`).

```
npm start
```

Now open a browser session and browse to [http://localhost:3000/explorer](http://localhost:3000/exlorer). This gives an overview of all available REST endpoints of our business service. 

Try to invoke one of the endpoints (e.g. createProjectPledge) by clicking on the service (1). Next, click 'Try it' (2) and provide some sample values (3). Finally, click 'Execute' (4) to submit the request and test the service. 

![](./images/12-build-service-2.png)

The response will be a 500 internal server error. The reason for this is, that this is the current implementation of the service. It throws exactly this error message.

```ts
/**
 * The controller class is generated from OpenAPI spec with operations tagged
 * by CreateProjectPledgeController
 * 
 */
export class CreateProjectPledgeController {
  constructor() {}

  /**
   * 
   * 

   * @param requestBody 
   * @returns ResponseMessage model instance
   */
  @operation('post', '/CreateProjectPledge')
  async createProjectPledgeCreate(@requestBody() requestBody: CreateProjectPledge): Promise<ResponseMessage> {
    throw new Error('Not implemented');
  }

}
```

It is our job to implement the business service in a meaningful way. This is done in Part 4 of this pattern, but first we have to develop the smart contract.

## 3. Build the Smart Contract

Typically, business rules that need to be executed --- before a transaction can be recorded on the blockchain --- are captured in a so-called smart contract, also referred to as chaincode in Hyperledger Fabric. As this is an important part of our application, it is good to get familiar with the basics of smart contract development in Hyperledger Fabric. For this, we will follow the beginner tutorial that is part of the IBM Blockchain extension in Visual Code. After you completed the tutorial you should have a better understanding of how to develop and debug smart contracts, but also of how to deploy them to your network. The IBM blockchain extension itself is built to provide developers an integrated workflow for building blockchain applictions. To get started with the tutorial, click on the Visual Code launcher in your VirtualBox image.

![](./images/13-build-service-3.png)

Next, click the IBM Blockchain extension on the left-hand side (1), followed by the blockchain icon on the top right (2). This opens the IBM Blockchain Platform welcome page. On this page, click the 'Beginner tutorial' link (3) to start the tutorial.

![](./images/14-build-service-4.png)

At this point, you have completed the tutorial and gained basic knowledge on smart contract development. The next step is to build the smart contract for the tracking donations use case. For this, add the `smart contract` sub-folder from the `hlfabric-lab-code` repository to your workspace in Visual Code. Your workspace should look similar to.

![](./images/15-build-service-5.png)

Now inspect the `smart contract` folder. The structure of the project is similar to the one from the tutorial. There are 2 assets and 5 transactions. The transactions are defined and implemented in the file `src/my-projectpledge-contract.ts`. Take a look at the `createProjectPledge` transaction.

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

The last part of this section is to deploy the smart contract to our blockchain network. For this, open the IBM Blockchain extension in your Visual Code editor by clicking the blockchain icon on the left-hand side (1). Next, in the Smart Contract Packages panel, select the Options (2) and click 'Package a Smart Contract' (3).

![](./images/16-build-service-6.png)

Next, choose your `smart-contract` folder as workspace folder to package (1). 

![](./images/17-build-service-7.png)

The result should be a `global-citizen@0.0.1` package under the Smart Contract Packages view. The final step in this section is to test whether the smart contract was successfully deployed. For this, click on the contract in the Fabric Gateways panel (expend local_fabric, then click mychannel -> global-citizen@0.0.1). Now, click the createProjectPledge (1) method and select 'Submit Transaction' (2).

![](./images/18-build-service-8.png)

Pass the following arguments to the function and hit Enter.

![](./images/19-build-service-9.png)

The transaction should complete successfully. Finally, test whether you can read the project pledge by invoking the `readProjectPledge` function. Pass `aid1:001` as pledgeId and check if the response matches the screenshot below.

![](./images/20-build-service-10.png)

## 4. Connecting all parts

