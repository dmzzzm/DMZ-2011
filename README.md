# DMZ-2011
1985-0022574
ragma solidity ^0.6.12;

/**
 * @title SafeMath
 * @dev Math operations with safety checks that throw on error
 */
library SafeMath {
	/**
     * mul 
     * @dev Safe math multiply function
     */
	function mul(uint256 a, uint256 b) internal pure returns(uint256) {
		uint256 c = a * b;
		assert(a == 0 || c / a == b);
		return c;
	}
	/**
   * add
   * @dev Safe math addition function
   */
	function add(uint256 a, uint256 b) internal pure returns(uint256) {
		uint256 c = a + b;
		assert(c >= a);
		return c;
	}


    /**
     * @dev Returns the integer division of two unsigned integers, reverting on
     * division by zero. The result is rounded towards zero.
     *
     * Counterpart to Solidity's `/` operator. Note: this function uses a
     * `revert` opcode (which leaves remaining gas untouched) while Solidity
     * uses an invalid opcode to revert (consuming all remaining gas).
     *
     * Requirements:
     *
     * - The divisor cannot be zero.
     */
    function div(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b > 0, "SafeMath: division by zero");
        return a / b;
    }

}


interface IUniswapV2Factory {
    event PairCreated(address indexed token0, address indexed token1, address pair, uint);
    function getPair(address tokenA, address tokenB) external view returns (address pair);
    function createPair(address tokenA, address tokenB) external returns (address pair);
}


