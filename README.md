# Call stack, Asynchronous call stack, Task queue and Event loop

## 1. Call stack

The **Call Stack** is where JavaScript keeps track of what function is currently running and what to return to when a function finishes.

> JavaScript is single-threaded, so only one function can run at a time.How it works:
> 
- When a function is called → it's pushed to the top of the stack.
- When it finishes → it’s popped off.

### 📦 Example:

```jsx
javascript
CopyEdit
function multiply(x, y) {
  return x * y;
}

function square(n) {
  return multiply(n, n);
}

function printSquare(num) {
  const result = square(num);
  console.log(result);
}

printSquare(5);

```

### 📋 Call Stack Trace:

```jsx
1. printSquare(5)    // pushed
2. square(5)         // pushed
3. multiply(5, 5)    // pushed
4. return 25         // multiply popped
5. return 25         // square popped
6. console.log(25)   // pushed
7. log done          // console.log popped
8. printSquare done  // popped

```
2.Asynchronous call stack

When JavaScript encounters an asynchronous function like:

- `setTimeout()`
- `fetch()`
- `fs.readFile()` (Node.js)
- `XMLHttpRequest` or `addEventListener`

…it **doesn’t block the call stack**. Instead:

1. JavaScript **hands off** the task to a **browser API** (or Node API).
2. The API performs the task (e.g., a timer or HTTP request).
3. Once done, it places a **callback** in the **task queue** or **microtask queue**.
4. The **event loop** monitors the call stack and queues. When the stack is empty, it dequeues a callback. Code Example

---

```jsx
console.log("1: Script Start");

setTimeout(() => {
  console.log("4: setTimeout callback");
}, 0);

fetch("https://jsonplaceholder.typicode.com/todos/1")
  .then(response => response.json())
  .then(data => {
    console.log("5: fetch callback (Promise)");
  });

Promise.resolve().then(() => {
  console.log("3: Promise microtask");
});

console.log("2: Script End");

```

---

### Step-by-Step Breakdown:

### 1. Synchronous code starts executing:

```jsx
console.log("1: Script Start");  // → printed immediately

```

→ Enters and exits call stack.

---

### 2. `setTimeout(..., 0)` is called:

- Timer function is registered with **Web API**.
- After ~0ms (next tick), callback `() => { console.log("4: setTimeout callback") }` is placed in the **task queue**.

---

### 3. `fetch(...)` is called:

- `fetch` is **asynchronous** and uses **browser API (Fetch API)**.
- It makes an HTTP request **outside** of JS.
- When response arrives:
    - First `.then()` is queued in **microtask queue**.
    - Then second `.then()` (chained) is also queued when ready.

---

### 4. `Promise.resolve().then(...)`:

- Promise resolves **immediately**.
- Callback `() => { console.log("3: Promise microtask") }` is placed in the **microtask queue**.

---

### 5. `console.log("2: Script End")`

→ Prints immediately.

---

### ******Now the Call Stack is Empty******

### The **Event Loop** kicks in:

1. Checks the **Microtask Queue** → runs:
    - `console.log("3: Promise microtask")`
    - Then `.then()` from `fetch`, if resolved
2. Then checks the **Task Queue**:
    - `console.log("4: setTimeout callback")`

---

### 📋 Final Output:

```jsx
1: Script Start
2: Script End
3: Promise microtask
5: fetch callback (Promise)
4: setTimeout callback
```

---

## Real-World Example

Imagine the **call stack** is a **chef** making food.

- When a **dish takes time** (e.g., boiling water), the chef **asks a helper** (Web API) to handle it.
- The chef keeps making other dishes (synchronous code).
- Once the boiling is done, the helper places a **note (callback)** in a queue.
- When the chef finishes all current dishes, he checks the notes (task/microtask queue) and continues.


3. Difference between the Task Queue and the Microtask Queue ?

The JavaScript Event Loop processes:

1. **Synchronous code** first.
2. Then **all microtasks** from the **Microtask Queue**.
3. Then the **next task** from the **Task Queue**.
4. Repeats...

---

## Microtask Queue

**Higher priority** than task queue.

### Sources:

- `Promise.then()`, `catch()`, `finally()`
- `queueMicrotask()`
- `MutationObserver`

---

## Task Queue

**Lower priority** than microtasks.

### Sources:

- `setTimeout()`
- `setInterval()`
- `setImmediate()` (Node.js)
- DOM events
- Network events

