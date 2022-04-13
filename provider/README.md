# BlockWallet Ethereum Provider API

The BlockWallet extension injects an ethereum provider on every site that the user visits. This document aims to help developers to connect to BlockWallet.

## If your project already supports injected providers or specific wallets like MetaMask, your site is most-likely already ready to be used with BlockWallet. As long as you don't specifically check for a certain property like `isMetaMask`, you can connect BlockWallet as you would other browser wallets.

## Provider detection

The BlockWallet provider is a class that's instantiated as `window.ethereum` , to check if the user has BlockWallet or any other compatible wallet installed, you could use the following:

```typescript
const detectProvider = (): Promise<EthereumProvider | null> => {
  return new Promise((resolve) => {
    const handleProvider = () => {
      window.removeEventListener('ethereum#initialized', handleProvider);

      const { ethereum } = window;

      if (ethereum) {
        resolve(ethereum);
      } else {
        console.error('Unable to detect window.ethereum.');
        resolve(null);
      }
    };

    if (window.ethereum) {
      handleProvider();
    } else {
      window.addEventListener('ethereum#initialized', handleProvider, {
        once: true,
      });
    }
  });
};

const provider = await detectProvider();

if (provider) {
  // Initialize your app
} else {
  console.log('Provider not detected');
}
```

## Detect BlockWallet

To detect if the user has BlockWallet installed on his browser, you can check if the `isBlockWallet` property is available in the detected provider.

```typescript
const isBlockWallet: boolean | undefined = ethereum.isBlockWallet;

if (!isBlockWallet) {
  console.log('Please install BlockWallet 😁');
}
```

This value will be undefined if it's not a BlockWallet provider.

## Request permissions

The only permission that we grant to external dapps is the `eth_accounts` permission.

We recommend you use the standard EIP-1193 connection method:

```typescript
const connect = (): string[] => {
  ethereum.request({ method: 'eth_requestAccounts' }).catch((error) => {
    if (error.code === 4001) {
      // See: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1193.md#provider-errors
      console.log('Connection rejected.');
    } else {
      console.error(err);
    }
  });
};
```

At all times, BlockWallet will return only one account in the array, that would be the account the user decides to connect and is active.

## Web3Modal

Since version 1.9.6, Web3Modal natively supports BlockWallet. With a default implementation, the BlockWallet option will automatically show up in your modal, if the user has BlockWallet installed and activated.

## Custom Web3Modal connection

You can also add BlockWallet as a custom Web3Modal connection, if for some reason you cannot or do not want to use the default connection.

```typescript
const blockWalletProvider = {
    id: "injected",
    name: "BlockWallet",
    logo: BlockWalletLogo,
    type: "injected",
    check: "isBlockWallet"
}

const providerOptions = {
    "custom-blockWallet": {
        display: {
            logo: BlockWalletLogo,
            name: "BlockWallet",
            description: "Connect to BlockWallet",
        },
        package: blockWalletProvider,
        connector: async () => {
            let provider = null

            if (window.ethereum) {
                provider = window.ethereum

                if (!provider.isBlockWallet) {
                    throw new Error("Not a BlockWallet")
                }

                try {
                    await provider.request({ method: "eth_requestAccounts" })
                } catch (error) {
                    throw new Error("User Rejected")
                }
            } else {
                throw new Error("BlockWallet not found")
            }

            return provider
        }
    },
}

const web3Modal = new Web3Modal({
    providerOptions
})
```

## Events

### Accounts

Whenever there is an update on the accounts that the provider should communicate to your app, the event `accountsChanged` is emitted.

```typescript
ethereum.on('accountsChanged', handler: (accounts: string[]) => void);
```

If there is no account present, your app may not be connected to BlockWallet, or the wallet may be locked.
