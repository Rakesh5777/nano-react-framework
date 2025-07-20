# 🚀 Nano React Framework

A minimal React-like framework implementation that demonstrates the core mental model and internal workings of React. This educational project helps developers understand how React manages state, handles component rendering, and implements hooks under the hood.

## 🎯 Purpose

This repository serves as a learning tool to understand:
- How React's state management works internally
- The implementation of React hooks (useState, useEffect)
- Component lifecycle and re-rendering mechanisms
- State isolation between multiple component instances
- The virtual DOM concept (simplified)

## 🌟 What You'll Learn

By studying this implementation, you'll gain insights into:

1. **React's Mental Model**: How React thinks about components, state, and updates
2. **Hook Implementation**: The actual mechanics behind useState and useEffect
3. **State Management**: How React isolates state between component instances
4. **Rendering Process**: How React decides when and what to re-render
5. **Event Handling**: How user interactions trigger state updates and re-renders

## 🚀 Quick Start

1. **Clone the repository**
   ```bash
   git clone https://github.com/Rakesh5777/nano-react-framework.git
   cd nano-react-framework
   ```

2. **Run the demo**
   ```bash
   # Option 1: Simple HTTP server
   python3 -m http.server 8000
   
   # Option 2: Using Node.js
   npx serve .
   
   # Option 3: Using PHP
   php -S localhost:8000
   ```

3. **Open in browser**
   ```
   http://localhost:8000
   ```

## 🏗️ Architecture Overview

The Nano React Framework consists of three main parts:

### 1. Core Framework (`NanoReact`)
```javascript
const NanoReact = (() => {
    let componentStates = new Map();        // State storage per component
    let currentlyRenderingComponent = null; // Current rendering context
    
    // Hook implementations
    function useState(initialValue) { /* ... */ }
    function useEffect(callback, depArray) { /* ... */ }
    function render(Component, container) { /* ... */ }
})();
```

### 2. Component Definitions
```javascript
function Counter1() {
    const [count, setCount] = useState(0);
    return `<div class="card">Count: ${count}</div>`;
}
```

### 3. Event System
```javascript
window.increment1 = () => App.setters.setCount1(c => c + 1);
```

## 📚 Deep Dive: How It Works

### State Management - The Core Problem

**Traditional Problem**: In naive React implementations, all components share the same state array:

```javascript
// ❌ WRONG: All components share state
let hooks = [];
let hookIndex = 0;

function useState(initial) {
    hooks[hookIndex] = hooks[hookIndex] || initial;
    return [hooks[hookIndex++], /* setter */];
}
```

**Our Solution**: Each component gets its own state storage:

```javascript
// ✅ CORRECT: Each component has isolated state
let componentStates = new Map(); // Key: container, Value: component state

function useState(initialValue) {
    const state = componentStates.get(currentlyRenderingComponent);
    // Each component maintains its own hooks array
}
```

### useState Implementation

```javascript
function useState(initialValue) {
    const state = componentStates.get(currentlyRenderingComponent);
    const hookIndex = state.hookIndex;
    
    // Initialize if first time
    if (state.hooks[hookIndex] === undefined) {
        state.hooks[hookIndex] = initialValue;
    }

    const currentIndex = hookIndex;
    const componentId = currentlyRenderingComponent;
    
    const setState = (action) => {
        const componentState = componentStates.get(componentId);
        const oldValue = componentState.hooks[currentIndex];
        const newValue = typeof action === 'function' ? action(oldValue) : action;
        componentState.hooks[currentIndex] = newValue;
        
        // Re-render only the specific component
        render(componentId.Component, componentId);
    };

    state.hookIndex++;
    return [state.hooks[hookIndex], setState];
}
```

**Key Concepts Demonstrated:**
- **Closure Capture**: The `setState` function captures the component's identity
- **Hook Index**: Each hook call gets its own position in the hooks array
- **Selective Re-rendering**: Only the component that changed gets re-rendered

### useEffect Implementation

```javascript
function useEffect(callback, depArray) {
    const state = componentStates.get(currentlyRenderingComponent);
    const hookIndex = state.hookIndex;
    const oldHookData = state.hooks[hookIndex];
    const oldDeps = oldHookData ? oldHookData.deps : undefined;
    
    let hasChanged = true;
    if (oldDeps) {
        hasChanged = depArray.some((dep, i) => !Object.is(dep, oldDeps[i]));
    }

    if (hasChanged) {
        // Cleanup previous effect
        if (oldHookData && oldHookData.cleanup) {
            oldHookData.cleanup();
        }
        
        // Run new effect
        const newCleanup = callback();
        state.hooks[hookIndex] = { deps: depArray, cleanup: newCleanup };
    }
    
    state.hookIndex++;
}
```

