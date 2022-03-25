+++
fragment = "content"
weight = 100

title = "How to update a Model Element"

[sidebar]
  sticky = true
+++

If you want to change values for some element properties on the client side, you can send an action event to the model server to apply the new values to your model. For this purpose you need to implement a corresponding GLSP Server Action so that your client can send an update. On the Server side you need to handle the action and update your GModel.

For the following example let's assume that you have a model element named 'Event' with the properties 'name' and 'documentation'. And you have some part on your client that adds new information to the property 'documentation'. Let's see how this will work from the server side:


### The Apply Operation

First of all you need to define an Apply Operation on your Server which accepts the ID of the element and an expression describing what changed. This action is later send from your client to the server: 


````java
public class ApplyEventUpdateOperation extends Operation {

    private String id;
    private String expression;

    public ApplyEventUpdateOperation() {
        super("applyEventUpdate");
    }

    public ApplyEventUpdateOperation(final String nodeId, final String expression) {
        this();
        this.id = nodeId;
        this.expression = expression;
    }

    public String getId() {
        return id;
    }

    public void setId(final String nodeId) {
        this.id = nodeId;
    }

    public String getExpression() {
        return expression;
    }

    public void setExpression(final String expression) {
        this.expression = expression;
    }
}
````

In this example we use an expression holding the name of the property and the new value:

	documentation:My New Doc....
	
Or you can also describe your changes into some kind of XML or other format.  


### The Apply Operation Handler

Next we need to implement a OperationHandler for our new Apply Action. This class handles the incomming action and injects a GModelState to update the corresponding Model element:

````java
public class ApplyEventUpdateOperation extends AbstractOperationHandler<ApplyEventUpdateOperation> {

    @Inject
    protected ActionDispatcher actionDispatcher;

    @Inject
    protected GModelState modelState;

    @Override
    protected void executeOperation(final ApplyEventUpdateOperation operation) {
        // fetch the corresponding model element
        Optional<EventNode> element = modelState.getIndex().findElementByClass(operation.getId(), EventNode.class);
        if (element.isEmpty()) {
            throw new RuntimeException("Cannot find element with id '" + operation.getId() + "'");
        }
        String expression = operation.getExpression();
        // extract the property and the value
        ....
        // is it an update of the property 'documentation' ?
        if (expression.startsWith("documentation:")) {
	        // Update the model element
	        element.get().setDocumentation(value);
        }
    }

}
````

The OperationHandler extracts the property name and the value from the expression. If you send more complex data like XML you have to parse the data here. 

The Operation Handler also fetches the corresponding model element and updates the property. This will automatically inform the client about a new model state.

### Binding the Operation Handlers

Finally we need to bind the handler to your DiagramModule in the method configureOperationHandlers

````java
public class BPMNDiagramModule extends GModelJsonDiagramModule {
    ....
    @Override
    protected void configureOperationHandlers(final MultiBinding<OperationHandler> binding) {
        super.configureOperationHandlers(binding);

		.....

        // bind Apply Operation Hander (send from the client)
        binding.add(ApplyEventUpdateOperationHandler.class);
    }
}
````

That's it. Now you can send property updates from your client to the server.

## The Client Side

On the GLSP Client side we can now fire a corresponding applyEventUpdate Action each time a property of our Event Element has changed:

````javascript
@injectable()
export class MyUIExtension extends AbstractUIExtension implements EditModeListener, SelectionListener {

	@inject(TYPES.IActionDispatcher)
	protected readonly actionDispatcher: ActionDispatcher;

	....
	// Update the property 'documentation' for the selected element ID
	const action = new ApplyEventUpdateOperation(this.selectedElementId, 'documentation:' + _newValue);
	this.actionDispatcher.dispatch(action);

}

export class ApplyEventUpdateOperation implements Action {
	static readonly KIND = 'applyEventUpdate';
	readonly kind = ApplyEventUpdateOperation.KIND;
	constructor(readonly id: string, readonly expression: string) { }
}


````

That's it. The changes send by your GLSP Client will be processed by the server which is updating the GModel directly and finally sends the updated GModel back to the client.


