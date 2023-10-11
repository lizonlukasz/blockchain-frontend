---
sidebar_position: 6
---

# Lista TODO

## Pobieranie listy tasków

po pierwsze stwórzmy logikę do pobierania listy tasków. W pliku `hooks/useTodoContract.ts` dodajmy: 

#### Typy oraz funkcję mapującą pobrane dane
```typescript
export type TaskResponse = [string, boolean];
export type TaskTableData = {
  index: number;
  text: string;
  completed: boolean;
};

const mapTasksToTable = (tasksResponse: TaskResponse[]): TaskTableData[] => tasksResponse.map(
  ([text, completed], index) => ({
    text, completed, index,
  }),
);
```

#### Stan Reactowy do przechowywania listy tasków oraz stanu ładowania
```typescript
const [tasks, setTasks] = useState<TaskTableData[]>([]);
const [tasksLoading, setTasksLoading] = useState<boolean>(false);
```

#### Funkcję do pobierania tasków:
```typescript
  const getTasks = async () => {
    setTasksLoading(true);

    if (contract) {
        try {
            const response: TaskResponse[] = await contract.getTodoList(activeAccount);
            setTasks(mapTasksToTable(response));
            setTasksLoading(false);
        } finally {
            setTasksLoading(false);
        }
    }
};
```

#### useEffect do wywoływania funkcji fetchującej
```typescript
  useEffect(() => {
    getTasks();
}, [contract, activeAccount, pendingTransactionsCount]);
```

Na koniec zwróćmy też w hooku stan tak, aby był dostępny na zewnątrz
```typescript
  return {
    tasks,
    tasksLoading,
  };
```

## UI do wyświetlania tasków

potrzebujemy początkowy UI do wyświetlania listy todo. Stwórzmy więc komponent `Todo` w pliku `views/Todo/Todo.tsx`:


```typescript jsx
import {
  FC, FormEvent, useState,
} from 'react';
import { useTodoContract } from './useTodoContract';
import { PageTitle } from '../../components/PageTitle';
import { PageWrapper } from '../../components';

export const Todo: FC = () => {
  const { tasks, tasksLoading } = useTodoContract();

  return (
    <PageWrapper>
      <PageTitle>Todo list</PageTitle>

      <h4 className="font-bold text-xl mb-2">My curent tasks</h4>
      <ul>
        {tasks.map((task) => (<ol>{task.text}</ol>))}
      </ul>
    </PageWrapper>
  );
};

```