**Key Concepts Demonstrated:**
- **Dependency Comparison**: Using `Object.is()` for precise equality checking
- **Cleanup Functions**: Proper cleanup of previous effects
- **Effect Scheduling**: Effects run only when dependencies change

### Rendering Process

```javascript
function render(Component, container) {
    // Set current rendering context
    currentlyRenderingComponent = container;
    
    // Initialize component state if first render
    if (!componentStates.has(container)) {
        componentStates.set(container, { hooks: [] });
        container.Component = Component;
    }
    
    // Reset hook index for this render
    componentStates.get(container).hookIndex = 0;
    
    // Execute component function
    const output = Component();
    container.innerHTML = output;
    
    // Clear rendering context
    currentlyRenderingComponent = null;
}
```

**Key Concepts Demonstrated:**
- **Rendering Context**: Tracking which component is currently rendering
- **Hook Index Reset**: Each render starts from hook index 0
- **Component Memoization**: Storing component reference for re-renders

## 🎨 Demo Application

The demo showcases two independent components:

### Component 1: Simple Counter
```javascript
function Counter1() {
    const [count, setCount] = useState(0);
    return `
        <div class="card text-center space-y-4">
            <h1>Component One</h1>
            <p class="text-6xl font-bold text-blue-600">${count}</p>
            <button onclick="window.increment1()">Increment C1</button>
        </div>
    `;
}
```

### Component 2: Multi-State Counter
```javascript
function Counter2() {
    const [count, setCount] = useState(100);
    const [text, setText] = useState("Nano");
    
    return `
        <div class="card text-center space-y-4">
            <h1>Component Two</h1>
            <p class="text-6xl font-bold text-teal-600">${count}</p>
            <p class="text-xl font-medium text-gray-500">${text}</p>
            <button onclick="window.decrement2()">Decrement C2</button>
            <input type="text" value="${text}" oninput="window.changeText2(this.value)" />
        </div>
    `;
}
```

## 🧪 Try These Experiments

1. **State Isolation Test**: 
   - Click increment on Component 1
   - Notice Component 2's state remains unchanged
   - This demonstrates proper state isolation

2. **Multiple State Variables**:
   - Modify the text input in Component 2
   - Notice the counter remains unchanged
   - This shows multiple useState calls work correctly

3. **Re-rendering Scope**:
   - Open browser DevTools and watch console logs
   - See how only the modified component re-renders

4. **Hook Order Consistency**:
   - Try adding conditional useState calls and see errors
   - This demonstrates why "Rules of Hooks" exist in React

## 🔍 Key Differences from Real React

| Feature | Nano React | Real React |
|---------|------------|------------|
| Virtual DOM | String-based rendering | Fiber reconciliation |
| State Storage | Map with component containers | Fiber nodes |
| Scheduling | Immediate re-render | Prioritized scheduling |
| Event System | Global handlers | SyntheticEvents |
| Performance | No optimizations | Memoization, batching |

## 🎓 Learning Outcomes

After studying this implementation, you should understand:

1. **Why hooks have rules** (order must be consistent)
2. **How React isolates component state** (each component gets its own storage)
3. **Why React can be "magical"** (closures capture component identity)
4. **How selective re-rendering works** (only modified components update)
5. **The foundation of more complex features** (context, reducers, etc.)

## 🚀 Next Steps

To deepen your understanding:

1. **Add more hooks**: Implement useContext, useReducer, or useMemo
2. **Improve performance**: Add memoization or batching
3. **Study real React**: Compare with React's actual source code
4. **Build components**: Create more complex UI components
5. **Add testing**: Write tests for the framework functionality

## 🤝 Contributing

This is an educational project! Feel free to:
- Add more hook implementations
- Improve documentation
- Create additional examples
- Fix bugs or edge cases

## 📄 License

This project is open source and available under the MIT License.

---

**Happy learning! 🎉**

Remember: The goal isn't to replace React, but to understand how React thinks and works internally. This knowledge will make you a better React developer!