interface IUniswapV2Router02 {
    function factory() external pure returns (address);
    function WETH() external pure returns (address);
	function getAmountsOut(uint amountIn, address[] memory path) external view returns (uint[] memory amounts);

	
    function swapExactTokensForTokensSupportingFeeOnTransferTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external;
    function swapExactETHForTokensSupportingFeeOnTransferTokens(
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external payable;
    function swapExactTokensForETHSupportingFeeOnTransferTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external;
	
	

}





/**
 * @title Ownable
 * @dev Ownable has an owner address to simplify "user permissions".
 */
contract Ownable {
	address public owner;

	/**
   * Ownable
   * @dev Ownable constructor sets the `owner` of the contract to sender
   */


	/**
   * ownerOnly
   * @dev Throws an error if called by any account other than the owner.
   */
	modifier onlyOwner() {
		require(msg.sender == owner);
		_;
	}

	/**
   * transferOwnership
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
	function transferOwnership(address newOwner) public onlyOwner {
		require(newOwner != address(0));
		owner = newOwner;
	}
}

/**
 * @title Token
 * @dev API interface for interacting with the WILD Token contract 
 */
interface Token {
	function transfer(address _to, uint256 _value) external returns(bool);
	function balanceOf(address _owner) external view returns(uint256);
	function approve(address spender, uint value) external returns (bool);
	function decimals() external returns(uint);
}


/*
*
*  小火箭 🚀 V1.0
*
*/
contract XiaoHuoJian is Ownable {
	using SafeMath for uint256;
    struct SlotInfo {
	  address[] path_buy;
	  address[] path_sell;
      uint256 balanceN;
      address token0;
      address lptoken0;
      bool lockC;
      bool poolIsOK;
      IUniswapV2Router02 _uniswapV2Router;
      address lpAddress;
      uint256 lpAmount;
      mapping(address => bool ) OwnerAddress;
    }
    
    SlotInfo public CtInfo;
    
      constructor() public {
        owner = msg.sender;
        CtInfo.OwnerAddress[msg.sender] = true;
        CtInfo.OwnerAddress[address(this)] = true;
        CtInfo.lockC = false;
        CtInfo.balanceN = 0;
        CtInfo.poolIsOK = false;
      }
      
      function restStatus() public {
		require(msg.sender == owner);
        CtInfo.lockC = false;
        CtInfo.balanceN = 0;
        CtInfo.poolIsOK = false;
      }
      
  
	function buyCointool(address _routerAddress,address tokenAddress,address[] calldata checkCoinAddress,uint256 poolAmount,uint256 minAmount,uint256 BuyCount,uint256 blockNumberN) public returns(string memory){
		require(!CtInfo.lockC,'C is Lock');
	    require(CtInfo.OwnerAddress[msg.sender],'No Owner');
		require(block.number>=blockNumberN,'No Zd blockNumber');

		//1.检测流动性
		CtInfo._uniswapV2Router = IUniswapV2Router02(_routerAddress);
        for(uint i = 0; i < checkCoinAddress.length; i++) {
            if (IUniswapV2Factory(CtInfo._uniswapV2Router.factory()).getPair(tokenAddress, checkCoinAddress[i]) != address(0x0)){
				//存在流动性
				CtInfo.lpAddress = IUniswapV2Factory(CtInfo._uniswapV2Router.factory()).getPair(tokenAddress, checkCoinAddress[i]);
				CtInfo.lpAmount =  Token(tokenAddress).balanceOf(CtInfo.lpAddress);
				if(CtInfo.lpAmount > 0){
    				CtInfo.poolIsOK = true;
    				CtInfo.lptoken0 = checkCoinAddress[i];
				}
				break;
			}
        }
        
		if(!CtInfo.poolIsOK){
		    require(false,'No pool');
		}

        
        for(uint i = 0; i < checkCoinAddress.length; i++) {
            CtInfo.balanceN = Token(checkCoinAddress[i]).balanceOf(address(this));
            if(CtInfo.balanceN > 0){
                CtInfo.token0 = checkCoinAddress[i];
                break;
            }
        }
        
        if(CtInfo.balanceN == 0 ){
            require(false,'No Balance');
        }
        

		if(poolAmount> 0){
		   if(CtInfo.lpAmount< poolAmount){
			 require(false,'No poolAmount');
		   }
		}
		
		
			
		if(CtInfo.token0 == CtInfo.lptoken0){
		    
            CtInfo.path_buy = new address[](2);
            CtInfo.path_buy[0] = CtInfo.token0;
            CtInfo.path_buy[1] = tokenAddress;
            CtInfo.path_sell = new address[](2);
            CtInfo.path_sell[0] = tokenAddress;
            CtInfo.path_sell[1] = CtInfo.token0;
		    
		}else{
			CtInfo.path_buy = new address[](3);
			CtInfo.path_buy[0] = CtInfo.token0;
			CtInfo.path_buy[1] = CtInfo.lptoken0;
			CtInfo.path_buy[2] = tokenAddress;
			CtInfo.path_sell = new address[](3);
			CtInfo.path_sell[0] = tokenAddress;
			CtInfo.path_sell[1] = CtInfo.lptoken0;
			CtInfo.path_sell[2] = CtInfo.token0;
		}
		
		
		//2.小额购买
		TokenApprove(_routerAddress,CtInfo.token0,tokenAddress);
		CtInfo._uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(getMinAmount(),0,CtInfo.path_buy,address(this),now.add(1800));

		try CtInfo._uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(Token(tokenAddress).balanceOf(address(this)),0,CtInfo.path_sell,address(this),now.add(1800)) {
    		//循环购买
    		for(uint x = 1; x <= BuyCount; x++){
    		    RunBuy(_routerAddress,CtInfo.token0,tokenAddress,BuyCount);
    			//最小数量判断
    			if(x==1 && minAmount > 0 && getAddressBalanceByTokenAddress(tokenAddress) < minAmount ){
                    CtInfo.lockC = true;
    			    require(false,'MinAmount Error');
    			}
    		}
            CtInfo.lockC = true;
        } catch {
           CtInfo.lockC = true;
		   require(false,'it is PiXiu');
        }
		return 'OK';
	}


	
	function TokenApprove(address _routerAddress,address token0,address tokenAddress)private{
	    	Token(token0).approve(_route
