# Hardhat NFT Marketplace 

<br/>
<p align="center">
<img src="./img/hero.png" width="500" alt="Hardhat NextJS Marketplace">
</a>
</p>
<br/>

This is a repository showing how to make an NFT Marketplace from scratch!

Huge Shoutout to Patrick Alpha for his help on this repo!



- [Hardhat NFT Marketplace](#hardhat-nft-marketplace)
- [Getting Started](#getting-started)
  - [Requirements](#requirements)
  - [Quickstart](#quickstart)
    - [Optional Gitpod](#optional-gitpod)
- [Usage](#usage)
  - [Testing](#testing)
- [Deployment to a testnet or mainnet](#deployment-to-a-testnet-or-mainnet)
- [Thank you!](#thank-you)


# Getting Started

## Requirements

- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
  - You'll know you did it right if you can run `git --version` and you see a response like `git version x.x.x`
- [Nodejs](https://nodejs.org/en/)
  - You'll know you've installed nodejs right if you can run:
    - `node --version` and get an ouput like: `vx.x.x`
- [Yarn](https://classic.yarnpkg.com/lang/en/docs/install/) instead of `npm`
  - You'll know you've installed yarn right if you can run:
    - `yarn --version` and get an output like: `x.x.x`
    - You might need to install it with `npm`

## Quickstart

```
git clone https://github.com/PatrickAlphaC/hardhat-nextjs-nft-marketplace-fcc
cd hardhat-nextjs-nft-marketplace-fcc
yarn
```


### Optional Gitpod

If you can't or don't want to run and install locally, you can work with this repo in Gitpod. If you do this, you can skip the `clone this repo` part.

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#github.com/PatrickAlphaC/hardhat-nft-marketplace-fcc)

> Remember if you are using gitpod then you cannot connect your local hardhat node with metamask. To resolve this you can use vs code or testnets instead of local node.


# Usage

Deploy:

```
yarn hardhat deploy
```

## Testing

```
yarn hardhat test
```



# Deployment to a testnet or mainnet

1. Setup environment variabltes

You'll want to set your `KOVAN_RPC_URL` and `PRIVATE_KEY` as environment variables. You can add them to a `.env` file, similar to what you see in `.env.example`.

- `PRIVATE_KEY`: The private key of your account (like from [metamask](https://metamask.io/)). **NOTE:** FOR DEVELOPMENT, PLEASE USE A KEY THAT DOESN'T HAVE ANY REAL FUNDS ASSOCIATED WITH IT.
  - You can [learn how to export it here](https://metamask.zendesk.com/hc/en-us/articles/360015289632-How-to-Export-an-Account-Private-Key).
- `KOVAN_RPC_URL`: This is url of the kovan testnet node you're working with. You can get setup with one for free from [Alchemy](https://alchemy.com/?a=673c802981)

2. Get testnet ETH

Head over to [faucets.chain.link](https://faucets.chain.link/) and get some tesnet ETH. You should see the ETH show up in your metamask.

3. Deploy

```
yarn hardhat deploy --network kovan
```



Tasks
1. Create a decentralised NFT marketplace
        1. `listItem`We'd need the listItem to list NFTs on the marketplace
        2. `buyItem` To buy the Nts
        3. `cancelItem` to cancel the listing
        4. `updateListing` to update the price
        5. `withdrawProceeds` So we can proceed to making withdrawals after someone pays for our listed nfts

        Creating our Nftmarketplace.sol
        Lets start by creating how we list the items 
        so we create listItem function

        -So here we dont want our marketplace to own this nft, so we are going to allow owners still own their NFTs at the same time giving the marketplace access so it can sell this NFT for owner
        so we ne3ed to call the getApproved function on our token to make sure that the market has this approval, so to do this we need the IERC721 interface which we import from openZeppelin, and then since we are importing from opoenzeppelin we run 
        yaen add --dev @openzeppelin
        After doing that we 
IERC721 nft = IERC721(nftAddress);
After having these we are going to need a data structure to list all these Nfts.
The best data structure to have is mapping, cause if it's an array it's not necessarily wrong but it's going to be harder to use as the array gets bigger, so our mapping is going to bw a global variable
We create a new type which is the struct listing so we can attach the price of the NFT and also who the seller is
so then we can map our Nft CA to the tokenId and then mapped to the listing, then we update the mapping using 
   s_listings[nftAddress][tokenId] = Listing(price, msg.sender);
Also best practise is that we emit an event anytime we update a mapping, then we create our event globally
To make sure we don't relist somethign that has already been listed, we create a notListed modifier
We use an isOwner modifier so that the NFT that's been listed is owned by the person listing it at the moment
we add these modifiers to our listItem function so the function can check and if there's any need to prompt an error it does.
Now aftter finishing with our listitem function, we create the buyitem function, which takes the nft address, tokenid as parameters and also this is an external function that's payable since only people outside the contract can call this function
TASK HAVE THE BUY ITEM USE CHAINLINK TO PAY FOR THIS NFT IN A SUBTOKEN, THAT WAY WE HAVE TO CONVERT THE TOKENS INBETWEEN111


Now for the buyitem we need a new modifier isListed where we check if the NFT has been listed, so we go into the mapping and just check the price, so 
if listing.price <= 0 we revert, i.e if there is no price for the NFT or if it's defaulted to zero 

Also important note in some part of the code _; is added just to complete our code in order for us to not have any errors
So under our buyItem function we  add the isListed modifier
then we add the condition to make sure that our listed price is less that the msg.value the sender is attaching to buy, then weadd an error incase price is not met, so nown once the money is sent it needs to belong to who listed the NFT which is whyw ecreate another data structure called proceeds where we keep tract of how much money people have earned by selling their NFTS.
We create a mapping that attaches the seller address and the amount they've being paid for the purchase, so now once someone's buy the item we update the proceeds with the msg.value. Now once someone's buys this NFT we'd want to delete the listing so as not to make another oerson pay for an NFT that's already sold so to delete the listing wwe just use the delete keyword
        delete (s_listings[nftAddress][tokenId]);
then we transfer NFT to new buyer using 
        IERC721(nftAddress).safeTransferFrom(listedItem.seller, msg.sender, tokenId);
so we are transferring from sellers address to buyer's address and whats being transferred is the token ID
Now in solidity and when working with blockchains best practise is to shift the risk of send ing moeny and all that to the user, i.e we shouldn't send the user the money but rather have them withdraw it from us instead
For the transfer we are going to use safetransferfrom as this is in this case better than transfer frrom and we are going to get errors if there is any issue
so since we are updating a mapping we emit an event, and then at the top we create a new event just like we did for the first case

Reentrancy attacks
this is one of the two most common hacks in the blockchain industry, second one which is the oracle attacks, it's under the sublesson contracts
Under the  contract we have a function for depositing  and then function for withdrawing

So the malicious way of clearing this account is we first create a new contract under which we then deposit to said contract and immediately we call the reentrant withdrawal, now like this it's not harmless but if there is a fallback function in place we can make another withdrawal before the withdraw function updates our account balance
so under thefalback function we use an if conditional statement to see if there is stll money in the contract and if there is still a balance left we withdraw that, so it's going to be a loop of us just withdrawing the money, which is why it's always important to update the state before sending the transaction to the user
But in essence thee are 2 ways to prevent this, the easy way and then the mutex way
Easy way, is to always call any external contract as the last step in our function and trasnaction, and we need to update the balances before we call the external code
Second step is by using a mutex lock using openzeppelin
so if we globalise a boolean locked
then under the withdrawal function, we require locked, so by requiring lock we only alow the command of our function to run once, openzeppelin also comes with a reentrancy guard which has a modifier of nonReentrant which helps us and does something similar to our locked, so to use this in our ocde we just import @openzeppelin...security
then we inherit the contract by using 
contract NftMarketplace is ReentrancyGuard 
and then any function we feel is going to have this reentrance issue we jsut add the modifier nonreentrant, and this adds the mutex mechanism though the mutex way is the best way
so thats why we add our safetransferfrom to the bottom of our buyitem function cause if it's way up someone could easi;y attack our contract using the reentrant method.
So now we have our buyItem and our listItem so let's create a cancelItem function,
so to cancel this, we are just going to use the delete keywword and delete the listiing, but we check if nft has already been listed before and if the person asking for deletion isOwner, then we emit an event, after that niw e update our listing with a new function
under the update functioin, we check if price is above zero then we update item if not we revert.

Now we need one more funciton which is our withdraw proceeds function
function withdrawProceeds() external {
        uint256 proceeds = s_proceeds[msg.sender];
        if (proceeds <= 0) {
            revert NoProceeds();
        }
        s_proceeds[msg.sender] = 0;
        (bool success, ) = payable(msg.sender).call{value: proceeds}("");
        require(success, "Transfer failed");
    }
so here we first check if our proceeds is > than 0 if ! we revert if true we then update the proceeds to zero adn after that send the proceeds to receiver, we a re updating proceeds first to zero so as not to haev vulnerability to reentrancy attacks and then we could require success

Then we add our getter functions and that's it for the code.

Now for Deploy
since we dont have any constructor, under our module.export function we have wheere we get our deployer from our namedaccounts that's in our helper-hardhat config
const/let args =[]
then const nftMarketplace = await deploy("NftMarketplace", {
        from: deployer,
        args: arguments,
        log: true,
        waitConfirmations: waitBlockConfirmations,
    })
    so we can await to get our contract.
    Then we verify our contract if we are not on a development chain,
    and then fiinally we have module.exports.tags = ["all", "nftmarketplace"]

So now since this is going to be an nft market place we are going to need some nfts w=and we add them to the test folder under our contracts
then now we are going to create a new deploy file for our basic nft


TESTS
Try to make as much coverage as possible
We create a const price, so that the price of our nft is always the same
 nftMarketplace = nftMarketplaceContract.connect(player)
 this is the line we use if we want to get our player to interact with the contract
 but here we are just going to connnect to our deployer which the nftmarket place does even if we dont explicitly tell it to do so, and also the nftmarketplace can't approve the CA and token id of the NFT cause it does not own this nft and the deployer does so only the deployer can approve.
 And only after it's been approved by the deployer before the nftmarketplace contract can call the transferfrom function
 And for the rest of the tests we just check to get as much coverage as possible
 SCRIPTS!
 OUR mint and list script
