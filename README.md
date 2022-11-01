# Ref SDK

This SDK is designed to assist developers when interacting with the main functions of the protocol. Main functions that can be defined as:

* Trade: Swap tokens with our Automated Market Maker (AMM)
* Pool: Add/Remove liquidity and earn revenue from swap fee (Coming soon)
* Farm: Stale LP tokens into farms and earn liquidity incentives (Coming soon)
* Boost Farm: Stake LOVE tokens to get boosted liquidity incentives (Coming soon)
* Stake: Stake REF tokens to earn fees generated by the protocol (Coming soon)
* Vote: Lock REF<>NEAR LP tokens to get veTokens and participate in the governance of the protocol and the allocation of liquidity incentives (Coming soon)

## Install

yarn: `yarn add @ref_finance/ref-sdk`

npm: `npm install @ref_finance/ref-sdk` 

## Initialization

Ref SDK identifies env variable NEAR_ENV to get global configuration. We suggest to use `export NEAR_ENV=mainnet` or `export NEAR_ENV=testnet` to set up the corresponding NEAR network.

```plain
export function getConfig(env: string | undefined = process.env.NEAR_ENV) {
  switch (env) {
    case 'mainnet':
      return {
        networkId: 'mainnet',
        nodeUrl: 'https://rpc.mainnet.near.org',
        walletUrl: 'https://wallet.near.org',
        WRAP_NEAR_CONTRACT_ID: 'wrap.near',
        REF_FI_CONTRACT_ID: 'v2.ref-finance.near',
      };
    case 'testnet':
      return {
        networkId: 'testnet',
        nodeUrl: 'https://rpc.testnet.near.org',
        walletUrl: 'https://wallet.testnet.near.org',
        WRAP_NEAR_CONTRACT_ID: 'wrap.testnet',
        REF_FI_CONTRACT_ID: 'ref-finance-101.testnet',
      };
    default:
      return {
        networkId: 'mainnet',
        nodeUrl: 'https://rpc.mainnet.near.org',
        walletUrl: 'https://wallet.near.org',
        REF_FI_CONTRACT_ID: 'v2.ref-finance.near',
        WRAP_NEAR_CONTRACT_ID: 'wrap.near',
      };
  }
}
```
## Tokens

#### ftGetTokenMetadata

Get token metadata.

Parameters

```plain
id: string;
```
Example
```plain
const WrapNear = await ftGetTokenMetadata('wrap.testnet');
```
Response
```plain
{
  decimals: 24;
  icon: null;
  id: 'wrap.testnet';
  name: 'Wrapped NEAR fungible token';
  reference: null;
  reference_hash: null;
  spec: 'ft-1.0.0';
  symbol: 'wNEAR';
}
```

---
#### ftGetTokensMetadata

Get tokens metadata and set token id as index.

Parameters

```plain
tokenIds: string[]
```
Example
```plain
const tokensMetadata = await ftGetTokensMetadata([
  'ref.fakes.testnet',
  'wrap.testnet',
]);
```
Response
```plain
{
  "ref.fakes.testnet":{
    decimals: 18
    icon: "data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='16 24 248 248' style='background: %23000'%3E%3Cpath d='M164,164v52h52Zm-45-45,20.4,20.4,20.6-20.6V81H119Zm0,18.39V216h41V137.19l-20.6,20.6ZM166.5,81H164v33.81l26.16-26.17A40.29,40.29,0,0,0,166.5,81ZM72,153.19V216h43V133.4l-11.6-11.61Zm0-18.38,31.4-31.4L115,115V81H72ZM207,121.5h0a40.29,40.29,0,0,0-7.64-23.66L164,133.19V162h2.5A40.5,40.5,0,0,0,207,121.5Z' fill='%23fff'/%3E%3Cpath d='M189 72l27 27V72h-27z' fill='%2300c08b'/%3E%3C/svg%3E%0A"
    id: "ref.fakes.testnet"
    name: "Ref Finance Token"
    reference: null
    reference_hash: null
    spec: "ft-1.0.0"
    symbol: "REF"
  },
  "wrap.testnet":{
    decimals: 24
    icon: null
    id: "wrap.testnet"
    name: "Wrapped NEAR fungible token"
    reference: null
    reference_hash: null
    spec: "ft-1.0.0"
    symbol: "wNEAR"
  }
}
```

