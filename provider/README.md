# Blank Ethereum Provider API

The Blank Wallet extension injects an ethereum provider on every site that the user visits. This document aims to help developers to connect to Blank Wallet.

## Provider detection

The blank provider is a class that's instantiated as `window.ethereum` , to check if the user has blank or any other compatible wallet installed, you could use the following:

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

## Detect Blank Wallet

To detect if the user has Blank Wallet installed on his browser, you can check if the `isBlank` property is available in the detected provider.

```typescript
const isBlankWallet: boolean | undefined = ethereum.isBlank;

if (!isBlankWallet) {
  console.log('Please install Blank Wallet 😁');
}
```

This value will be undefined if it's not a blank provider.

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

At all times, blank wallet will return only one account in the array, that would be the account the users decides to connect and is active.

## Events

### Accounts

Whenever there is an update on the accounts that the provider should communicate to your app, the event `accountsChanged` is emitted.

```typescript
ethereum.on('accountsChanged', handler: (accounts: string[]) => void);
```

If there is no account present, your app may have not be connected to the blank wallet, or the wallet may be locked.
