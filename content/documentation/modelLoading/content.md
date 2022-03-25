+++
fragment = "content"
weight = 100

title = "Source Model & Model State"

[sidebar]
  sticky = true
+++

In order to render a diagram, the GLSP client first requests the graphical model from the server by sending a [RequestModelAction](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md#241-requestmodelaction).
With such a request model action, the client can provide a set of custom arguments in the client options, such as a URI or an ID of a diagram, to specify the diagram requested by the user.
Based on these arguments, the GLSP server determines which source model it needs to load, transform into a graphical model and send back to the client.

Therefore, a request model action is handled as follows by the the _GLSP server_ (see also `RequestModelActionHandler`):

1. Invoke the configured `SourceModelStorage` to load the source model from an arbitrary resource into the model state.
2. Invoke the configured `GModelFactory` to transform the model state into the graphical model (GModel; see [GModelFactory]({{< ref "modelGeneration" >}})) that describes the diagram to be rendered.
3. Invoke the model submission to send the graphical model to the client.

In this section, we’ll look at the interfaces involved step 1: the `SourceModelStorage` and the `GModelState`.

Every GLSP server needs to provide an implementation of the `SourceModelStorage`.
Implementations of the `SourceModelStorage` interface are reponsible for loading source models from a specific resource, such as an EMF model, a JSON file, or a database, into the GLSP server's model state.

The `ModelState` is the central stateful object within a client session that represents the information about the current state of the original source model.
All other services and handlers may access the model state to obtain the required information about the model in order to perform their diagram editing tasks.

As you typically need to store custom information that is specific to your model in the model state, you typically also bind a custom implementation that represents your model state.
This implementation usually implements the interface `GModelState` (and extends the base class `DefaultGModelState`).

<details open><summary>Java GLSP Server</summary>

```java
protected Class<? extends GModelState> bindGModelState() {
  return MyModelState.class;
}
```

</details>

<details><summary>Node GLSP Server</summary>

```ts
protected configure(bind: interfaces.Bind, unbind: interfaces.Unbind,
isBound: interfaces.IsBound, rebind: interfaces.Rebind): void {
    super.configure(bind, unbind, isBound, rebind);
    bind(MyModelState).toSelf().inSingletonScope();
    bind(ModelState).toService(MyModelState);
}
```

</details>
</br>

Now as we have registered our model state implementation, we can look at the source model loading.
First, we need to make the GLSP server aware of your `SourceModelStorage` implementation, so you have to bind your implementation of the `ModelSourceLoader` interface in the server’s DI module:

<details open><summary>Java GLSP Server</summary>

```java
  @Override
  protected Class<? extends SourceModelStorage> bindSourceModelStorage() {
    return MySourceModelStorage.class;
  }
```

</details>

<details><summary>Node GLSP Server</summary>

```ts
  protected configure(bind: interfaces.Bind, unbind: interfaces.Unbind,
   isBound: interfaces.IsBound, rebind: interfaces.Rebind): void {
      super.configure(bind, unbind, isBound, rebind);
      bind(SourceModelStorage).to(MySourceModelStorage);
  }
```

</details>
</br>

The registered implementation of the source model storage needs to provide two functionalities:

1. Loading source models, based on the parameters that are contained in the [RequestModelAction](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md#241-requestmodelaction), and adding them into the session’s model state.
The implementation mostly depends on where you need to load your source model(s) from and what kind of model(s) you are dealing with (files, XML, JSON, EMF, a database, etc.).
2. Saving the current version the source model of the `GModelState` back into its original resource (files, XML, EMF, database, etc.). This method is invoked when the client sends a [SaveModelAction](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md#251-savemodelaction) (see also `SaveModelActionHandler`).

<details open><summary>Java GLSP Server</summary>

```java
public class MySourceModelStorage implements SourceModelStorage {

   @Inject
   protected MyModelState modelState;

   @Override
   public void loadSourceModel(final RequestModelAction action) {
      final String uri = MapUtil.getValue(action.options, "sourceURI");
      // load your model from file, EMF, database, etc.
      final YourModel model = …;
      // add information needed about your model into the model state
      modelState.setModel(model);
   }

   @Override
   public void saveSourceModel(final SaveModelAction action) {
      // get the current version of your model
      final YourModel model = modelState.getModel();
      // get the information to know where to store your model
      final String uri = action.getFileUri().get();
      
      try {
         // store your model
      } catch (IOException e) {
         LOG.error(e);
         throw new GLSPServerException("An error occured while saving the model.", e);
      }
   }

}
```

</details>

<details opn><summary>Node GLSP Server</summary>

```ts
@injectable()
export class MySourceModelStorage implements SourceModelStorage {

    @inject(MyModelState)
    protected modelState: MyModelState;

    loadSourceModel(action: RequestModelAction): MaybePromise<void> {
      const uri = action.options!["sourceURI"];
      // load your model from file, EMF, database, etc.
      const model = …;
      // add information needed about your model into the model state
      this.modelState.model=model;
    }

   saveSourceModel(action: SaveModelAction): MaybePromise<void> {
      // get the current version of your model
      const model = this.modelState.model;
      // get the information to know where to store your model
      const uri = this.modelState.sourceUri;
      
      try {
         // store your model
      } catch (error) {
         throw new GLSPServerError(`Could not load model from file: ${this.modelState.sourceUri}`, error);
      }
   }
}
```

</details>
</br>

In the GLSP Workflow example there is also an example, in which the source model is a JSON file that contains the GModel directly.
In such a scenario, you can use the plain GModelState and the JsonFileGModelLoader.

Once the source model has been loaded into the model state, the GLSP server invokes the configured `GModelFactory` to derive the graphical model from the source model and issues model update for the client. (see [Graphical Model Generation]({{< ref "modelGeneration" >}}))
