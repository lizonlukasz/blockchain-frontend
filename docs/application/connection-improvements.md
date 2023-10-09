---
sidebar_position: 4
---

# Niezbędne usprawnienia

## Przekierowanie na odpowiednią stronę

Nasza aplikacja będzie miała za zadanie robić odpowiednie przekierowania:
- z LandingPage do App jeśli użytkownik **jest połączony** z portfelem
- z App do Landing page jeśli użytkownik **nie jest połączony**

Aby to osiągnąc dodajmy odpowiednio do `LandingPage` oraz użyjmy komponentu `Navigate` z React Router do przekierowania:

```typescript jsx
const { activeAccount } = useAppStore();
if (activeAccount) return <Navigate to="/app" />;

```

Teraz analogicznie w komponencie `App`, który jest pierwszym komponentem całej aplikacji:

```typescript jsx
const { activeAccount } = useAppStore();
if (!activeAccount) return <Navigate to="/" />;
```

od teraz jesteśmy w stanie wyświetlić tylko jedną ścieżkę z dwóch możliwych, w zależności od stanu połączenia z portfelem

### Sprawdzanie stanu połączenia przy każdym załadowaniu (odświeżeniu) aplikacji

Należałoby również sprawdzić przy każdym odświeżeniu strony, czy użytkownik nie jest już połączony z portfelem. 
Aby to zrobić, owrapujemy całą aplikację komponentem `Entrypoint`, który będzie robił niezbędne checki
oraz odpowiednio aktualizował stan aplikacji. 
Stwórzmy więc pusty komponent `Entrypoint`...

```typescript jsx
import { FC, PropsWithChildren, useEffect } from 'react';
import { useAppStore } from 'store';

export const Entrypoint: FC<PropsWithChildren> = ({ children }) => {
    const { setMetamaskError, setActiveAccount } = useAppStore();

    useEffect(() => {
        // logic check goes here
    }, []);

    return (
        <>{children}</>
    );
};
```

...a następnie owrapujmy nim aplikację (w pliku `main.ts`)

```typescript jsx
ReactDOM.createRoot(document.getElementById('root')!).render(
    <React.StrictMode>
        <NextUiWrapper>
            <Entrypoint>
                <RouterProvider router={router} />
            </Entrypoint>
        </NextUiWrapper>
    </React.StrictMode>,
);
```

Sprawdzenie połączenia zrobimy wewnątrz hook'a `useEffect`:

```typescript jsx
  useEffect(() => {
    if (!window.ethereum) return () => undefined;

    const handleAccountChange = (accounts: string[]) => {
        setActiveAccount(accounts[0]);
        console.log('account has been changed or connected');
    };

    // check if user is already connected
    window.ethereum.request({ method: 'eth_accounts' })
        .then(handleAccountChange)
        .catch(() => setMetamaskError('Unable to get accounts'));

    return () => undefined;
}, []);

```

## Obsługa rozłączenia / zmiany konta

Kolejne rzeczy które należy obsłużyć to disconnect oraz zmianę konta. Użyjemy w tym celu
listenerów udostępnianych przez provider metamaska (czyli po prostu `window.ethereum`):

```typescript
window.ethereum.addListener('accountsChanged', () => {});
window.ethereum.addListener('disconnect', () => {});
```

I cyk - wrzućmy to do naszego kodu


```typescript jsx
  useEffect(() => {
    if (!window.ethereum) return () => undefined;

    const handleAccountChange = (accounts: string[]) => {
        setActiveAccount(accounts[0]);
        console.log('account has been changed or connected');
    };

    const handleDisconnect = () => {
        setActiveAccount(null);
        console.log('wallet disconnected');
    };

    window.ethereum.addListener('accountsChanged', handleAccountChange);
    window.ethereum.addListener('disconnect', handleDisconnect);
    // check if user is already connected
    window.ethereum.request({ method: 'eth_accounts' })
        .then(handleAccountChange)
        .catch(() => setMetamaskError('Unable to get accounts'));

    return () => {
        window.ethereum.removeListener('accountsChanged', handleAccountChange);
        window.ethereum.removeListener('disconnect', handleDisconnect);
    };
  }, []);
```
