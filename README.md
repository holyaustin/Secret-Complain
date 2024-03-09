# Secret-Complain on the Polygon and Secret Contracts


### Introduction
Have ever been in a situation that you which to make a striking revelation or complain about something you dont like but you are scared of revealing your identity. This dApp solves that problem by allowing you to drop 
1. Complains
2. revelations
3. Whitle blowing 

Install the node dependencies:
npm install
Update the env file with your Secret Network wallet mnemonic, EVM wallet private key, and Infura API key:

Make sure your Infura API key is configured for Polygon Matic testnet 😎
Next, generate encryption keys for your EVM contract and automatically add them to your  env file by running create_keys.js: 
npx hardhat --network polygon run ./scripts/create_keys.js
Now you are ready to upload the smart contracts! 🚀
Upload and Instantiate Secret contract
cd into examples/evm-confidential-voting/secret-network:
cd examples/evm-confidential-voting/secret-network
Compile the Secret Network smart contract:
make build-mainnet
If you are on a Mac and run into compilation error:
error occurred: Command “clang”
Make sure you have the latest version of Xcode installed and then update your clang path by running the following in your terminal: 

cargo clean
AR=/opt/homebrew/opt/llvm/bin/llvm-ar CC=/opt/homebrew/opt/llvm/bin/clang cargo build --release --target wasm32-unknown-unknown

See here for instructions on updating your clang path. 
cd into examples/evm-confidential-voting/secret-network/node
cd examples/evm-confidential-voting/secret-network/node
Install the node dependencies: 
npm install
Upload the Secret Network smart contract:
node index 
Upon successful upload a codeHash and contract address is returned: 
Starting deployment…
codeId:  3226
Contract hash: 4fb0433133d3e9441790ab713ad8000bb99c3894a36b679f355ffaea052035b9
Instantiating contract…
contract address:  secret1lft908ws8h034zpa6y2gsq2shpsksekl05gqgq
Update the env file with your codeHash and contract address: 

Execute Secret Network Smart Contract
Now that your Secret Network smart contract is instantiated, you can execute the contract to generate encryption keys as well as decrypt encrypted messages. To learn more about the encryption schema, see the EVM encryption docs here.
Create Keys
To create encryption keys, run node create_keys: 
node create_keys
After you generate your keys successfully, query your public key:
node get_keys
Which returns your public key as a string: 
2,251,34,75,188,184,127,245,254,38,103,132,60,248,107,222,239,201,55,224,56,34,139,127,66,213,21,19,126,68,113,76,233
Add the public_key to your env file:
SECRET_PUBLIC_KEY=2,251,34,75,188,184,127,245,254,38,103,132,60,248,107,222,239,201,55,224,56,34,139,127,66,213,21,19,126,68,113,76,233
Now it's time to upload a Voting contract to the EVM, which you will use to store encrypted votes that can only be decrypted by your Secret Network smart contract. 
Upload and Instantiate Polygon Smart Contract
cd into examples/evm-confidential-voting/polygon:
cd evm-confidential-voting/polygon
Compile your Solidity smart contract: 
npx hardhat compile
Once the contract is compiled successfully, upload the contract to Polygon testnet: 
npx hardhat run scripts/deploy.js --network polygon
Note the contract address:
PrivateVoting deployed to: 0x90c6C32994c622f3882579C76C4b4c41022da494
 Add the Polygon testnet contract address to your env file:
CONTRACT_ADDRESS="0x90c6C32994c622f3882579C76C4b4c41022da494"
Execute Polygon Testnet Smart Contract
Now that your Polygon smart contract is instantiated, you can execute the contract to create voting proposals as well as vote on existing proposals. You can review the solidity contract here. 
Create Voting Proposal
To create a proposal, you must include a proposal description (a "yes" or "no" question) as well as a quorum number, which is the number of unique wallet addresses required to vote on the proposal before it closes. 
For testing purposes, set quorum to 1 unless you want to test with multiple wallet addresses
  function createProposal(string memory description, uint quorum) external returns (uint proposalId) {
        proposalId = nextProposalId;
        nextProposalId++;
        Proposal storage proposal = proposals[proposalId];
        proposal.id = proposalId;
        proposal.description = description;
        proposal.quorum = quorum;
        emit ProposalCreated(proposalId, description);
    }
