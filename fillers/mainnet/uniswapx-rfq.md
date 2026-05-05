# UniswapX RFQ

UniswapX RFQ V2 is designed to improve price execution and minimize quoter gaming by redesigning the flow of order quote information. The primary goal is to make trading on Mainnet Ethereum more efficient and more reliable.

### **RFQ Overview** <a href="#rfq-overview" id="rfq-overview"></a>

UniswapX RFQ is architected such that the time between a quoter submitting their quote and being required to fill the order is minimized. This enforces a near 100% fill rate and causes most orders to fill almost instantly.

To achieve this, the quoting flow is divided into two phases:

1. **Indicative Quotes**: Provided pre-signature when the swapper is exploring quotes but has not committed to trading. Quoters can participate here without being held fully accountable.
2. **Hard Quotes**: Provided post-signature when the swapper has committed to trading. Quoters are held fully accountable for these quotes.

Since hard quotes are provided post-signature, they are instantly executable by fillers removing the risk of market movements causing the order to fail.

To prevent gaming, quoters cannot distinguish between indicative and hard quotes. As a result, they must always assume all quotes are hard and provide competitive prices.

### V2 Flow <a href="#v2-flow" id="v2-flow"></a>

<div align="center"><img src="https://docs.uniswap.org/assets/images/v2-flow-1c31af9e8e1c805e48a4ee0f1b51ab32.png" alt="image" width="599"></div>

1. The user requests a quote from the interface.
2. The Uniswap Backend (URA) runs an indicative RFQ auction, soliciting quotes for the best price. (**Note:** Quoters do not know whether the RFQ is indicative or hard, forcing them to always provide competitive prices.)
3. The quotes from the RFQ process parameterize an [initial quote](https://github.com/Uniswap/UniswapX/blob/33fa564cfaa6d58f6e3fcf7e7988cb5fc1c61de7/src/lib/V2DutchOrderLib.sol#L31) order shown to the user.
4. The user signs the order and submits it for execution.
5. The Uniswap Backend (GPA) runs a second “hard” RFQ process, soliciting new quotes.
6. Quoters return their best prices again.
   * If the price is within or improves on the user’s signed parameters, it is finalized and added to the order’s [CosignerData](https://github.com/Uniswap/UniswapX/blob/33fa564cfaa6d58f6e3fcf7e7988cb5fc1c61de7/src/lib/V2DutchOrderLib.sol#L20).
   * If the price moves outside the signed parameters, the order fails, and the user must try again.
7. The finalized order containing CosignerData is posted to the filler network for execution.
8. The winning filler executes the order.

### Why is a cosigner needed? <a href="#why-is-a-cosigner-needed" id="why-is-a-cosigner-needed"></a>

In UniswapX RFQ, users commit to a range of prices when signing their order. Without safeguards, a malicious auctioneer could provide users the worst price within their range.

<div align="center"><img src="https://docs.uniswap.org/assets/images/cosigner-38a7de719736b3c6a9eabb0f05091c7e.png" alt="image" width="599"></div>

The cosigner field allows users to designate an auctioneer they trust to run the auction fairly, ensuring the best executable price within the signed parameters. Currently, the trading API sets the cosigner to Uniswap Labs, though this could be updated in the future.

### Current Status <a href="#current-status" id="current-status"></a>

UniswapX RFQ V2 is currently the default version of the protocol running on Mainnet across Uniswap's interfaces.

&#x20;interfaces.

<details>

<summary>How did previous versions of UniswapX RFQ work?</summary>

In V1, the RFQ flow operated entirely pre-signature. Users requested a swap quote, which was parameterized using an RFQ quote. If they accepted the quote, they signed the order. This approach was straightforward but caused quoters in the network to be uncertain about when a quote request would convert into a signed order. This uncertainty led to occasional unfilled orders and degraded performance.

\
![image](https://docs.uniswap.org/assets/images/v1-flow-c19ebe2f784563f62c47cab02cd10cfe.png)

</details>

### Smart Contracts <a href="#smart-contracts" id="smart-contracts"></a>

* [V2DutchOrderReactor](https://github.com/Uniswap/UniswapX/blob/main/src/reactors/V2DutchOrderReactor.sol)