## Pools

### fetchAllPools

Fetch all existing pools, including vanilla/simple pools, stable pools and rated pools (designed for yield-bearing tokens).

Parameters

None

Example

```plain
const { ratedPools, unRatedPools, simplePools } = await fetchAllPools();
```
Response
```plain
{
  ratedPools:[{
    fee: 5,
    id: 568,
    pool_kind: "RATED_SWAP",
    shareSupply: "80676034815429711745720012070",
    supplies:{
      "meta-v2.pool.testnet": "1298314415249170366960739764"
    	"wrap.testnet": "80182803630538035347294614770"
  	},
    token0_ref_price: undefined,
    tokenIds: ["meta-v2.pool.testnet", "wrap.testnet"],
    tvl: undefined
  },...]
  unRatedPools:[...],
  simplePools:[...],
}
```

---


#### 
### getStablePools

We define `unRatedPools` and `ratedPools` as `stablePool`. You can use this function to get details of stable pools.

Parameters

```plain
stablePools: Pool[]
```
Example
```plain
const stablePools: Pool[] = unRatedPools.concat(ratedPools);

const stablePoolsDetail: StablePool[] = await getStablePools(stablePools);
```
Response
```plain
[
  {
    amounts: ['1298314415249170366960739764', '80182803630538035347294614770'],
    amp: 240,
    c_amounts:["1298314415249170366960739764","80182803630538035347294614770"],
    decimals:[24,24],
    id: 568,
    pool_kind: "RATED_SWAP",
    rates:["1972101024157559347385372","1000000000000000000000000"],
    shares_total_supply: "80676034815429711745720012070",
    token_account_ids:["meta-v2.pool.testnet","wrap.testnet"],
    total_fee:5
  },
	...
]
```

---
## Swap

### estimateSwap

Get token output amount and corresponding route.

As there is a memory limitation on Ledger, note that we set `enableSmartRouting` option for developers.

This function integrates a smart routing algorithm, designed to deliver the best output token amount.

Parameters

