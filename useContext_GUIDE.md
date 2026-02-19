# 🎯 useContext Hook - Complete Guide & Reference

## Table of Contents
1. [Theory & Concepts](#theory--concepts)
2. [Step-by-Step Setup](#step-by-step-setup)
3. [Real Project Example](#real-project-example)
4. [Best Practices](#best-practices)
5. [Common Patterns](#common-patterns)
6. [Troubleshooting](#troubleshooting)

---

## Theory & Concepts

### What is Context API?
The **Context API** is a React feature that allows you to pass data (state & functions) through the component tree without having to pass props manually at every level. This solves the "prop drilling" problem.

### What is useContext?
**useContext** is a React Hook that lets you consume (access) data from a Context in any component without prop drilling.

### Problem it Solves: Prop Drilling
```
❌ WITHOUT Context:
App → Layout → Dashboard → Card → Button
Props passed: button_handler, user_data, theme...passed through every component

✅ WITH Context:
App → (Context wraps all) → Button can access directly
No need to pass through intermediate components
```

### The 3-Step Pattern
1. **Create** a Context
2. **Provide** the Context (wrap components with Provider)
3. **Consume** the Context (useContext hook in any child component)

---

## Step-by-Step Setup

### Step 1️⃣: Create Context Folder & File

```bash
src/
├── context/
│   └── index.jsx    ← Create this file
├── App.jsx
└── main.jsx
```

### Step 2️⃣: Create Context Provider (`src/context/index.jsx`)

```jsx
// ===== STEP 1: Import React Functions =====
import { createContext, useState } from "react";

// ===== STEP 2: Create Context Object =====
const MyContext = createContext();

// ===== STEP 3: Create Provider Component =====
const MyProvider = (props) => {
  
  // ===== STEP 4: Define Your State Variables =====
  const [state1, setState1] = useState(initialValue);
  const [state2, setState2] = useState(initialValue);

  // ===== STEP 5: Define Handler Functions =====
  const handleAction1 = () => {
    // Your logic here
  };

  const handleAction2 = () => {
    // Your logic here
  };

  // ===== STEP 6: Return Provider with Value =====
  return (
    <MyContext.Provider
      value={{
        // Share STATE
        state1: state1,
        state2: state2,
        
        // Share METHODS/HANDLERS
        handleAction1: handleAction1,
        handleAction2: handleAction2,
      }}
    >
      {props.children}
    </MyContext.Provider>
  );
};

// ===== STEP 7: Export Both =====
export { MyContext, MyProvider };
```

### Step 3️⃣: Wrap App with Provider (`src/main.jsx`)

```jsx
import { createRoot } from 'react-dom/client';
import { MyProvider } from "./context/index.jsx";  // ← Import Provider
import App from './App.jsx';

createRoot(document.getElementById('root')).render(
  <MyProvider>  {/* ← Wrap entire app */}
    <App />
  </MyProvider>
);
```

### Step 4️⃣: Use Context in Any Component

```jsx
// ===== Inside ANY child component =====
import { useContext } from "react";
import { MyContext } from "./context/index.jsx";

const MyComponent = () => {
  // ===== CONSUME Context =====
  const context = useContext(MyContext);

  return (
    <div>
      {/* Access state */}
      <p>Value: {context.state}</p>
      
      {/* Call methods */}
      <button onClick={context.handleFunc1}>
        Click Me
      </button>
    </div>
  );
};
```



---

## Best Practices

### ✅ DO's

| Practice | Example |
|----------|---------|
| **Create custom hooks** for context | `useMyContext.js` → `useContext(MyContext)` |
| **Split large contexts** | Separate AuthContext, ThemeContext, etc. |
| **Use descriptive names** | `removePlayer` not `remove` |
| **Validate before dispatch** | Check data before updating state |
| **Memoize context value** | Use `useMemo()` for expensive operations |
| **Group related data** | Keep related state & methods together |

### ❌ DON'Ts

| Anti-pattern | Why? |
|------------|------|
| **Pass entire context** when you need 1 value | Causes unnecessary re-renders |
| **Deeply nest providers** | Hard to debug & manage |
| **Store everything in context** | Use local state when possible |
| **Update state directly** | Always use setState/updaters |
| **Export raw context** | Create custom hook wrapper |

---

## Common Patterns

### Pattern 1: Custom Hook Wrapper
```jsx
// ✅ BETTER: Create a custom hook
import { useContext } from 'react';
import { MyContext } from './context/index.jsx';

export const useMyContext = () => {
  const context = useContext(MyContext);
  if (!context) {
    throw new Error('useMyContext must be used within MyProvider');
  }
  return context;
};

// Now in components:
import { useMyContext } from './hooks/useMyContext';

const MyComponent = () => {
  const { state, handler } = useMyContext();
  // ...
}
```

### Pattern 2: Separating Concerns
```jsx
// Split into multiple contexts by domain
const AuthContext = createContext();        // Authentication logic
const ThemeContext = createContext();       // Theme/UI data
const DataContext = createContext();        // App data

// Wrap with multiple providers
<AuthProvider>
  <ThemeProvider>
    <DataProvider>
      <App />
    </DataProvider>
  </ThemeProvider>
</AuthProvider>
```

### Pattern 3: Reducer Pattern (Advanced)
```jsx
import { useReducer } from 'react';

const appReducer = (state, action) => {
  switch(action.type) {
    case 'ACTION_TYPE_1':
      return { ...state, property: action.payload };
    case 'ACTION_TYPE_2':
      return { ...state, otherProperty: action.payload };
    default:
      return state;
  }
};

const MyProvider = (props) => {
  const [state, dispatch] = useReducer(appReducer, initialState);
  
  return (
    <MyContext.Provider value={{ state, dispatch }}>
      {props.children}
    </MyContext.Provider>
  );
};
```

---

## Troubleshooting

### ❌ Issue: "useContext is returning undefined"
```
Cause: Component not wrapped with Provider
Solution: Ensure Provider wraps the component in main.jsx
```

### ❌ Issue: "Cannot read property 'xxx' of undefined"
```
Cause: Trying to access context that doesn't exist
Solution: Export both MyContext and MyProvider correctly
        Verify import path is correct
```

### ❌ Issue: "Everything re-renders when context changes"
```
Cause: Context value recreated every render
Solution: Wrap value in useMemo():
  const value = useMemo(() => ({
    players,
    addPlayer,
    ...
  }), [players])
```

### ❌ Issue: "Props aren't updating in child component"
```
Cause: Forgot to invoke setState function
Solution: Make sure you're calling methods from context, not local state
```

---

## Quick Reference Checklist

```
Setup Checklist:
[ ] Created src/context/index.jsx
[ ] Created Context with createContext()
[ ] Created Provider component with state
[ ] Added value={{state, methods}} to Provider
[ ] Exported both MyContext and MyProvider
[ ] Wrapped App with <MyProvider> in main.jsx
[ ] Imported useContext and MyContext in components
[ ] Called useContext(MyContext) in components
[ ] Accessed values as context.stateName or context.methodName
```

---

## Memory Tricks 🧠

### **"CREATE, PROVIDE, CONSUME"**
1. **CREATE**: `createContext()`
2. **PROVIDE**: Wrap app with `<MyContext.Provider>`
3. **CONSUME**: Use `useContext(MyContext)` in components

### **"Value is a Package"**
Think of the Context value as a package you're mailing:
```jsx
value={{
  // 📦 Package contents (what you're sharing)
  state_data: myState,
  action_methods: myFunction,
}}
```

### **"Props Drilling → Context Highways"**
```
Without Context (props drilling):
Comp A → Comp B → Comp C → Comp D 
(data travels through all levels)

With Context (data highway):
Comp A ──┐
         ├→ Comp B (access directly)
         ├→ Comp C (access directly)  
         └→ Comp D (access directly)
```

---

## Real-World Use Cases

✅ **Perfect for Context:**
- Authentication (user logged in, user data)
- Theme (dark mode, light mode)
- Language/Localization
- Global UI state (modal open/close, notifications)
- Shopping cart data
- App-wide settings & preferences
- User session information

❌ **Not ideal for Context:**
- Frequently changing data (use Redux/Zustand)
- Large forms (use local state)
- Temporary UI state (use local state)
- Real-time data (use other state management)

---

## Resources & Review

**Remember the flow:**
```
main.jsx: Wrap <MyProvider>
   ↓
context/index.jsx: Define MyContext & MyProvider
   ↓
App.jsx & Components: import { useContext } from 'react'
   ↓
const context = useContext(MyContext)
   ↓
Use: context.stateName or context.methodName()
```

---

**Last Updated:** Feb 2026  
**Status:** Ready to use & reference ✅
