---
sidebar_position: 9
---

# Oczekiwanie na transakcję

Transakcja którą wysłaliśmy musi zostać sfinalizowana na blockchainie żebyśmy mogli stwierdzić że task został poprawnie dodany

## Obsługa oczekującej transakcji
wróźmy znów do hooka `useTodoContract` i dodajmy kolejny stan, który będzie przechowywał liczbę oczekujących transakcji:

```typescript
  const [pendingTransactionsCount, setPendingTransactionsCount] = useState<number>(0);
```


Następnie zmodyfikujmy trochę funkcję odpowiedzialną za tworzenie taska
```typescript
    const addTask = async (task: string) => {
    if (contract) {
        try {
            const tx = await contract.addTask(task);
            setPendingTransactionsCount((prev) => prev + 1);
            await tx.wait();
            setPendingTransactionsCount((prev) => prev - 1);
        } catch (_e) {
            console.log('Unable to create new task');
        }
    }
};
```

oraz oczywiście dołużmy nasz stan do zwrotki hooka

```typescript
  return {
    tasks,
    addTask,
    tasksLoading,
    pendingTransactionsCount,
    completeTask,
};
```

Teraz w komponencie `Todo`, przekażmy do tabeli tasków liczbę oczekujących transakcji:
```typescript jsx
const {
    addTask, tasks, completeTask, tasksLoading, pendingTransactionsCount,
} = useTodoContract();

...

<TasksTable
    tasks={tasks}
    isLoading={tasksLoading}
    onComplete={completeTask}
    pendingRows={pendingTransactionsCount}
/>
```



