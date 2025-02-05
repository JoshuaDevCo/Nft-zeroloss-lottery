<div id="top"></div>

<!-- ABOUT THE PROJECT -->

# NFT Lossless lottery

This project contains smart contracts for a lossless NFT lottery that leverage the AAVE protocol to guarentee a zero entrance fee and is completely automated with the help of chainlink keepers.

### Built With

* [Solidity](https://docs.soliditylang.org/)
* [Hardhat](https://hardhat.org/getting-started/)
* [openzeppelin](https://docs.openzeppelin.com)
* [chainlink](https://docs.chain.link/docs/conceptual-overview/)

<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#how-it-works">How it works</a></li>
    <li><a href="#project-structure">Project structure</a></li>
    <li>
      <a href="#how-to-run">How to Run</a>
      <ul>
       <li><a href="#prerequisites">Prerequisites</a></li>
       <li><a href="#contracts">Contracts</a></li>
      </ul>
    </li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#license">License</a></li>
  </ol>
</details>

<!-- Working -->

## How it works

####  - Overview :

This lottery is developed to be like the poolTogether protocol where users can participate in multiple lotteries without losing any money, this is possible by levereging the power of the AAVE protocol. To enter the lottery users must deposit an entrance fee in DAI which is immediately deposited into the AAVE pool to generate yield. 

As long as the user leaves his DAI into the lottery contract he can participate in the successive lotteries until he wins in which case he will recieve an NFT and get his DAI back or decide to leave and withdraw his entrance DAI, by doing so the user will always get his entrance money back and the lottery protocol will also make profit from the yield generated by the AAVE protocol.

####  - How long does the lottery last ?

The protocol will run multiple successive lotteries each one of them will last a given period of time represented by the variable `lotteryPeriod`, at the beginning it's set to 1 day but can be later augmented or decreased according to our community choices.

Between each lottery there will be a delay to allow admin to make changes in the lottery parameters (period, reward,...), this delay is represented by the variable `lotteryDelay` set to 1 hour by default.

####  - How is the lottery automated ?

The lottery uses the Chainlink keepers to allow the contract to start, pick winners and end lottery without any external intervention, when the lottery period ends the contract will automaticaly change the lottery state to `CALCULATING_WINNER` and send a request to chainlink VRFCoordinator in order to recieve random numbers, when the request is fufilled the contract will choose the winners, send them their NFTs and change the lottery state to `CLOSED`.

When the lottery contract return to the `CLOSED` state the keepers will automaticaly start another lottery, and this cycle will continue forever until the contract is paused by the admin to make some changes in the lottery parameters.

Note that admin can only pause the contract if the lottery is in the `CLOSED` state.

####  - How the entrance fee is calculated ?

The entrance fee is paid in DAI and is calculated using the following formula :

   ```sh
   entranceFee = ticketBasePrice * (1 + userNftCount)
   ```

Where : 

* ticketBasePrice = the base entrance fee, always equal to 100 DAI.

* userNftCount = the number of NFTs already won by the user from the lottery. 

When a new user enters the lottery his won't have any NFT in his wallet and will only pay the base ticket price, but once a participant wins the lottery will automaticaly get his NFT(s) and if he tries to enter the lottery again th new entrance fee will be 2 * ticketBasePrice because the participant has now more nfts in his wallet.

By using this formula the lottery ensures that the old winners pays more when entering the lottery again and makes a little bit more difficult to win multiple times.

####  - How many NFTs are given to the winner ?

Each lottery will have a determined number of NFTs to be given as reward to the winner(s), the NFT reward count is represented by the variable `lotteryNftRewardCount` which is set to 1 NFT by default and can be changed later.

####  - How many winners well be per lottery ?

Each lottery will have a determined number of winners represented by the variable `winnersPerLottery` which is set to 1 by default and can be changed later.

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- PROJECT STRUCTURE -->

## Project Structure

The contracts development and testing is done using the Hardhat framework, for this project there are 2 main contracts :
      <ul>
       <li><b>NFTCollection.sol :</b></li>
This is the NFT collection contract, i used the ERC721Enumerable standard with a fixed maximum supply, the NFT Lottery contract is set as  controller of the collection and is the only address able to mint new items to the lottery winners. 
       <li><b>NFTLottery.sol :</b></li>
The lottery contract allow user to enter and quit the lottery and it uses chainlink VRF coordinator to get random numbers and pick the winners, the contract is automated using chainlink keepers to make lottery start/end process run without any external interaction, the concept of zero money loss lottery is created using the AAVE protocol by using the deposit APY as a profit mechanism for the lottery.    
      </ul>
<p align="right">(<a href="#top">back to top</a>)</p>

<!-- USAGE GUIDE -->
## How to Run

### Prerequisites

Please install or have installed the following:
* [nodejs](https://nodejs.org/en/download/) and [yarn](https://classic.yarnpkg.com/en/)

Clone this repository with the command :

   ```sh
   git clone https://github.com/kaymen99/Nft-zeroloss-lottery.git
   ```

### Contracts

As mentioned before the contracts are developed with the Hardhat framework, before deploying them you must first install the required dependancies by running :
   ```sh
   cd Nft-zeroloss-lottery
   yarn
   ```
   
Next you need to setup the environement variables in the .env file, this are used when deploying the contracts :

   ```sh
    MAINNET_FORK_ALCHEMY_URL=https://eth-mainnet.alchemyapi.io/v2/<api-key>
    RINKEBY_ETHERSCAN_API_KEY=<api-key>
    RINKEBY_RPC_URL=https://eth-rinkeby.alchemyapi.io/v2/<api-key>
    POLYGON_RPC_URL="Your polygon RPC url from alchemy or infura"
    MUMBAI_RPC_URL="Your mumbai RPC url from alchemy or infura"
    PRIVATE_KEY=<private-key>
   ```
* <b>NOTE :</b> Only the private key is needed when deploying to the ganache network, the others variables are for deploying to the testnets or real networks and etherscan api key is for verifying your contracts on rinkeby etherscan.

* <b>NOTE :</b> To deploy on real/test networks you must get a chainlink subscription [here](https://vrf.chain.link). And before the deployment you'll need to add your chainlink subscription data and required external contracts addresses into the utils/helpers.js file for the network you wish to deploy to, for example for the Ethereum mainnet it would look like this :

   ```sh
   1: {
        name: "mainnet",
        daiAddress: "0x6b175474e89094c44da98b954eedeac495271d0f",
        aDaiAddress: "0x028171bCA77440897B824Ca71D1c56caC55b68A3",
        AAVELendingPool: "0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9",
        subscriptionId: 6926,
        gasLane: "0xd89b2bf150e3b9e13446986e571fb9cab24b13cea0a43ea20a6049a85cc807cc", // 30 gwei
        keepersUpdateInterval: "60",
        callbackGasLimit: 500000, // 500,000 gas
        vrfCoordinatorV2: "0x6168499c0cFfCaCD319c818142124B7A15E857ab",
    }
   ```

After going through all the configuration step, you'll need to deploy the 2 contracts to the ganache network by running: 
   ```sh
   yarn deploy --network ganache
   ```
   
* <b>IMPORTANT :</b> I used the ganache network for development purposes only, you can choose another testnet or real network if you want, for that you need to add it to the hardhat.config file for example for the rinkeby testnet  

   ```sh
   rinkeby: {
      url: RINKEBY_RPC_URL,
      accounts: [process.env.PRIVATE_KEY],
      chainId: 4,
    }
   ```

If you want to go through the contracts unit tests, you can do it by running:
   ```sh
   yarn test ./test/lottery-unit-test.js
   ```
And to test the end to end working cycle of the lottery use the command :
   ```sh
   yarn test ./test/lottery-e2e-test.js
   ```

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- Contact -->
## Contact

If you have any question or problem running this project just contact me: aymenMir1001@gmail.com

<p align="right">(<a href="#top">back to top</a>)</p>

<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE.txt` for more information.

<p align="right">(<a href="#top">back to top</a>)</p>
