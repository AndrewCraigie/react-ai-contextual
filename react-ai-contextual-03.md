# React AI Contextual Application - Architecture Refinement

## Overview
This document captures the architectural refinement discussion following the initial pattern decisions, focusing on implementation organization and API design.

## Library Parts/Components Identified

Based on the chosen architecture (mediated command pattern with dynamic registration), the following parts are needed:

### 1. **Registry Manager**
- Maintains dynamic list of registered components
- Stores: component IDs, capabilities, callbacks, metadata
- Handles registration/unregistration lifecycle
- Provides query methods

### 2. **Capability Aggregator**
- Compiles current system capabilities from all registered components
- Deduplicates and organizes methods
- Maintains "methods" list and "features" mapping
- Provides capability manifest for API communication

### 3. **Communication Layer**
- WebSocket/SSE connection management
- Message serialization/deserialization (JSON-RPC format)
- Request/response correlation tracking
- Connection retry logic and error handling

### 4. **Message Router/Dispatcher**
- Parses incoming AI responses
- Matches responses to registered components via correlation IDs
- Orchestrates multi-step operations
- Routes to UI components or registered callbacks

### 5. **UI Components**
- ChatbotContainer, FAB, MessageDisplay, InputControls
- Rich controls for user confirmation/actions
- Registers itself using same pattern as external components

### 6. **Context Provider**
- Exposes registration/unregistration methods
- Provides communication methods
- Manages overall library state
- Wraps consuming application

### 7. **Command Executor**
- Interprets AI responses as executable commands
- Manages user confirmation flow
- Executes callbacks with error handling
- Handles command queuing/sequencing

## Code Organization Patterns Considered

### Pattern 1: Multiple Nested Contexts
```jsx
<CommunicationProvider>
  <RegistryProvider>
    <CapabilityProvider>
      <CommandProvider>
        {children}
      </CommandProvider>
    </CapabilityProvider>
  </RegistryProvider>
</CommunicationProvider>
```
- **Pros**: Clear separation, single responsibility
- **Cons**: Provider hell, performance issues, complex dependencies

### Pattern 2: Single Context + Custom Hooks
```jsx
<AIContextProvider>
  {children}
</AIContextProvider>

// Modular hooks
useRegistry()
useCapabilities()
useCommunication()
```
- **Pros**: Clean API, avoids nesting, React-like
- **Cons**: Large context might cause unnecessary re-renders

### Pattern 3: Service Pattern with Hooks
```jsx
// Services exist outside React
const registryService = new RegistryService()
const communicationService = new CommunicationService()

// Single context holds service instances
<AIContextProvider services={{registry, communication}}>
```
- **Pros**: Testable business logic, familiar pattern
- **Cons**: Less React-like, lifecycle management complexity

### Recommended Approach: Hybrid
```
// Pure business logic modules
- /lib/registry.ts
- /lib/capabilities.ts  
- /lib/communication.ts

// Single context that orchestrates
- /contexts/AIContext.tsx

// Focused hooks that expose functionality
- /hooks/useRegistration.ts
- /hooks/useCapabilities.ts
```

## Final API Design Decision

### Minimalist Consumer-Facing API

**Single Provider:**
```jsx
<AIProvider>
  {children}
</AIProvider>
```

**Single Hook with Minimal Surface:**
```jsx
const {
  // State
  isConnected,
  isLoading,
  error,
  
  // Methods (just these two!)
  registerComponent,
  unregisterComponent
} = useAIContext();
```

### Registration Object Structure
```jsx
registerComponent({
  id: "shipping.AddressForm",
  name: "Shipping Address Form",
  capabilities: {
    "form.suggest": {
      description: "Auto-fill shipping address fields",
      fields: ["street", "city", "state", "zip"],
      handler: (data) => {
        // Developer's callback to fill their form
        setFormValues(data);
      }
    },
    "form.clear": {
      description: "Clear all form fields",
      handler: () => {
        // Developer's callback to clear
        resetForm();
      }
    }
  }
});
```

### Usage Example for Developers
```jsx
function MyForm() {
  const { registerComponent, unregisterComponent } = useAIContext();
  const [formData, setFormData] = useState({});

  useEffect(() => {
    const registration = registerComponent({
      id: "myapp.ContactForm",
      name: "Contact Form",
      capabilities: {
        "form.suggest": {
          description: "Auto-fill contact information",
          fields: ["name", "email", "phone"],
          handler: (data) => {
            setFormData(data);
          }
        }
      }
    });

    // Cleanup
    return () => {
      unregisterComponent(registration);
    };
  }, []);

  return (
    // Form JSX
  );
}
```

## Internal Architecture (Hidden from Consumers)

```
AIProvider
  ├── Registry (internal module)
  ├── Capabilities (internal module)
  ├── Communication (internal module)
  ├── Router (internal module)
  ├── CommandExecutor (internal module)
  └── UI Components (Chatbot, FAB)
```

All complexity is hidden behind the simple registration API. Developers only need to understand:
1. Wrap app with `<AIProvider>`
2. Register components with capabilities and handlers
3. Unregister on cleanup

## Key Design Principles
- **Minimal API Surface**: Just two methods exposed
- **Capability-Centric**: Each capability has its own handler
- **Self-Documenting**: Registration object structure is clear
- **Zero Configuration**: Works out of the box when wrapped
- **Progressive Enhancement**: Components opt-in to AI capabilities

## Benefits
- Dead simple mental model for developers
- All complexity hidden in internal modules
- Testable business logic separated from React
- Maintains React patterns while organizing business logic
- Easy adoption across teams with minimal learning curve