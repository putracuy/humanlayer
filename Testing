// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

// ERC20 token standard
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

// EnumerableSet library for managing sets of addresses
import "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";


contract WeightedVoting is ERC20 {
    using EnumerableSet for EnumerableSet.AddressSet;

    // --- State Variables ---
    uint256 public constant maxSupply = 1_000_000;
    
    // Tracks which addresses have claimed tokens to prevent multiple claims.
    mapping(address => bool) private hasClaimed;

    // --- Errors ---
    error TokensClaimed();
    error AllTokensClaimed();
    error NoTokensHeld();
    error QuorumTooHigh(uint256 proposedQuorum);
    error AlreadyVoted();
    error VotingClosed();

    // --- Structs & Enums ---
    // The main data structure for each voting issue.
    // The order of these variables is crucial for the unit tests.
    struct Issue {
        EnumerableSet.AddressSet voters;
        string issueDesc;
        uint256 votesFor;
        uint256 votesAgainst;
        uint256 votesAbstain;
        uint256 totalVotes;
        uint256 quorum;
        bool passed;
        bool closed;
    }

    // A separate struct to return issue data, as EnumerableSet cannot be returned.
    struct IssueData {
        address[] voters;
        string issueDesc;
        uint256 votesFor;
        uint256 votesAgainst;
        uint256 votesAbstain;
        uint256 totalVotes;
        uint256 quorum;
        bool passed;
        bool closed;
    }

    // A mapping from an issue ID to the issue data.
    Issue[] private issues;

    // Enum to represent the different vote options.
    enum Vote {
        AGAINST,
        FOR,
        ABSTAIN
    }

    // --- Constructor ---
    constructor() ERC20("WeightedVoting", "WV") {
        // As per the exercise requirements, we add a dummy element at index 0.
        // This ensures all valid issues will start at index 1.
        issues.push();
    }

    // --- Functions ---
    function claim() public {
        if (hasClaimed[msg.sender]) {
            revert TokensClaimed();
        }
        if (totalSupply() >= maxSupply) {
            revert AllTokensClaimed();
        }

        hasClaimed[msg.sender] = true;
        _mint(msg.sender, 100);
    }

    function createIssue(string calldata _issueDesc, uint256 _quorum) external returns (uint256) {
        // Crucial: This check must come before the quorum check to pass the unit tests.
        if (balanceOf(msg.sender) == 0) {
            revert NoTokensHeld();
        }
        if (_quorum > totalSupply()) {
            revert QuorumTooHigh(_quorum);
        }

        uint256 newId = issues.length;
        Issue storage newIssue = issues.push();
        newIssue.issueDesc = _issueDesc;
        newIssue.quorum = _quorum;

        return newId;
    }

    function getIssue(uint256 _issueId) external view returns (IssueData memory) {
        Issue storage issue = issues[_issueId];
        uint256 votersCount = issue.voters.length();
        address[] memory votersArray = new address[](votersCount);

        for (uint256 i = 0; i < votersCount; i++) {
            votersArray[i] = issue.voters.at(i);
        }

        return IssueData({
            voters: votersArray,
            issueDesc: issue.issueDesc,
            votesFor: issue.votesFor,
            votesAgainst: issue.votesAgainst,
            votesAbstain: issue.votesAbstain,
            totalVotes: issue.totalVotes,
            quorum: issue.quorum,
            passed: issue.passed,
            closed: issue.closed
        });
    }

    function vote(uint256 _issueId, Vote _vote) public {
        Issue storage issue = issues[_issueId];
        
        if (issue.closed) {
            revert VotingClosed();
        }
        if (issue.voters.contains(msg.sender)) {
            revert AlreadyVoted();
        }
        
        uint256 tokenBalance = balanceOf(msg.sender);
        if (tokenBalance == 0) {
            revert NoTokensHeld();
        }

        if (_vote == Vote.FOR) {
            issue.votesFor += tokenBalance;
        } else if (_vote == Vote.AGAINST) {
            issue.votesAgainst += tokenBalance;
        } else if (_vote == Vote.ABSTAIN) {
            issue.votesAbstain += tokenBalance;
        }

        issue.totalVotes += tokenBalance;
        issue.voters.add(msg.sender);

        // Check if the quorum has been met or exceeded.
        if (issue.totalVotes >= issue.quorum) {
            issue.closed = true;
            if (issue.votesFor > issue.votesAgainst) {
                issue.passed = true;
            }
        }
    }
}
