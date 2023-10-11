---
sidebar_position: 8
---

# Tworzenie nowego taska

## Logika tworzenia nowego taska:
w naszym hooku obsługującym kontrakt, czyli `useTodoContract` dodajmy teraz funkcje do tworzenia oraz odznaczania tasków:

```typescript
  const addTask = async (task: string) => {
    if (contract) {
      try {
        await contract.addTask(task);
      } catch (_e) {
        console.log('Unable to create new task');
      }
    }
  };

  const completeTask = async (idx: number) => {
    if (contract) {
      try {
        await contract.markTaskCompleted(idx);
      } catch (_e) {
        console.log('Unable to complete task');
      }
    }
  };
```

Oraz dodajmy te funkcje do zwrotki
```typescript
  return {
    tasks,
    addTask,
    tasksLoading,
    completeTask,
  };
```

## Komponent dodawania taska
W komponencie `Todo` dołużmy UI odpowiadający za wprowadzanie nowego taska. Na początek zdefiniujmy jednak zmienne stanowe
oraz funkcję do dodawania taska:

```typescript jsx
  const {
    addTask, tasks, completeTask, tasksLoading,
  } = useTodoContract();
  const [inputText, setInputText] = useState<string>('');

  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    addTask(inputText);
  };
```

Następnie dorzućmy UI:
```typescript jsx
<h4 className="font-bold text-xl mb-2">Add new TODO ticket</h4>
<Card className="mb-10">
    <CardBody>
        <div className="flex flex-col gap-4">
            <form onSubmit={handleSubmit} className="flex w-full gap-6 items-end">
                <Input
                    type="text"
                    label="Task text"
                    placeholder="Type here..."
                    labelPlacement="outside"
                    value={inputText}
                    onChange={(e) => setInputText(e.target.value)}
                    minLength={3}
                />
                <Button variant="bordered" type="submit">Add</Button>
            </form>
        </div>
    </CardBody>
</Card>
```