```plain
interface SwapParams {
  tokenIn: TokenMetadata;
  tokenOut: TokenMetadata;
  amountIn: string;
  simplePools: Pool[];
  options?: SwapOptions;
}

interface SwapOptions {
  enableSmartRouting?: boolean;
  stablePools?: Pool[];
  stablePoolsDetail?: StablePool[];
}
```
Example (enableSmartRouting == false)
```plain
// enableSmartRouting as FALSE, swap from Ref to wNear, with amount 1
const tokenIn = await ftGetTokenMetadata('ref.fakes.testnet');
const tokenOut = await ftGetTokenMetadata('wrap.testnet');

const swapTodos: EstimateSwapView[] = await estimateSwap({
  tokenIn,
  tokenOut,
  amountIn: '1',
  simplePools,
});
```
Response (enableSmartRouting == false)
```plain
// enableSmartRouting as FALSE, swap from Ref to wNear, with amount 1

[
  {
    estimate: '0.7338604246699393',
    inputToken: 'ref.fakes.testnet',
    outputToken: 'wrap.testnet',
    pool: {
      fee: 30,
      id: 38,
      partialAmountIn: '1000000000000000000',
      pool_kind: 'SIMPLE_POOL',
      shareSupply: '1000587315520795219676332',
      supplies: {
        'ref.fakes.testnet': '7789776060978885018',
        'wrap.testnet': '6467670222256390319335181',
      },
      token0_ref_price: undefined,
      tokenIds: (2)[('ref.fakes.testnet', 'wrap.testnet')],
      tvl: undefined,
    },
  },
];
```
Example (enableSmartRouting == true)
```plain
// enableSmartRouting as TRUE, swap from Ref to wNear, with amount 1
const tokenIn = await ftGetTokenMetadata('ref.fakes.testnet');
const tokenOut = await ftGetTokenMetadata('wrap.testnet');

const options: SwapOptions = {
  enableSmartRouting: true,
  stablePools,
  stablePoolsDetail,
};

const swapTodos: EstimateSwapView[] = await estimateSwap({
  tokenIn,
  tokenOut,
  amountIn: '1',
  simplePools,
  options,
});
```
Response (enableSmartRouting == true)
```plain
// enableSmartRouting as true, swap from Ref to wNear, with amount 1
[
  {
    estimate: "0.000225321544275095902371355566972009167",
    inputToken: "ref.fakes.testnet",
    outputToken: "nusdt.ft-fin.testnet",
    pool:{
      id: 341,
      partialAmountIn: "836859596261755688",
      tokenIds: ["ref.fakes.testnet","nusdt.ft-fin.testnet"],
      ...
    },
    ...
  },{
    estimate: "0.5203255327171591155634413370660973429264",
    inputToken: "nusdt.ft-fin.testnet",
    outputToken: "wrap.testnet",
    pool:{
      id: 1625,
      tokenIds: ["nusdt.ft-fin.testnet","wrap.testnet"],
      ...
    },
    ...
  },{
    estimate: "5.3206339674140066489000963525772735539612",
    inputToken: "ref.fakes.testnet",
    outputToken: "usdn.testnet",
    pool:{
      id: 376,
      tokenIds: ["usdn.testnet","ref.fakes.testnet"],
      partialAmountIn: "163140403738244312",
      ...
    },
    ...
  },{
    estimate: "0.2036473132202839680683206178037243543302",
    inputToken: "usdn.testnet",
    outputToken: "wrap.testnet",
    pool:{
      id: 385,
      tokenIds: ["usdn.testnet","wrap.testnet"],
      ...
    },
    ...
  }
]
```

---
### getExpectedOutputFromSwapTodos

Get token output amount from swapTodos.

Parameters

```plain
(swapTodos: EstimateSwapView[], outputToken: string)
```
Example
```plain
const swapTodos: EstimateSwapView[] = await estimateSwap({
  tokenIn,
  tokenOut,
  amountIn: '1',
  simplePools,
  options,
});

const amountOut:string = getExpectedOutputFromSwapTodos(swapTodos, tokenOut.id);
```
Response
```plain
"0.723972845937443"
```

---


#### 
### getPoolEstimate

Get token output amount from one single pool. Method can be used for simple and stable pools.

Parameters

```plain
{
  tokenIn: TokenMetadata;
  tokenOut: TokenMetadata;
  amountIn: string;
  pool: Pool;
  // please input stablePoolDetail if you want estimate output on stable pool or the pool will be recognized as simple pool
  stablePoolDetail?: StablePool;
}
```
Example (on simple pool)
```plain
// estimate on simple Pool
const estimate = await getPoolEstimate({
  tokenIn,
  tokenOut,
  amountIn: '1',
  pool,
});
```
Response (on simple pool)
```plain
// estimate on simple pool, swap from Ref to wNear, with amount 1
{
  estimate: "0.7338604246699393",
  inputToken: "ref.fakes.testnet",
  outputToken: "wrap.testnet",
  pool:{
    fee: 30,
    id: 38,
    partialAmountIn: "1000000000000000000",
    pool_kind: "SIMPLE_POOL",
    shareSupply: "1000587315520795219676332",
    supplies: {"ref.fakes.testnet": '7789776060978885018', "wrap.testnet": 								'6467670222256390319335181'},
    token0_ref_price: undefined,
    tokenIds: (2) ['ref.fakes.testnet', 'wrap.testnet'],
    tvl: undefined
  }
}
```
Example (on stable pool)
```plain
// estimate on stable pool
const estimate = await getPoolEstimate({
  tokenIn,
  tokenOut,
  amountIn: '1',
  pool: stablePool,
  stablePoolDetail: stablePoolDetail,
});
```
Response (on stable pool)
```plain
// estimate on stable pool, swap from stNear to wNear, with amount 1
{
  estimate: "2.4898866773442284",
  inputToken: "meta-v2.pool.testnet",
  noFeeAmountOut: "2.491132243465961",
  outputToken: "wrap.testnet",
  pool:{
    amounts:["1298314415249170366960739764","80182803630538035347294614770"],
    amp: 240,
    c_amounts:["1298314415249170366960739764","80182803630538035347294614770"],
    decimals:[24,24],
    id: 568,
    pool_kind: "RATED_SWAP",
    rates:["1972204647926836788049038","1000000000000000000000000"],
    shares_total_supply: "80676034815429711745720012070",
    token_account_ids:["meta-v2.pool.testnet", "wrap.testnet"],
    total_fee: 5,
  }
}
```