---

## Example with Both Queues

```jsx

console.log('Start');

setTimeout(() => {
  console.log('Task: setTimeout');
}, 0);

Promise.resolve().then(() => {
  console.log('Microtask: Promise 1');
}).then(() => {
  console.log('Microtask: Promise 2');
});

queueMicrotask(() => {
  console.log('Microtask: queueMicrotask');
});

console.log('End');

```

---

## Output:

```jsx
Start
End
Microtask: Promise 1
Microtask: Promise 2
Microtask: queueMicrotask
Task: setTimeout
```

---

## Why this order?

1. `console.log('Start')` → **synchronous**
2. `setTimeout(...)` → **goes to Task Queue**
3. `Promise.then(...)` and `queueMicrotask(...)` → **go to Microtask Queue**
4. `console.log('End')` → **synchronous**
5. After all synchronous code:
    - Run **all microtasks**:
        - `Microtask: Promise 1`
        - `Microtask: Promise 2`
        - `Microtask: queueMicrotask`
6. Then run **one task** from Task Queue:
    - `Task: setTimeout`

---

## 🔁 Visual Diagram

```
text
CopyEdit
[Call Stack ↓]
┌────────────────────┐
│  console.log('Start')      │
│  console.log('End')        │
└────────────────────┘
       ↓
┌────────────────────────────┐
│      Microtask Queue       │
│  - Promise 1               │
│  - Promise 2               │
│  - queueMicrotask          │
└────────────────────────────┘
       ↓
┌────────────────────────────┐
│        Task Queue          │
│  - setTimeout              │
└────────────────────────────┘

```

4. Event loop

The **Event Loop** is the "traffic controller" of JavaScript’s concurrency model.

It does **3 main things**:

1. Continuously checks if the **Call Stack** is empty.
2. If it is, it first **empties the Microtask Queue** (Promises, MutationObservers).
3. Then it processes one **Task** from the **Task Queue** (e.g., `setTimeout`, `setInterval`).

This cycle repeats **forever** while your program runs.

---

## JS Concurrency Components

```

     ┌────────────────────┐
     │   Call Stack       │
     └────────────────────┘
                ↓ (empty)
     ┌────────────────────┐
     │ Microtask Queue    │ ← run ALL microtasks
     └────────────────────┘
                ↓
     ┌────────────────────┐
     │ Task Queue          │ ← run ONE task
     └────────────────────┘
                ↓
        (Back to Top)

```

---

## 🔁 Extended Example

```jsx
console.log("1: Start");

setTimeout(() => {
  console.log("5: setTimeout callback");
}, 0);

Promise.resolve().then(() => {
  console.log("3: Promise microtask 1");
});

Promise.resolve().then(() => {
  console.log("4: Promise microtask 2");
});

console.log("2: End");
```

---

### Step-by-Step Timeline::

### Step 1: Synchronous Phase (Call Stack)

Runs top to bottom:

1. `console.log("1: Start")` → prints
2. `setTimeout(...)` → registers with Web API
3. `Promise.resolve().then(...)` → microtask added to queue
4. Second `Promise.then()` → also added to microtask queue
5. `console.log("2: End")` → prints

---

### Step 2: Microtask Phase

Now the **event loop** finds the stack empty, and executes **ALL** microtasks:

- Executes `Promise microtask 1` → prints `3: Promise microtask 1`
- Executes `Promise microtask 2` → prints `4: Promise microtask 2`

---

### Step 3: Task Phase

Now the event loop processes the **next task** from the **Task Queue**:

- Executes `setTimeout callback` → prints `5: setTimeout callback`

---

### 🧾 Final Output:

```
1: Start
2: End
3: Promise microtask 1
4: Promise microtask 2
5: setTimeout callback
```

---

---

## `async/await` in the Event Loop

```jsx
async function foo() {
  console.log("2: Inside async");
  await Promise.resolve();
  console.log("4: After await");
}

console.log("1: Start");
foo();
console.log("3: After foo()");
```

### 🧾 Output:

```
1: Start
2: Inside async
3: After foo()
4: After await
```

### 📋 Explanation:

- `foo()` logs `"2"`.
- Hits `await` → pauses execution and pushes the continuation (`"4"`) into **microtask queue**.
- Logs `"3"` synchronously.
- Once the call stack is empty, microtask `"4"` runs.