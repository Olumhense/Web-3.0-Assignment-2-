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
    