---


### 
## Transactions

### instantSwap

Set up transactions through swap routes. Please ensure that the AccountId has an active balance storage in the token-in contract, otherwise the transaction will fail and the user will lose the token input amount.

Parameters

```plain
{
  tokenIn: TokenMetadata;
  tokenOut: TokenMetadata;
  amountIn: string;
  slippageTolerance: number;
  swapTodos: EstimateSwapView[];
  AccountId: string;
}
```
Example
```plain
const transactionsRef: Transaction[] = await instantSwap({
  tokenIn,
  tokenOut,
  amountIn: '1',
  swapTodos,
  slippageTolerance = 0.01,
  AccountId: 'your-account-id.testnet'
});
```
Response
```plain
[
  {
    functionCalls: [
      {
        amount: '0.000000000000000000000001',
        args: {
          amount: '1000000000000000000',
          msg:
            '{"force":0,"actions":[{"pool_id":38,"token_in":"ref.fakes.testnet","token_out":"wrap.testnet","amount_in":"1000000000000000000","min_amount_out":"730191122546589600000000"}]}',
          receiver_id: 'ref-finance-101.testnet',
        },
        gas: '180000000000000',
        methodName: 'ft_transfer_call',
      },
    ],
    receiverId: 'ref.fakes.testnet',
  },
];
```

---


#### 
### getSignedTransactionsByMemoryKey (Node)

In the local env, developers can add credentials by `near login` .

This function utilizes credentials stored in the local env to sign transactions.

Parameters

```plain
{
  transactionsRef: Transaction[];
  AccountId: string;
  keyPath: string;
}
```
Example
```plain
const signedTransactions:nearTransactions.SignedTransaction[] = getSignedTransactionsByMemoryKey({
  transactionsRef;
  AccountId: "your-account-id.testnet",
  keyPath: "/.near-credentials/testnet/your-account-id.testnet.json"
})
```
Response
```plain
[
  SignedTransaction {
    transaction: Transaction {
      signerId: 'your-account-id.testnet',
      publicKey: [PublicKey],
      nonce: 91940092000042,
      receiverId: 'ref.fakes.testnet',
      actions: [Array],
      blockHash: <Buffer 45 e5 fd 36 87 3b 10 59 81 d9 a7 b5 20 c7 29 33 f7 27 48 59 06 90 ca 8a 17 03 5c 25 f2 76 ab 7c>
    },
    signature: Signature { keyType: 0, data: [Uint8Array] }
  }
]
```

---
### sendTransactionsByMemoryKey (Node)

This function utilizes credentials stored in the local env to send transactions.

Parameters

```plain
{
  signedTransactions: nearTransactions.SignedTransaction[];
}
```
Example
```plain
sendTransactionsByMemoryKey({
  signedTransactions,
});
```
Response
```plain
[
  {
    receipts_outcome: [
      [Object], [Object],
      [Object], [Object],
      [Object], [Object],
      [Object], [Object],
      [Object], [Object]
    ],
    status: { SuccessValue: 'xxxxxx' },
    transaction: {
      actions: [Array],
      hash: 'xxxxxxxx',
      nonce: 91940092000042,
      public_key: 'ed25519:xxxxxxxx',
      receiver_id: 'ref.fakes.testnet',
      signature: 'ed25519:xxxxxxxxxxxxx',
      signer_id: 'your-account-id.testnet'
    },
    transaction_outcome: {
      block_hash: '3QXEF941UvsHzXrMhwbchk7wmy3qmPNs3w9gkfvW84QK',
      id: 'xxxxxxxx',
      outcome: [Object],
      proof: []
    }
  }
]
```
#  

