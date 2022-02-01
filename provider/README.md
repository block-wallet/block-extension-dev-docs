# BlockWallet Ethereum Provider API

The BlockWallet extension injects an ethereum provider on every site that the user visits. This document aims to help developers to connect to Blank Wallet.

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

At all times, BlockWallet will return only one account in the array, that would be the account the users decides to connect and is active.

## Events

### Accounts

Whenever there is an update on the accounts that the provider should communicate to your app, the event `accountsChanged` is emitted.

```typescript
ethereum.on('accountsChanged', handler: (accounts: string[]) => void);
```

If there is no account present, your app may have not be connected to the BlockWallet, or the wallet may be locked.
