# Web-3.0-Assignment-2-
ERC20, ERC721 &amp; NFT Staking Contract

//SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";


contract RewardToken is ERC20, ERC20Burnable, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor() ERC20("RewardToken", "MTK") {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }
}

SwordFISH.sol
//SPDX-License-Identifier:MIT;
pragma solidity ^0.8.4;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract SwordFISH is ERC721, ERC721Burnable, AccessControl {
    using Counters  for Counters.Counter;

    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    Counters.Counter private _tokenIdCounter;

    constructor () ERC721("SwordFISH", "SF") {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(MINTER_ROLE, msg.sender);
    }

    function baseURI() internal pure override returns (string memory) {
        return "ipfs::/ipfs_link/";
    }
   
     function safeMint(address to) public onlyRole(MINTER_ROLE) {
         uint256 tokenId = _tokenIdCounter.current();
         _tokenIdCounter.increment();
         _safeMint(to, tokenId);
     }
// The following functions are overrides required by solidity.

    function supportInterface (bytes4 InterfaceId)
    public
    view
    override(ERC721, AccessControl)
    returns (bool)
    {
    return super.supportsInterface(InterfaceId);
    }

    interface IRewardToken is IERC20 {
     function mint(address to, uint256 amount) external;
    }

}
NFT Stakng.sol
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

    import "@openzeppelin/contracts/token/ERC721/IERC721.sol";
    import "@openzeppelin/contracts/token?ERC20/IERC20.sol";
    import "@openzeppeiln/contracts/utils/math/SafeMath.sol";
    import "@openzeppelin/contracts/access/Ownable.sol";
    import "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";

    interface IRewardToken is IERC20 {
        function mint(address to, uint256 amount) external;
    }

    uint256 public stakedTotal;
    uint256 public stakingStartTime;
    uint256 constant stakingTime = 180 seconds;
    uint256 constant token = 10e18;
    }

    struct Staker {
        uint256[] tokenIds;
        mapping(uint256 => uint256) tokenStakingCoolDown;
        uint256 balance;
        uint256 rewardReleased;
    }
    
      //mapping of a staker to its wallet
       mapping(address => Staker) public Stakers;

      //mapping from Token ID to owner address
      mapping(uint256 => address) public tokenOwner;
      bool public tokensClaimable;
      bool initialized;

      event Staked(address owner, uint256 amount);
      event Unstaked(address owner, uint256 amount);
      event RewardPaid(address indexed user, uint256 reward);
      event ClaimableStatusUpdated(bool status);
      event EmergencyUnstake(address indexed user, uint256 tokenId);

    function initStaking public owner {
        //needs access control
        require(!initialised, "Already initialized");
        stakingStartTime = block.timestamp;
        intialized = true;

    }
 
    function setTokensClaimable(bool _enabled) public onlyOwner {
        //needs access control
        tokensClaimable = _enabled;
        emit ClaimableStatusUpdated(_enabled);
    }

    function getStakedToken(address _user)
    public 
    view 
    returns(uint256[] memory tokenIds)
    {
        returns Stakers[_user].tokenIds;
    }

    function stake(uint256 tokenId) public {
        _stake(msg.sender, tokenId);
    }
    
    function stakeBatch(uint256[] memory tokenIds) public {
        for (uint256 i = 0; i < tokenIds.length; i++) {
            _stake(msg.sender, tokenIds[i]);
        }
    }

    function _stake(address _user, uint256 _tokenId) internal {
        require(intialised, "Staking System: the staking has not started");
        require(
            nft.ownerOf(_tokenId) == _user,
            "user must be owner of the token"
        );
        Staker storage staker = stakers[_user];

        staker.tokenIds.push(_tokenId);
        staker.tokenStakingCoolDown[_tokenId] = block.timestamp;
        tokenOwner[_tokenId] = _user;
        nft.approve(address(this), _tokenId);
        nft.safeTransferFrom(_user, address(this), _tokenId);


        emit Staked(_user, _tokenId);
        stakedTotal++;
    }
    
    function _unstake(address _user, uint256 tokenId) internal {
        require(
            tokenOwner[_tokenId] == _user;
            "Nft Staking System: user must be the owner of the staked nft"
        );
        Staker storage staker = stakers[_user];

        uint256 lastIndex = staker.tokenIds.length - 1;
        uint256 lastIndexKey = staker.tokenIds[lastIndex];
        if (staker.tokensIds.length > 0) {
            staker.tokenIds.pop();
        }

        staker.tokenStakingCoolDown[_tokenId] = 0;
        if (staker.balance == 0) {
            delete stakers[_user];
        }
        delete tokenOwner[_tokenId];

        nft.safeTransferFrom(address(this), _user, _tokenId);

        emit Unstaked(_user, _tokenId);
        stakedTotal--;
    }

    function _unstake(address _user, uint256 tokenId) internal {
        require(
            tokenOwner[_tokenId] == _user;
            "Nft Staking System: user must be the owner of the staked nft"
        );
        Staker storage staker = stakers[_user];

        uint256 lastIndex = staker.tokenIds.length - 1;
        uint256 lastIndexKey = staker.tokenIds[lastIndex];
        if (staker.tokensIds.length > 0) {
            staker.tokenIds.pop();
        }

        staker.tokenStakingCoolDown[_tokenId] = 0;
        if (staker.balance == 0) {
            delete stakers[_user];
        }
        delete tokenOwner[_tokenId];

        nft.safeTransferFrom(address(this), _user, _tokenId);

        emit Unstaked(_user, _tokenId);
        stakedTotal--;
    }

    function updateReward(address _user) public {
       
        Staker storage staker - stakers[_user];
        uint256[] storage ids = staker.tokenIds;
        for (uint256 i = 0; i < ids.length; i++) {
         if (
            staker.tokenStakingCoolDown[ids[i]] <
            block.timestamp + stakingTime &&
            staker.tokenStakingCoolDown[ids[i]]> 0;
        ) {

            uint256 stakedDays - ((block.timestamp - uint(stake.tokenStakingCoolDown[ids[i]]))) / stakingTime;
            uint256 partialTime = ((block.timestamp - uint(stake.tokenStakingCoolDown[ids[i]]))) % stakingTime;

            staker.balance +- token * stakedDays;

            staker.tokenStakingCoolDown[ids[i]] = block.timestamp + partialTime;
             
            console.loguint(staker.tokenStakingCoolDown[ids[i]]);
            console.loguint(staker.balance);
        }
        }
  
    
    }

    function claimReward(address _user) public {
         require(tokensClaimable == true, "Tokens cannot be claimed yet");
         require(stakers[_user].balance > 0 , "0 rewards yet");

         stakers[_user].rewardsReleased += stakers[_user].balance;
         stakers[_user].balance = 0;
         rewardsToken.mint(_user, stakers[_user].balance);

         emit RewardPaid(_user, stakers[_user].balance);
    } 
     
    