# Ref Swap Widget

## Description

The Ref Swap Widget is a useful tool, allowing any third party service to access Ref's liquidity. Users of ecosystem dapps have the ability to swap via the Widget, without the need to go to Ref app, thus improving the user experience.

Here are some use cases: 

* Swapping stablecoins for your project's token
* Swapping tokens to lend, farm or stake
* Swapping one token for a specific token, which can be used to buy a NFT in the associated marketplace 

Using the Ref Swap Widget, with a few customizations, developers can integrate the Swap funtion directly into their dapps. Both mobile and/or website version are available.
![图片](https://user-images.githubusercontent.com/50706666/199178215-f2b184dc-f683-4740-af9f-fd67efd41503.png)



For the default theme, developers can chose between the light mode and dark mode. 

More themes can be selected: [Click here to check them on figma](https://www.figma.com/file/v069nTXfE8pXDJQcDcC5wl/Swap-Widget?node-id=0%3A1).

To integrate the Ref Swap Widget, please follow this guide.

## Getting started

A QuickStart of Ref Swap component.

### Props

```plain
export interface SwapWidgetProps {
    theme?: Theme;
    extraTokenList?: string[];
    onSwap: (transactionsRef: Transaction[]) => void;
    onDisConnect: () => void;
    width: string;
    height?: string;
    enableSmartRouting?: boolean;
    className?: string;
    darkMode?: boolean;
    connection: {
        AccountId: string;
        isSignedIn: boolean;
    };
    defaultTokenIn?: string;
    defaultTokenOut?: string;
    transactionState?: {
        state: 'success' | 'fail' | null;
        tx?: string;
        detail?: string;
    };
    onConnect: () => void;
}
```
* theme: widget theme for customization.
* extraTokenList: introduce extra tokens with ref whitelist into default token list in the widget.
* onSwap: Swap button triggers this function.
* width: width of widget component.
* height: height of widget component.
* enableSmartRouting: option to choose if enable smart routing in swap routes estimation.
* className: extra className added to widget component.
* darkMode: if true, will automatically set theme to default dark mode.
* connection: connection to wallets, input { AccountId:"", isSignedIn:false } if wallet not connected.
* defaultTokenIn: default token-in.
* defaultTokenOut: default token-out.
* transactionState: entry to input transaction states after you send transactions.
    * state: denote if last transaction is failed or successfull.
    * setState: used to change setState to interact with pop-up.
    * tx: will add link to near explorer according to this tx.
    * detail: you could input some tips to show on sucess pop-up.
    
![111](https://user-images.githubusercontent.com/50706666/199178453-8d09be3f-5a00-4b62-a6f1-af42ce4beae6.png)


* onDisConnect: Disconnect button triggers this function.
* onConnect: Connect to Near Wallet button triggers this function.
### Usage

#### Theme

```plain
export interface Theme {
  container: string; // container background
  buttonBg: string; // button background
  primary: string; // primary theme color
  secondary: string; // secondary theme color
  borderRadius: string; // border radius
  fontFamily: string; // font family
  hover: string; // hovering color
  active: string; // active color
  secondaryBg: string; // secondary background color
  borderColor: string; // border color
  iconDefault: string; // default icon color
  iconHover: string; // icon hovering color
  refIcon?: string; // ref icon color, default to be black
}

export const defaultTheme: Theme = {
  container: '#FFFFFF',
  buttonBg: '#00C6A2',
  primary: '#000000',
  secondary: '#7E8A93',
  borderRadius: '4px',
  fontFamily: 'sans-serif',
  hover: 'rgba(126, 138, 147, 0.2)',
  active: 'rgba(126, 138, 147, 0.2)',
  secondaryBg: '#F7F7F7',
  borderColor: 'rgba(126, 138, 147, 0.2)',
  iconDefault: '#7E8A93',
  iconHover: '#B7C9D6',
};

export const defaultDarkModeTheme: Theme = {
  container: '#26343E',
  buttonBg: '#00C6A2',
  primary: '#FFFFFF',
  secondary: '#7E8A93',
  borderRadius: '4px',
  fontFamily: 'sans-serif',
  hover: 'rgba(126, 138, 147, 0.2)',
  active: 'rgba(126, 138, 147, 0.2)',
  secondaryBg: 'rgba(0, 0, 0, 0.2)',
  borderColor: 'rgba(126, 138, 147, 0.2)',
  iconDefault: '#7E8A93',
  iconHover: '#B7C9D6',
  refIcon: 'white',
};
```

#### 
#### Component

```plain
// an example of combining SwapWidget with wallet-selector
import * as React from 'react';
import { SwapWidget } from '@ref_finance/ref-sdk';

// please check on wallet-selector example about how to set WalletSelectorContext
import { useWalletSelector } from './WalletSelectorContext';

import { WalletSelectorTransactions, NotLoginError } from '@ref_finance/ref-sdk';

export const Widget = ()=>{
  
  const { modal, selector, accountId } = useWalletSelector();
  
  const [swapState, setSwapState] = React.useState<'success' | 'fail' | null>(
    null
  );
  const [tx, setTx] = React.useState<string | undefined>(undefined);
  React.useEffect(() => {
    const errorCode = new URLSearchParams(window.location.search).get(
      'errorCode'
    );

    const transactions = new URLSearchParams(window.location.search).get(
      'transactionHashes'
    );

    const lastTX = transactions?.split(',').pop();

    setTx(lastTX);

    setSwapState(!!errorCode ? 'fail' : !!lastTX ? 'success' : null);

    window.history.replaceState(
      {},
      '',
      window.location.origin + window.location.pathname
    );
  }, []);
  
  const onSwap = async (transactionsRef: Transaction[]) => {
    const wallet = await selector.wallet();
    if (!accountId) throw NotLoginError;

    wallet.signAndSendTransactions(
      WalletSelectorTransactions(transactionsRef, accountId)
    );
  };
  
  const onConnect = () => {
    modal.show();
  };

  const onDisConnect = async () => {
    const wallet = await selector.wallet();
    return await wallet.signOut();
  };

  return (
    <SwapWidget
      onSwap={onSwap}
      onDisConnect={onDisConnect}
      width={'400px'}
      connection={{
        AccountId: accountId || '',
        isSignedIn: !!accountId,
      }}
      transactionState={{
        state: swapState,
        setState: setSwapState,
        tx,
        detail: '(success details show here)',
      }}
      enableSmartRouting={true}
      onConnect={onConnect}
      defaultTokenIn={'wrap.testnet'}
      defaultTokenOut={'ref.fakes.testnet'}
    />
  );
}
```


## Integrating Ref Swap function using the SDK

The SDK provides more flexibility/options. 

The default Swap version can serve as a reference. [Click here to check on figma.](https://www.figma.com/file/v069nTXfE8pXDJQcDcC5wl/Swap-Widget?node-id=0%3A1)

For more details about the SDK, please refer to [here].

SDK integration tips:

* You can use 'ftGetTokensMetadata' function to get all available tokens, and list them in the token selector to allow users to select their trading pair. If you do not need all available tokens, you can limit the list at the frontend level, thus only displaying specific tokens such as REF, NEAR, and USDT, for example. 
* Please pay attention to the user's 'tokenIn' balance. If the balance is zero, you should **DISABLE** the swap button. 
* You can let the user set the slippage, or you can pre set a default number. 
* For a better user experience, before the execution of the swap, you can show more details about the swap (ex: fee, rate, route, etc.), allowing users to take better data-driven decisions.
* You can redirect the user to the NEAR Explorer, once the transaction is confirmed.

# 

