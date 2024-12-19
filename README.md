// SPDX-License-Identifier: MIT

pragma solidity 0.8.26;

struct Bet {
    uint amount;
    uint candidate;
    uint timestamp;
    uint claimed;
}

struct Dispute {
    string candidate1;
    string candidate2;
    string image1;
    string image2;
    uint total1;
    uint total2;
    uint winner;
}

contract BetCandidate {

    Dispute public dispute;
    mapping (address => Bet) public allBets;

    address owner;
    uint fee = 1000;//% escala 4 zeros
    uint public netPrize;
   
    constructor(){
        owner = msg.sender;
        dispute = Dispute({
           candidate1: "Donald Trump",
           candidate2: "Kamala Harris",
           image1: "https://media.wired.com/photos/64e8dfe4cedb032afb4ceccd/master/pass/Donald-Trump-Mugshot-Depth-Of-Field-Culture-1621335357.jpg",
           image2: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRMuqzX2Sy0wPYpDSgjdrFAbGQ9h8Bevzmu8A&s",
           total1: 0,
           total2: 0,
           winner: 0
        });
    
    }

    function bet(uint candidate) external payable {
         require(candidate == 1 || candidate == 2, "invalid candidate");
         require(msg.value > 0, "invalid bet");
         require(dispute.winner == 0, "dispute closed");

         Bet memory newBet;
         newBet.amount = msg.value;
         newBet.candidate = candidate;
         newBet.timestamp = block.timestamp;

         allBets[msg.sender] = newBet;

         if (candidate ==1)
            dispute.total1 += msg.value;
         else 
            dispute.total2 += msg.value;
    }

    function finish(uint winner) external {
        require(msg.sender == owner, "Invalid account");
        require(winner == 1 || winner == 2, "Invalid candidate");
        require(dispute.winner == 0, "Dispute closed");

        dispute.winner = winner;

        uint grossPrize = dispute.total1 + dispute.total2;
        uint commission = (grossPrize * fee) / 1e4;
        netPrize = grossPrize - commission;

        payable (owner).transfer (commission);
    }

   function claim () external {
        Bet memory userBet = allBets [msg.sender];
        require(dispute.winner > 0 && dispute.winner == userBet.candidate && userBet.claimed == 0, "invalid claim" );

        uint winnerAmount = dispute.winner == 1 ? dispute.total1 : dispute.total2;
        uint ratio = (userBet.amount * 1e4) / winnerAmount;
        uint individualPrize = netPrize * ratio /1e4;
        allBets [msg.sender].claimed = individualPrize;
        payable (msg.sender).transfer (individualPrize);
    }
}
