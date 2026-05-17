# ApprovalManager
BaseApprovalManager
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts/access/Ownable.sol";

contract BaseApprovalManager is Ownable {

    struct ApprovalInfo {
        address token;
        address owner;
        address spender;
        uint256 amount;
    }

    event ApprovalRevoked(address indexed token, address indexed owner, address indexed spender);
    event AllApprovalsRevoked(address indexed owner);

    constructor() Ownable(msg.sender) {}

    // Query single approval
    function getApproval(address token, address owner, address spender) external view returns (uint256) {
        return IERC20(token).allowance(owner, spender);
    }

    // Batch query
    function getApprovalsBatch(
        address owner,
        address[] calldata tokens,
        address[] calldata spenders
    ) external view returns (ApprovalInfo[] memory) {
        require(tokens.length == spenders.length, "Length mismatch");
        
        ApprovalInfo[] memory result = new ApprovalInfo[](tokens.length);
        
        for (uint256 i = 0; i < tokens.length; i++) {
            result[i] = ApprovalInfo({
                token: tokens[i],
                owner: owner,
                spender: spenders[i],
                amount: IERC20(tokens[i]).allowance(owner, spenders[i])
            });
        }
        return result;
    }

    // Revoke single approval
    function revokeApproval(address token, address spender) external {
        IERC20(token).approve(spender, 0);
        emit ApprovalRevoked(token, msg.sender, spender);
    }

    // Batch revoke
    function revokeApprovalsBatch(
        address[] calldata tokens,
        address[] calldata spenders
    ) external {
        require(tokens.length == spenders.length, "Length mismatch");
        
        for (uint256 i = 0; i < tokens.length; i++) {
            IERC20(tokens[i]).approve(spenders[i], 0);
            emit ApprovalRevoked(tokens[i], msg.sender, spenders[i]);
        }
    }

    // One-click revoke common tokens on Base
    function revokeCommonApprovals() external {
        address[4] memory commonTokens = [
            0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913, // USDC
            0x4200000000000000000000000000000000000006, // WETH
            0x50c5725949A6F0c72E6C4a641F24049A917DB0Cb, // DAI
            0x2Ae3F1Ec7F1F5012CFEab0185bfc7aa3cf0DEc22  // cbETH
        ];

        for (uint256 i = 0; i < commonTokens.length; i++) {
            IERC20(commonTokens[i]).approve(msg.sender, 0);
        }
        
        emit AllApprovalsRevoked(msg.sender);
    }

    // Check if approval is infinite
    function isInfiniteApproval(address token, address owner, address spender) external view returns (bool) {
        return IERC20(token).allowance(owner, spender) == type(uint256).max;
    }
}

interface IERC20 {
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
}
