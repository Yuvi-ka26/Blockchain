I used REMIX IDE to run these codes. 
Main Code files are provided within the Abhishek_Blockchain folder 
--- 
// SPDX-License-Identifier: MIT 
pragma solidity 0.8.26; 
--- 
 
=>These lines refers to the license of solidity and version of solidity language 
 
--- 
import "@openzeppelin/contracts/token/ERC20/IERC20.sol"; 
import "@openzeppelin/contracts/access/Ownable.sol"; 
import "@openzeppelin/contracts/utils/math/SafeMath.sol"; 
--- 
 
=> IERC20: Interface for interacting with ERC20 tokens. 
 
=> Ownable: Provides ownership and access control functionality. 
 
=> SafeMath: Provides safe mathematical operations to prevent overflow and underflow. 
 
--- 
contract VestingContract is Ownable { 
using SafeMath for uint256; 
IERC20 public token; 
uint256 public totalAllocatedTokens; 
uint256 public start; 
 
enum Role { User, Partner, Team } 
--- 
 
=> VestingContract is a smart contract written in Solidity for managing the vesting. 
=> Oenable: Is the access control, only the contract owner can start vesting and add beneficiaries. 
=> SafeMath helps in initializing mathematical terms in code. 
 
=> token: The ERC20 token to be distributed. 
=> totalAllocatedTokens: Total number of tokens allocated for vesting. 
=> start: The timestamp when vesting starts. 
=> Role: Enum defining three roles: User, Partner, Team. 
 
Structs and Mappings:=> 
 
--- 
struct VestingSchedule { 
uint256 totalAmount; 
//totalAmount: The total number of tokens allocated to the beneficiary. 
uint256 cliff; 
//cliff: The minimum time that must pass before any tokens can be released. 
uint256 duration; 
//duration: The total period over which the tokens will vest. 
uint256 released; 
//released: The amount of tokens that have been already released. 
} 
--- 
 
VestingSchedule: Struct defining the vesting terms (total amount, cliff period, duration, released amount). 
 
--- 
mapping(address => VestingSchedule) public beneficiaries; 
mapping(address => Role) public roles; 
--- 
 
beneficiaries: Mapping from addresses to their vesting schedules. 
 
roles: Mapping from addresses to their assigned roles. 
 
Events:=> 
 
--- 
event VestingStarted(uint256 start); 
event BeneficiaryAdded(address indexed beneficiary, Role role, uint256 amount); 
event TokensReleased(address indexed beneficiary, uint256 amount); 
--- 
 
VestingStarted: Emitted when vesting begins. 
BeneficiaryAdded: Emitted when a new beneficiary is added. 
TokensReleased: Emitted when tokens are released to a beneficiary. 
 
Functions:=> 
 
Constructor: 
 
--- 
constructor(address _token, uint256 _totalAllocatedTokens) Ownable(msg.sender) { 
token = IERC20(_token); 
totalAllocatedTokens = _totalAllocatedTokens; 
} 
--- 
Initializes the contract with the token address and total allocated tokens. 
 
startVesting: 
 
--- 
function startVesting() external onlyOwner { 
require(start == 0, "Vesting already started"); 
start = block.timestamp; 
emit VestingStarted(start); 
} 
--- 
 
Starts the vesting process and sets the start timestamp. Only callable by the owner. 
 
addBeneficiary: 
 
--- 
function addBeneficiary(address _beneficiary, Role _role, uint256 _amount) external onlyOwner { 
require(start == 0, "Cannot add beneficiaries after vesting has started"); 
require(beneficiaries[_beneficiary].totalAmount == 0, "Beneficiary already added"); 
 
uint256 cliff; 
uint256 duration; 
 
if (_role == Role.User) { 
require(_amount == totalAllocatedTokens.mul(50).div(100), "Invalid amount for User role"); 
cliff = 305 days; 
duration = 730 days; 
} else if (_role == Role.Partner) { 
require(_amount == totalAllocatedTokens.mul(25).div(100), "Invalid amount for Partner role"); 
cliff = 60 days; 
duration = 365 days; 
} else if (_role == Role.Team) { 
require(_amount == totalAllocatedTokens.mul(25).div(100), "Invalid amount for Team role"); 
cliff = 60 days; 
duration = 365 days; 
} 
 
beneficiaries[_beneficiary] = VestingSchedule({ 
totalAmount: _amount, 
cliff: cliff, 
duration: duration, 
released: 0 
}); 
roles[_beneficiary] = _role; 
 
emit BeneficiaryAdded(_beneficiary, _role, _amount); 
} 
--- 
 
Adds a new beneficiary with a specific role and amount of tokens. Each role has specific token allocation, 
cliff, and duration. 
 
releaseTokens: 
 
--- 
function releaseTokens(address _beneficiary) external { 
require(start != 0, "Vesting has not started"); 
VestingSchedule storage schedule = beneficiaries[_beneficiary]; 
require(schedule.totalAmount > 0, "No vesting schedule for beneficiary"); 
 
uint256 vestedAmount = _vestedAmount(schedule); 
uint256 unreleased = vestedAmount.sub(schedule.released); 
 
require(unreleased > 0, "No tokens to release"); 
 
schedule.released = schedule.released.add(unreleased); 
token.transfer(_beneficiary, unreleased); 
 
emit TokensReleased(_beneficiary, unreleased); 
} 
--- 
 
Releases the vested tokens to the beneficiary. Calculates the vested amount based on time passed and 
transfers the unreleased tokens. 
 
_vestedAmount: 
 
--- 
function _vestedAmount(VestingSchedule memory schedule) internal view returns (uint256) { 
//block.timestamp: Significes the current time. 
if (block.timestamp < start.add(schedule.cliff)) { 
return 0; 
} else if (block.timestamp >= start.add(schedule.duration)) { 
return schedule.totalAmount; 
} else { 
return schedule.totalAmount.mul(block.timestamp.sub(start)).div(schedule.duration); 
} 
--- 
 
Internal function to calculate the vested amount based on the vesting schedule.
