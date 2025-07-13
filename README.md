// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DouDiZhu {
    struct Player {
        address playerAddress;
        uint8[] handCards; // 玩家手中的牌
        bool isLandlord;   // 是否是地主
    }

    Player[3] public players; // 三个玩家
    uint8[] public deck;       // 牌组
    uint8[] public landCards;  // 底牌
    uint8 public currentPlayer; // 当前出牌玩家
    uint8 public landlordIndex; // 地主索引

    constructor(address[3] memory playerAddresses) {
        for (uint8 i = 0; i < 3; i++) {
            players[i] = Player({
                playerAddress: playerAddresses[i],
                handCards: new uint8[](0),
                isLandlord: false
            });
        }
        initializeDeck();
    }

    function initializeDeck() internal {
        for (uint8 i = 1; i <= 54; i++) {
            deck.push(i); // 1-54代表一副牌
        }
        shuffleDeck();
    }

    function shuffleDeck() internal {
        for (uint8 i = 0; i < deck.length; i++) {
            uint8 j = uint8(uint256(keccak256(abi.encodePacked(block.timestamp, msg.sender))) % deck.length);
            (deck[i], deck[j]) = (deck[j], deck[i]); // 交换
        }
    }

    function dealCards() public {
        require(msg.sender == players[0].playerAddress, "Only the first player can deal cards");
        for (uint8 i = 0; i < 3; i++) {
            for (uint8 j = 0; j < 17; j++) {
                players[i].handCards.push(deck.pop());
            }
        }
        // 留底牌
        landCards = new uint8[](3);
        for (uint8 i = 0; i < 3; i++) {
            landCards[i] = deck.pop();
        }
    }

    function callLandlord(uint8 playerIndex) public {
        require(msg.sender == players[playerIndex].playerAddress, "Not your turn");
        require(!players[playerIndex].isLandlord, "Already landlord");
        
        // 逻辑处理
        players[playerIndex].isLandlord = true;
        landlordIndex = playerIndex;
    }

    function playCard(uint8[] memory cards) public {
        require(msg.sender == players[currentPlayer].playerAddress, "Not your turn");
        // 出牌逻辑
        currentPlayer = (currentPlayer + 1) % 3; // 轮到下一个玩家
    }

    function checkWin() public view returns (bool) {
        for (uint8 i = 0; i < 3; i++) {
            if (players[i].handCards.length == 0) {
                return true; // 游戏结束
            }
        }
        return false;
    }
}
