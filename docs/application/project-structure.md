---
sidebar_position: 2
---

# Struktura projektu

- 2 główne ścieżki 
  - Landing page dla niezalogowanych użytkowników - na ścieżce `/`
  - Aplikacja dla zalogowanych użytkowników - na ścieżce `/app/*`
- zcentralizowany stan aplikacji, utworzony na bibliotece Zustand


Do store dorzucimy od razu wszystkie zmienne, których użyjemy później

Najpierw zdefiniujmy odpowiedni interfejs - `types/store.ts`:

```typescript
export interface AppState {
  theme: 'dark' | 'light';
  toggleTheme: () => void;
  metamaskError?: string;
  setMetamaskError: (v?: string) => void;
  connectingMetamask: boolean;
  setConnectingMetamask: (isConnecting: boolean) => void;

  // metamask account
  activeAccount: string | null;
  setActiveAccount: (account: string | null) => void;
}
```

Następnie zainicjalizujmy store z wszystkimi wartościami: 

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { AppState } from 'types';

const initialState: Pick<
        AppState,
        'theme'
        | 'connectingMetamask'
        | 'activeAccount'
> = {
  theme: 'dark',
  connectingMetamask: false,
  activeAccount: null,
};

export const useAppStore = create<AppState>()(
        persist(
                (set, get): AppState => ({
                  ...initialState,
                  toggleTheme: () => set({ theme: get().theme === 'dark' ? 'light' : 'dark' }),
                  setMetamaskError: (value) => set({ metamaskError: value }),
                  setConnectingMetamask: (value) => set({ connectingMetamask: value }),
                  setActiveAccount: (value) => set({ activeAccount: value }),
                }),
                { name: 'zustand-state' },
        ),
);

```
