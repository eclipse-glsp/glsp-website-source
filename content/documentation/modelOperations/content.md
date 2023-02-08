+++
fragment = "content"
weight = 100

title = "Model Operations"

[sidebar]
  sticky = true
+++

### Operations are just special actions

In the previous section, we discussed [actions and action handlers]({{< relref  "actionhandler" >}}) as the general way of how a diagram client and a diagram server communicate with each other and how they can invoke behavior or query data from each other.
If such a behavior of an action now is intended to change the [underlying source model]({{< relref "sourceModel" >}}) on the server, there is a dedicated type of action for that: _a model operation_ or operation for short.

An operation is a special type of action denoting a request for performing a specific modification of the source model.
An operation `kind` identifier declares the specific modification to be applied.
GLSP defines a set of reusable pre-defined operation types, such as for adding and deleting nodes and edges (see [protocol](/documentation/protocol/) for a detailed list).
This pre-defined set of operations, however, can be easily extended by custom operation types.
Each operation type can define additional attributes, such as the coordinates on which a node is to be created, that are required to perform the model modification.
Note, however, that many pre-defined operation types support generic arguments (see property `args` in the type definitions), which allow to pass on simple, custom data with the existing pre-defined operations without the overhead of defining custom operation types.

In the following, we show an example of defining a custom operation.

<details open><summary>GLSP Client/Node GLSP Server</summary>

```ts
export interface MyOperation extends Operation {
    kind: MyOperation.KIND;
    elementId: string;
    location?: Point;
}

export namespace MyOperation {
    export const KIND = 'runMyOperation';
    export function create(
        elementId: string,
        options: { location?: Point } = {}
    ): MyOperation {
        return {
            kind: KIND,
            isOperation: true,
            elementId,
            ...options
        };
    }
}
```

</details>

<details closed><summary>Java GLSP Server</summary>

```java
public class MyOperation extends Operation {

   private String elementId;
   private GPoint location;

   public MyOperation(final String elementId) {
    this(elementId, null);
   }

   public MyOperation(final String elementId, final GPoint location) {
      this();
      this.elementId = elementId;
      this.location = location;
   }

   public MyOperation() {
      super("runMyOperation");
   }

   public String getElementId() { return elementId; }

   public void setElementId(final String elementId) { this.elementId = elementId; }

   public Optional<GPoint> getLocation() { return Optional.ofNullable(location); }

   public void setLocation(final GPoint location) { this.location = location; }

}
```

</details>
</br>

As an operation is just a special type of action, it can be invoked as any other action via the action dispatcher.

```ts
@inject(TYPES.IActionDispatcher) protected actionDispatcher: GLSPActionDispatcher;
…
this.actionDispatcher.dispatch(MyOperation.create(element.id, { location: { ... } }));
```

### Operation handlers vs. operation action handler

Only the server is capable of performing source model modifications and, thus, operations must always be handled on the server only.
Therefore, there is a single action handler on the server that handles all types of operations: the `OperationActionHandler`.
This single operation action handler, however, delegates the execution of the actual source model modification to one of the specific `OperationHandler` implementations registered for specific operation types on the server.

In summary, the operation action handler manages the steps to be done before and after any model manipulation, whereas the actual source model manipulations are implemented in operation handlers that can be registered for specific operation types.

### Operation handlers

Operation handlers are implementations of a specific type of source model manipluations, such as adding or deleting nodes.
Thus, they are registered for a specific operation kind in the diagram server modules.
Of course, operation handlers are typically specific to the respective source model language and framework (i.e. the data format, metamodel, or model API), as they implement how the current source model needs to be changed for a specific operation kind.
Consequently, the actual implementation is specific to your particular scenario.

The registration and basic structure of operation handlers is set up as follows:

<details open><summary>Java GLSP Server</summary>

```java
public class MyServerModule extends DiagramModule {
  ...
  @Override
   protected void configureOperationHandlers(final MultiBinding<OperationHandler> binding) {
    binding.add(MyOperationHandler.class);
   }
}

public class MyOperationHandler extends AbstractOperationHandler<MyOperation> {
   @Inject
   protected MyModelState modelState;

   @Override
   protected void executeOperation(final MyOperation operation) {
    // modify your model state here
   }
}
```

</details>

<details><summary>Node GLSP Server</summary>

```ts
@injectable()
export class MyServerModule extends DiagramModule {
  ...
  configureOperationHandlers(binding: InstanceMultiBinding<OperationHandlerConstructor>): void {
    binding.add(MyOperationHandler);
  }
}

@injectable()
export class MyOperationHandler implements OperationHandler {
    readonly operationType = MyOperation.KIND;

    @inject(MyModelState)
    protected modelState: MyModelState;

    execute(operation: MyOperation): MaybePromise<void> {
        // modify your model state here
    }
}
```

</details>
</br>

### Operation action handler

The operation action handler manages the steps to be done before and after any model manipulation.
Usually, the operation action handler typically runs the operation handler's modification within the scope of a command stack, to support undo and redo and handles all other aspects that need to be taken care of for any type of modification.
For instance, after each successful source model modification done by an `OperationHandler`, the `OperationActionHandler` triggers the regeneration of the graphical model by calling the [`GModelFactory`]({{< ref "gmodel" >}}) and sends the updated graph model back to the client using an `UpdateModelAction`, which then renders the update of the diagram.
See also [the process of editing the source model]({{< ref "sourceModel" >}}#processing-edit-operations).

Also, the operation action handler is a good place in general to introduce any other steps that need to be performed before or after each model manipulation, such as triggering live validation, sending notifications to external services, etc.
Thus, it isn't unusual to overwrite the operation action handler for specific scenarios to inject before/after modification steps.

The operation action handler is typically agnostic of the actual source model language; it usually is, however, specific to the source model framework (e.g. EMF, JSON, etc.) as it needs to encapsulate the modification in framework-specific command API to support native undo/redo, or take snapshots before and after the manipulation and compute a patch, etc.
For the available [source model framework integration]({{< ref "integrations" >}}#source-model-integrations) of GLSP (EMF, GModel-only, and JSON), a dedicated operation action handler, which often also entails a specific interface for the operation handler implementations, is provided by the respective GLSP source model libraries.

The respective default implementation of the operation action handler can be overridden in the diagram module by rebinding the implementation bound to the `OperationActionHandler` interface.
