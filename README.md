# Mint an ERC-721 with Hardhat

This guide details how to mint an ERC-721 token using Hardhat and the OpenZeppelin Contracts Wizard, deploying it on the Swisstronik blockchain.

## Prerequites
- Node.js and npm
- Visual Studio Code
- MetaMask Wallet

## Steps
1. Setup Project
   - Create a new repository
   - Clone the new repository to your machine
     
     ```
     git clone <new-repo-url>
     ```
   - Enter the directory of the cloned repository and install Hardhat
     
     ```
     cd <new-repo-url>
     npm install --save-dev hardhat
     ```
   - Initialize Hardhat project and open in VS Code
     
     ```
     npx hardhat init
     code .
     ```
   - Delete unnecessary files
     - `Lock.sol` in Contracts folder and `Ignition` folder. 
   - Set variable with your MetaMask private key
     
     ```
     npx hardhat vars set PRIVATE_KEY
     ```
   - Configure Swisstronik network in `hardhat.config.js`
     
     ```
     require("@nomicfoundation/hardhat-toolbox");
     require("@nomicfoundation/hardhat-web3-v4");
     
     // Remember to use the private key of a testing account
     // For better security practices, it's recommended to use npm i dotenv for storing secret variables
     
     //use configuration-variables in hardhat to set PRIVATE_KEY variable
     const PRIVATE_KEY = vars.get("PRIVATE_KEY");
     
     module.exports = {
       defaultNetwork: "swisstronik",
       solidity: "0.8.20",
       networks: {
         swisstronik: {
           url: "https://json-rpc.testnet.swisstronik.com/",
           accounts: [`0x` + `${PRIVATE_KEY}`],
         },
       },
     };
     ```
2. Create ERC-721 Smart Contract
   - In your browser, search for OpenZeppelin Contracts Wizard
   - Navigate to the website and choose ERC-721.
     
     ![image](https://github.com/user-attachments/assets/558657c3-b664-486f-80a0-d45985c7512e)
     
   - Set your token name and symbol.
   - In Features, Enable "Mintable" and "Auto Increment Ids".
   - Copy the generated Solidity code.
   - Create a new file `MyToken.sol` in Contracts folder and paste the code below.
     
     `Contract/MyToken.sol`
     ```
     // SPDX-License-Identifier: MIT
     // Compatible with OpenZeppelin Contracts ^5.0.0
     pragma solidity ^0.8.20;
     
     import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
     import "@openzeppelin/contracts/access/Ownable.sol";
     
     contract MyToken is ERC721, Ownable {
         uint256 private _nextTokenId;
     
         constructor(address initialOwner)
             ERC721("MyToken", "MTK")
             Ownable(initialOwner)
         {}
     
         function safeMint(address to) public onlyOwner {
             uint256 tokenId = _nextTokenId++;
             _safeMint(to, tokenId);
         }
     }
     ```

   - Install some necessary hardhat tools, web3-utils, openzepplin contracts and swisstronik plugin
      
     ```
     npm install @nomicfoundation/hardhat-web3-v4 web3-utils @openzeppelin/contracts @swisstronik/web3-plugin-swisstronik web3 --save-dev
     ```
3. Deploy the Smart Contract
   - Create a new file `deploy.js` in Scripts folder and paste the code below.
     ```
     const { ethers } = require("hardhat");
     async function main() {
         // Get the ContractFactory and Signers here.
         const [deployer] = await ethers.getSigners();
         console.log("Deploying contracts with the account:", deployer.address);
         // We get the contract to deploy
         const TokenContract = await ethers.getContractFactory("MyToken");
         // Deploy the contract and pass the initialOwner address
         const initialOwner = deployer.address;
         const myToken = await TokenContract.deploy(initialOwner);
         console.log("Non-Fungible Token Contract address:", myToken.target);
     }
     main()
         .then(() => process.exit(0))
         .catch((error) => {
             console.error(error);
             process.exit(1);
         });
     ```
   - Run the deployment script to get the contract address
     ```
     npx hardhat run scripts/deploy.js --network swisstronik
     ```
   - Copy the deployed contract address
     
     ![image](https://github.com/user-attachments/assets/e0e2863d-b3f3-4ddd-8ee3-a8f5edbf1f18)

4. Mint 1 ERC-721 Token
   - Create a new file `mint.js` in Scripts folder and paste the code below.

     ```
     // Import necessary modules from Hardhat and SwisstronikJS
     const hre = require("hardhat");
     const { SwisstronikPlugin } = require("@swisstronik/web3-plugin-swisstronik");
     hre.web3.registerPlugin(new SwisstronikPlugin(hre.network.config.url));
     async function main() {
         const replace_contractAddress = "<Replace_Contract_Address>";
         const [from] = await hre.web3.eth.getAccounts();
         const contractFactory = await hre.ethers.getContractFactory("MyToken");
         const ABI = JSON.parse(contractFactory.interface.formatJson());
         const contract = new hre.web3.eth.Contract(ABI, replace_contractAddress);
         const replace_functionArgs = "<Replace_Wallet_Address>"; // Recipient address
         console.log("Minting 1 token...");
         try {
             const transaction = await contract.methods.safeMint(replace_functionArgs).send({ from });
             console.log("Transaction submitted! Transaction details:", transaction);
             // Display success message with recipient address
             console.log(`Transaction completed successfully! ✅ Non-Fungible Token minted to ${replace_functionArgs}`);
             console.log("Transaction hash:", transaction.transactionHash);
         } catch (error) {
             console.error(`Transaction failed! Could not mint NFT.`);
             console.error(error);
         }
     }
     main().catch((error) => {
         console.error(error);
         process.exitCode = 1;
     });
     ```
   - Run the mint script
     
     ```
     npx hardhat run scripts/mint.js --network swisstronik
     ```
5. Push Code to Github

   ```
   git add .
   git commit -m "First Commit"
   git push -u origin main
   ```


