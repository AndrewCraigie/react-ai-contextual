# React AI Context Aware Application

This document is for capturing findings during the ideation stages of a project where the end goal is to build a React application that can communicate via a backend API to an LLM or AI agentic framework. This document is not for detailing concrete architectures or impossibilities and findings that may spark new ideas or help coalesce the thinking leading towards a more refined, concrete implementation plan. Any code snippets that may be included in this document DO NOT represent suggested approaches and any components that have been given names in any code snippets DO NOT represent any final names to be used or naming conventions.

## Ideation Phase

### Overall Guiding Principles

- Modularity
- Separation of concerns
- Avoid React 'anti-patterns' - do things the 'React way' and avoid trying to shoe-horn in approaches from other frameworks that are not commonly used in React applications
- Maintainability
- Testability
- Simplicity
- Functional not class based

### General, rough-and-ready nebulous ideas/requirements

- A React library that exports a single context/provider (referred to as the 'library application')
- The single context/provider is to be imported in multiple product projects within a larger project mono-repo
- The context/provider will 'wrap' each product project application with no injected dependencies (developers simply add the context/provider to their product application and it will work out of the box)
- The library application will provide the following UI features:
  - A ChatbotContainer component that contains child components that includes the chatbot UI implementation, a message history display, text input, send button (other related atomic components)
  - A FAB (a floating button that is what the user clicks to open the chatbot)

A possible (not definitive) return for the provider may be:

```jsx
<MyProvider>{/* maybe pass in at this point details of the product application e.g. name, id... */}
  {children}
  <ChatbotContainer/>
  <Fab/>
</MyProvider>
```

The single exported context/provider may or may not in turn use/contain further contexts/providers but equally may use some other method for 'separation of concerns'.

### Initial (and still being ideated) thoughts on the main 'parts' and requirements needed within the library application

- All parts should be well separated using suitable architectural approaches e.g. contexts, component nesting, modular, isolated, swappable, defined boundaries and/or application interfaces
- All parts should be testable (where possible)
- Favour modularity, use pure functions where possible, small composable 'chunks' of code/logic

#### Communication (handling communication with a backend API)

- It is likely either websockets or server side events (SSE) will be used for requests/responses
- Ideally created in a way that the communication approach can be either decided upon later or swapped if necessary
- Not coupled to other parts of the library application but with a clear, well defined interface
- Handles connections, sending/receiving requests, API errors, retries
- Requests and Responses format: initial (but open to other ideas) suggestion is to use JSON-RPC as it appears to be well suited for context-aware messaging to and from AI enabled LLM or agentic frameworks
- Will not directly communicate with an LLM or AI agentic framework but with an API that then in turn communicates with an LLM or agentic framework (the backend API is not in scope for consideration in this document)

#### Capabilities

Some way of storing, handling application and component 'capabilities' to allow the LLM or AI agentic framework to 'know' about this application/components:

- Initial research on AI capabilities reporting and use suggest possibly adopting a similar approach to that used in MCP servers and in JSON-RPC e.g. chat.send, chat.stream, form.suggest, form.reset
- Possibly maintain some form of dynamic list of 'methods' and 'features' similar to that used with MCP servers/JSON-RPC
- Dynamic: Other than the chatbot implementation that will come with the library application, the product application that use the library application should be able to register and de-register their capabilities
- If a dynamic component registration approach is decided upon then even though the chatbot is implemented within the library application it should probably also just use the same registration approach and be considered a component like any that may be present in a product application
- Methods for adding/removing capabilities either directly or through some form of registration 'part'
- Methods that may be needed to get, collate, resolve capabilities to send as part of requests to the API

#### Registration

If a 'capabilities' approach is decided upon then some way of components being able to tell the library component what their capabilities are at any point in time will be needed:

- Possibly include methods to allow a component (via a context method) to say 'please register me and here are my capabilities, id, name, purpose etc.'
- Possibly include methods for a component to un-register (either the component does not want to be known about, or it may no longer be in the DOM so needs to unregister before unmounting as a clean up in a useEffect)
- Functionality may or may not be in the same 'part' as capabilities but ideally separate for a cleaner, more maintainable, testable code base

#### Chatbot (UI components to implement the chatbot feature)

- Atomic component design
- Where possible separation of business logic and presentation logic e.g. container/presentation pattern
- Will at least need the following features:
  - 'Drawer': the chatbot will open to the side and on top of any product project that is using the library component
  - A FAB: button like control that floats at the bottom right of any product project that is using the library component that the user clicks to open the chatbot drawer
  - A message input and send button
  - Message display area following typical 'chatbot' approaches
  - User messages
  - Single or multiple streamed responses from the API (LLM or agentic framework)
  - Utility controls e.g. copy, clear, new chat
  - Possibly will need 'rich controls' like buttons to allow the user to accept/trigger suggested actions
  - 'Thinking' messages or indicator(s)
- Chatbot drawer is a known requirement but there may also be the need for a full screen implementation as well or some form of 'expansion' of the drawer to full screen
- Flexible and responsive (and accessible as much as possible)

#### Signalling, routing or some form of middleware

- When a message or stream is received that is 'aimed' at let's say the chat then there needs to be some form of manager or routing code that will get the message to the right component and/or set the correct state
- This feature, its requirements and possible approaches to be used is the least known and very much still needs research, ideas and consideration
- May or may not be a single 'part' or may be multiple

#### Response to Component

Possibly all part of whatever signalling, routing or middleware implementation: some way of being able to receive a response and then perform the appropriate action.

**Example 1:** A response to a chat.send type message is received, some logic figures out this message is supposed to be displayed by the chatbot in the message display area so it finds out what component handles this and somehow sends the message/instruction to that component or possibly sets some state that the component will then use.

**Example 2:** A response to a form.suggest type message received as the LLM or AI agentic framework has been informed there is a form relevant to the message the user entered and has sent back the suggested auto fill values. Some logic figures this out, tells the chatbot to show a message telling the user "I can help you fill out x form, do you want me to populate the fields for you?" When the user clicks Yes, some logic then co-ordinates forwarding this to the form to say "Here are some values for the fields you have, please enter these suggested values."

## Other Questions/Issues/Areas for Research

Along with all the, as yet undecided or unknown features and possible approaches listed above there are other questions/issues/areas for research:

### What's the best approach for state management?

**No use of other libraries:**
- Simple useState use suggested as this is fairly standard/known by developers in the team
- May consider a reducer approach but only if it solves a big problem or is considerably better/cleaner/more efficient. Don't want a monolithic state object if possible

**What state needs/must be in the library component:**
- Likely: capabilities/registrations, open/close state of chatbot/FAB
- Maybe: response history?
- Likely Not: managed state for forms (see next point below)

**State management ideas:**
- One idea was that when a component registers they also say "There's a chunk of state I'd like you to manage", and then they would use that state for controlled components within them e.g. form fields, chat history display. Then when a response is received relevant to that component the controlling logic can simply set that portion of state and not need to "inform" the component in any way as React will take care of any updates. But this approach seems verging towards an anti-pattern of some sort (unless it's not, then this may be the way to go) - totally unsure about this.

**Registration callback approach:**
- An idea related to registration of components is that when they register they not only give information about themselves and their capabilities but for each capability, where necessary, also provide a callback function. So for example "I'm Form A and I support form.suggest, my fields are x, y, z. If you receive a message for form.suggest for Form A form.suggest, here's the callback function to call in which you can send the data as parameters and I'll take care of the rest."

### Architecture Patterns

Do any of the ideas or features being thought about in this document fall under known application or software 'patterns' or named architectures? If so, it can often be useful to actually name the approach or pattern being followed for research, documentation and implementation ideas.
