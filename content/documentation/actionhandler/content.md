+++
fragment = "content"
weight = 100

title = "Actions & Action Handler"

[sidebar]
  sticky = true
+++

### Overview

The client and the server communicate bidirectionally by sending actions via JSON-RPC.
In addition, they are also used for the internal event flow in both the *GLSP server* and the *GLSP client*.
Any service, mouse tool, etc. can issue actions by invoking the action dispatcher, either on the client or the server.

The action dispatcher – there is one on the client and one on the server – is the central component responsible for dispatching actions to their designated action handlers.

<p align="center">
<img src="action-handler.png" alt="Action Dispatching & Handlers"  />
</p>

When the dispatcher receives a new action for dispatching, it determines whether it should be dispatched to the internal action handlers only or submitted to the opposite component via JSON-RCP (server or client), based on the registered handlers on the server or the client.

The dispatcher distinguishes between notifications and request-response action pairs.
Notification actions are one-way actions transferred between client and server.
This means when the action dispatcher dispatches a notification it does not wait for a response and directly continues with dispatching the next incoming action.
Request actions are typically issued by the *GLSP client* and can be used to block client-side action dispatching until the server has sent a corresponding response action.

GLSP defines the standard action types of the [graphical language server protocol](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md).
However, adopters can add new custom action types.
Besides, adopters can replace and extend existing, or add additional action handlers for standard or custom action types.

To do that the following steps have to be performed:

1. Create a new action specification by providing a corresponding `Action` implementation
2. Create a new action handler for the newly created action type by providing a implementation of the `ActionHandler` interface
3. Configure the new action type and handler in the DI module.

### Action specification

Adopters can declare new custom actions by providing an implementation for the `Action` interface (resp. base class).

<details open><summary>Java GLSP Server</summary>

```java
public class MyCustomAction extends Action {
   public static final String KIND= "myCustomKind";
   private String additionalInformation;


   public MyCustomAction() {
      super(KIND);
   }

   public String getAdditionalInformation() { return additionalInformation; }

   public void setAdditionalInformation(final String additionalInformation) {
      this.additionalInformation = additionalInformation;
   }
}
```

</details>

<details open><summary>GLSP Client/Node GLSP Server</summary>

```ts
export class MyCustomAction implements Action {
    static readonly KIND = 'myCustomKind';
    kind = MyCustomAction.KIND;

    constructor(public readonly additionalInformation: string) {
        
    }
}
```
</details>
</br>

Each action specification has a unique “kind” and can optionally declare additional data properties
We recommend defining the action kind as a static constant of the implementing class so that it can be accessed from other places, e.g. when registering the handler.
Note that action instances need to be serializable to JSON.
Therefore the class should only contain plain data properties and no additional business logic.
In addition, references to graphical model elements should be done by id.

If an action is interchanged between client and server both need to provide the corresponding action definition.

#### Request-Response Actions

If the client should be able to dispatch the new action as a blocking request, the action specification class has to implement or extend `RequestAction`.

<details open><summary>Java GLSP Server</summary>

```java
public class MyCustomRequestAction extends RequestAction<MyCustomResponseAction> {
   public static final String KIND= "myCustomRequest";
   private String additionalInformation;

   public MyCustomRequestAction() {
      super("my.custom.kind");
   }

   public String getAdditionalInformation() { return additionalInformation; }

   public void setAdditionalInformation(final String additionalInformation) {
      this.additionalInformation = additionalInformation;
   }
}
```

</details>

<details open><summary>GLSP Client/Node GLSP Server</summary>

```ts
export class MyCustomRequestAction implements RequestAction<MyCustomResponseAction> {
    static readonly KIND = 'myCustomRequest';
    kind = MyCustomRequestAction.KIND;

    constructor(public readonly additionalInformation: string, public readonly requestId = '') {}
}
```

</details>
</br>

Each request action has a “requestId” and defines its response action as a type parameter.
Of course, the response action specification has to be specified as well:


<details open><summary>Java GLSP Server</summary>

```java
public class MyCustomResponseAction extends ResponseAction {
   public static final String KIND = "myCustomResponse";

   public MyCustomResponseAction() {
      super(KIND);
   }
}
```

</details>

<details open><summary>GLSP Client/Node GLSP Server</summary>

```ts
export class MyCustomResponseAction implements ResponseAction {
    static readonly KIND = 'myCustomResponse';
    kind = MyCustomResponseAction.KIND;

    constructor(public responseId = '') {}
}

```

</details>
</br>

The client can dispatch a request action either in blocking fashion awaiting the response:

```ts
@inject(TYPES.IActionDispatcher) protected actionDispatcher: GLSPActionDispatcher;
…
const response = await this.actionDispatcher.request(new MyCustomRequestAction(“info”));
// response is of type MyCustomResponseAction
```