Open create_proposal.js and update the proposal_description to a "yes" or "no" question of your choice:
const proposal_description = "Do you love Secret?";
Then run create_proposal.js: 
npx hardhat --network polygon run ./scripts/create_proposal.js
A transaction hash will be returned upon successful execution: 
Transaction hash: 0x1b26f860328a4f01236dac49cd20cfe4a06a80826514bb7a46a7ec890886ca4c
Create Proposal function executed successfully!
You can query the proposal by running query_by_proposal_id:
npx hardhat --network polygon run ./scripts/query_by_proposal_id.js
Be sure to update the proposalId in query_by_proposal_id.js with the proposalId you want to query!
Which returns your proposal: 
Fetched Proposal: { id: 1, description: 'Do you love Secret?', quorum: 1, voteCount: 0 }
Each time you create a proposal, the proposalId is incremented by 1. Your first proposalId is 1, your next proposalIdwill be 2, and so on. 
Vote on Proposal
Now it's time to vote on the proposal you created. Open vote.js and update your proposal answer to either "yes" or "no" in the msg object: 
let msg = {
    answer: "yes",
    proposal_id: proposal.id,
    proposal_description: proposal.description,
    salt: Math.random(),
  };
proposal.id and proposal.description will match the proposal info you input for getProposalById. 
This means that each time you vote, you need to make sure you update the proposal_id number that you pass to getProposalById() so that it matches the proposal you want to vote on! 
let proposal = await getProposalById(1);
Once you have updated your vote and proposalId, execute the vote script: 
npx hardhat --network polygon run ./scripts/vote.js
Your encrypted data and transaction hash are returned upon successful execution: 
Encrypted data: Uint8Array(120) [
  115,  69,  78, 152,  84,  64, 134,  83, 152, 110,  15, 162,
   90, 131,  84,  73, 128, 158, 159,  39, 103,   8, 131, 246,
   61,  95, 230, 131, 220,  79,  25,  68, 203, 174, 180, 168,
  244,  71, 125, 190,  46, 173, 207, 217, 150, 249, 150, 223,
   69, 229,  64,  98, 255, 145, 141, 136, 158, 181,  97, 137,
  148,  71,  25, 213, 184, 165, 116, 224,  80, 201, 138, 211,
    3, 112, 237, 103, 209,  77, 200,  23,  52, 178, 220, 147,
  143, 153, 120, 151,  74, 140, 137, 174,  86,   3,  38, 200,
   64, 197, 168, 165,
  ... 20 more items
]
Transaction hash: 0xd69235d34c0c326cf224661264035feec453eacd5749cbabdf7a018b6285d4f2
vote function executed successfully!
Decrypt Votes
Now it's time to decrypt your vote! First, query that the vote was successfully added to the proposal by running query_proposal_votes.js:  
Be sure to update the proposalId with the proposal you want to query. 
npx hardhat --network polygon run ./scripts/query_proposal_votes.js
query_proposal_vote returns your encrypted vote for the supplied proposalId: 
[
  '0x73454e9854408653986e0fa25a835449809e9f27670883f63d5fe683dc4f1944cbaeb4a8f4477dbe2eadcfd996f996df45e54062ff918d889eb56189944719d5b8a574e050c98ad30370ed67d14dc81734b2dc938f9978974a8c89ae560326c840c5a8a5be6dc79233f0bd9ed378e18ab56aa447e8c46883'
]
Run decrypt.js to decrypt the vote: 
npx hardhat --network polygon run ./scripts/decrypt.js
In decrypt.js, update the proposalId with the proposal you want to query. 
Which returns your decrypted vote:
{
  votes: [
    '{"answer":"yes","proposal_id":1,"proposal_description":"Do you love Secret?","salt":0.20849165534651148}'
  ]
}
Conclusion
Congrats! You have now deployed smart contracts on Polygon and Secret Network and implemented private cross-chain voting. If you have any questions or run into any issues, post them on the Secret Developer Discord and somebody will assist you shortly. 
