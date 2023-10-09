---
sidebar_position: 3
---

# Połącz sięz portfelem

## Sprawdzenie czy Metamask jest zainstalowany

W pierwszej kolejności należałoby sprawdzić czy użytkownik ma zainstolwany dodatek
MetaMask w przeglądarce. Jak czytamy w dokumentacji MetaMask:

```
The presence of the MetaMask Ethereum provider object, window.ethereum, in a user's browser indicates an Ethereum user.
```

W pliku `LandingPage.tsx` dodajemy:

```typescript jsx
const metamaskInstalled = !!window.ethereum;
const buttonText = metamaskInstalled ? 'Connect Wallet' : 'Please install Metamask to use this app';
```

a następnie ustawiamy text przycisku oraz atrybut `disabled` w przypadku jeśli aplikacja nie wykryła MetaMaska


### Łączęnie z portfelem 
W pierwszej kolejności dorzućmy klikalny kafelek do łączenia z portfelem oraz hanlder kliknięcia w niego

```typescript jsx
const handleWalletConnect = () => undefined;

...
  
<Card
  className="dark:bg-default-100/50 cursor-pointer"
  isPressable
  onPress={handleWalletConnect}
>
  <CardBody className="m-0 p-4">
    <div className="flex items-center">
      <Image
        alt="Metamask logo"
        width={50}
        shadow="sm"
        src={metamaskIcon}
      />

      <h1 className="text-xl ml-4 font-semibold">MetaMask</h1>
    </div>
  </CardBody>
</Card>
```

## Łączenie z portfelem

Następnie  zajmimy się próbą połączenia z portfelem. Czytając w dokumentacji:

```
To request a signature from a user or have a user approve a transaction, 
your dapp must access the user's accounts using the eth_requestAccounts RPC method.
```

Czyli potrzebujemy wywołać metodę `eth_requestAccounts`.W odpowiedzi dostaniemy listę połączonych kont, 
gdzie aktywne konto będzie zawsze pierwszym elementem z tablicy. Następnie możemy zapisać aktywne konto
do stanu aplikacji:


```typescript jsx
    setConnectingMetamask(true);

    if (window.ethereum) {
      window.ethereum.request({ method: 'eth_requestAccounts' })
        .then((accounts: string[]) => {
          setActiveAccount(accounts[0]);
          setConnectingMetamask(false);
        })
        .catch(() => setMetamaskError('Unable to connect to MetaMask'));
    }
```

## Wyświetlenie informacje na temat połączonego konta

W celu wyświetlenia informacji na temat połączonego konta, utwórzmy nowy komponent `AccountDetail.tsx`

```typescript jsx
import { User } from '@nextui-org/react';
import { useAppStore } from '../../store';

export const AccountDetails = () => {
  const { activeAccount } = useAppStore();

  if (!activeAccount) return null;

  return (
    <User
      name={activeAccount}
      description="TBA"
      avatarProps={{
        src: 'https://picsum.photos/200',
      }}
    />
  );
};
```

Następnie osadzamy komponent w komponencie `Navbar.tsx`
