---
sidebar_position: 5
---

# Zdefiniujmy nasz smart contract

## ABI - Application Binary Interface

Abi jest interfejsem pomiędzy smart contractem a klientem. Jest to konfiguracja definująca metody jakie zostały wystawione przez smart contract
W kontekście naszej aplikacji, ABI powinno zostać dostarczone przez twórcę smart contractu. Użyjmy gotowego ABI gotowego, zdeployowanego smart contractu



Utwórzmy więc plik `abis/todoListContractAbi.ts` i dodajmy w nim:

```typescript
export const todoListContractAbi = [
    {
        inputs: [
            {
                internalType: 'string',
                name: '_task',
                type: 'string',
            },
        ],
        name: 'addTask',
        outputs: [],
        stateMutability: 'nonpayable',
        type: 'function',
    },
    {
        inputs: [
            {
                internalType: 'address',
                name: '_user',
                type: 'address',
            },
        ],
        name: 'getTodoList',
        outputs: [
            {
                components: [
                    {
                        internalType: 'string',
                        name: 'task',
                        type: 'string',
                    },
                    {
                        internalType: 'bool',
                        name: 'completed',
                        type: 'bool',
                    },
                ],
                internalType: 'struct TodoList.TodoItem[]',
                name: '',
                type: 'tuple[]',
            },
        ],
        stateMutability: 'view',
        type: 'function',
    },
    {
        inputs: [
            {
                internalType: 'uint256',
                name: '_index',
                type: 'uint256',
            },
        ],
        name: 'markTaskCompleted',
        outputs: [],
        stateMutability: 'nonpayable',
        type: 'function',
    },
];
```


## Adres oraz sieć
Aby była możliwa jakakolwiek interakcja ze smart contractem powyżej, brakuje nam już tylko adresu na jaki został zdeployowany
oraz chainu (sieci) na której się znajduje.

Kontrakt którego użyjemy jest zdeployowany na sieci `Goerli` - jest to testnet sieci Ethereum (środowisko testowe). 
Dla uproszczenia założymy że nie będziemy narazie sprawdzać czy w portfelu jest wybrana prawidłowa sieć, natomiast 
aplikacja będzie działać prawidłowo tylko na sieci `Goerli`. W kolejnych krokach możemy dodać sprawdzenie sieci
i wymagać od użytkownika zmiany, gdy znajduje sięna nieprawidłowej

Adres naszego smart contractu to:
```
0x756f739a0c08d3cb341bdf8e41796a113ae00b16
```

Możemy również podejrzeć szczegóły smart contractu oraz wszystkie transakcje na `Etherscanner` (dla sieci goerli):
https://goerli.etherscan.io/address/0x756f739a0c08d3cb341bdf8e41796a113ae00b16


## Provider do interakcji ze smart contractami

Do interakcji ze smart contractem użyjemy biblioteki `ethers.js`. Dodajmy ją więc do zależności projektu:
```
yarn add ethers
```

Następnie utwórzmy provider, który umożliwi nam interakcje. W nowym pliku `provider.ts` dodajemy:

```typescript
import { BrowserProvider } from 'ethers';

export const provider = window?.ethereum
  ? new BrowserProvider(window.ethereum)
  : null;

```

Utwórzmy sobie również hook reactowy, który ułatwi nam korzystanie z providera tak aby trzeba było za każdym razem sprawdzać 
czy provider nie ma wartości `null`
`UWAGA!` utworzonego hooka będziemy używać tylko w ścieżce z połączonym portfelem - wszystko wewnątrz ścieżki `/app` w naszym routingu

Tworzymy plik `hooks/useProvider.ts` a w nim:

```typescript
import { provider } from '../provider';

export const useProvider = () => {
  if (provider === null) {
    throw new Error('You\'re trying to access eth provider outside of application');
  }

  return provider;
};

```

## Czas na konkretny kontrakt

Ok, mamy provider, mamy ABI, mamy adres, utwórzmy więc prosty hook do pobrania danych ze smart contractu. 
Utwórzmy plik `hooks/useTodoContract.ts`

```typescript
import { useEffect, useState } from 'react';
import { Contract } from 'ethers';
import { useProvider } from './useProvider';
import { todoListContractAbi } from '../abis';
import { useAppStore } from '../store';

const contractAddress = '0x756f739a0c08d3cb341bdf8e41796a113ae00b16';

export const useTodoContract = () => {
    const { activeAccount } = useAppStore();
    const provider = useProvider();
    const [contract, setContract] = useState<Contract | null>(null);

    const initContract = async () => {
        try {
            const signer = await provider.getSigner(activeAccount as string);
            const contractObject = new Contract(contractAddress, todoListContractAbi, signer);

            setContract(contractObject);
        } catch (_e) {
            console.log('Contract initialization failed');
        }
    };
    

    useEffect(() => {
        initContract();
    }, []);

    return {};
};
```