or simply dispatch the action as non-blocking notification:

```ts
@inject(TYPES.IActionDispatcher) protected actionDispatcher: GLSPActionDispatcher;
…
this.actionDispatcher.dispatch(new MyCustomRequestAction(“info”));
```

Response actions don’t necessarily have to be part of a response-request action pair and can also be dispatched without a preceding request action.
</br>

### Implementing an Action Handler (GLSP Server)

To create a new action handler, a class that implements the `ActionHandler` interface has to be created.
In general, an action handler can handle one or more action kinds.
However, handling multiple action kinds is typically reserved for rather uncommon edge cases. 
Therefore, the Java GLSP server provides an abstract base class that is designed for the single-action-kind-per-hanlder use case.

<details open><summary>Java GLSP Server</summary>

```java
public class MyCustomActionHandler extends AbstractActionHandler<MyCustomResponseAction> {

   @Override
   protected List<Action> executeAction(final MyCustomResponseAction actualAction) {
      // implement your custom logic to handle the action

      // Finally issue response actions
      // If no response actions should be issued 'none()' can be used;
      return listOf(new MyCustomResponseAction());
   }

}
```

</details>

<details><summary>Node GLSP Server</summary>

```ts
@injectable()
export class MyCustomActionHandler implements ActionHandler {
    actionKinds = [MyCustomRequestAction.KIND];

    execute(action: MyCustomRequestAction): MaybePromise<Action[]> {
        // implement your custom logic to handle the action

        // Finally issue response actions
        // If no response actions should be issued '[]' can be used;
        return [new MyCustomResponseAction()];
    }
}
```

</details>
</br>

The `executeAction()` method has to be implemented to provide the custom logic of your action handler.
It returns a set of response actions that should be dispatched after the handler execution.

Next, the custom handler has to be configured in the `DiagramModule`:

<details open><summary>Java GLSP Server</summary>

```java
@Override
protected void configureActionHandlers(final MultiBinding<ActionHandler> binding) {
    super.configureActionHandlers(binding);
    binding.add(MyCustomActionHandler.class);
}
```

</details>

<details><summary>Node GLSP Server</summary>

```ts
protected override configureActionHandlers(binding: InstanceMultiBinding<ActionHandlerConstructor>): void {
    super.configureActionHandlers(binding);
    binding.add(MyCustomActionHandler);
}
```

</details>
</br>

In addition, each response action of the handler that should be handled by the _GLSP client_ has to be configured as dedicated client action to indicate that it needs to be dispatched to the client:

<details open><summary>Java GLSP Server</summary>

```java
protected void configureClientActions(final MultiBinding<Action> binding) {
    super.configureClientAction(binding);
    binding.add(MyCustomRequestAction.class);
}
```

</details>

<details><summary>Node GLSP Server</summary>

```ts
protected configureClientActions(
    binding: InstanceMultiBinding<string>
): void {
    super.configureClientActions(binding);
    binding.add(MyCustomRequestAction.KIND);
}
```

</details>
</br>

#### Request-Response Handling

Action handlers can treat request-response actions in the same way as plain actions.
No special handling is required.
The action dispatcher tracks all incoming request actions and automatically intercepts the corresponding response action to set the correct response id.
</br>

### Implementing an Action Handler (GLSP Client)

On the client, GLSP reuses the `IActionHandler` API of [Sprotty](https://github.com/eclipse/sprotty).
Therefore, to create a new action handler, a class that implements the `IActionHandler` interface has to be created.

```ts
@injectable()
export class MyCustomResponseActionHandler implements IActionHandler{
    handle(action: MyCustomResponseAction): void | Action {
        // implement your custom logic to handle the action
        // Optionally issue a response action
    }
}
```

The `handle()` method has to be implemented to provide the custom logic of your action handler. 
It optionally returns a response action that should be dispatched after the handler execution.

A dedicated configuration function is available to configure the new action handler in the diagram module (“di.config.ts”):

```ts
const diagramModule = new ContainerModule((bind, _unbind, isBound, rebind) => {
    const context = { bind, _unbind, isBound, rebind };
    configureActionHandler(context, MyCustomResponseAction.KIND, MyCustomResponseActionHandler);
}
```

The `configureActionHandler()` function takes the inversify binding context, the action kind that should be handled, and the action handler class, as input.
It registers the action handler for the given action kind, so that it can be retrieved by the action dispatcher.

Note that we don’t have to explicitly declare which actions are handled by the GLSP Server. 
The GLSP server sends this information during the initialization process and the GLSP client automatically sets up the necessary action (handler) registrations.
