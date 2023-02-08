+++
fragment = "content"
weight = 100

title = "Source Model & Model State"

[sidebar]
  sticky = true
+++

### Source Model

The source model represents the actual data that is represented in the diagram and that is modified when the user applies changes in the diagram.
Typical source model formats are [EMF](https://www.eclipse.org/modeling/emf) models, JSON files, and databases, etc.
However, GLSP and the GLSP server frameworks don't put any restrictions on what the format of this source model is.
This is achieved by putting developers of GLSP diagram servers in charge of defining how to load a source model, how to transform it into a [graphical model]({{< ref "gmodel" >}}), which is the description of the diagram to be rendered, and how to manipulate the source model, if a user edits a diagram.

Therefore, a GLSP server needs to provide the following implementations:

1. [Source model storage](#source-model-storage-and-model-state-and), which defines how to load and store source models.
2. [Graphical model factory]({{< ref "gmodel" >}}), which defines how the source model is transformed into a graphical model.
3. [Edit operation handlers]({{< ref "modelOperations" >}}) that manipulate the source model, based on user actions performed in the diagram.

#### Loading Source Models and Showing a Diagram

To load a source model and show a diagram, the following steps are performed:

1. The client sends a [RequestModelAction](/documentation/protocol/#241-requestmodelaction) with a URI or other arguments for identifying a source model to the server
2. The server invokes the [source model storage](#source-model-storage-and-model-state-and) to load the source model identified by the arguments sent by the client
3. The server invokes the [graphical model factory]({{< ref "gmodel" >}}) to translate the source model into the graphical model
4. The server sends the created graphical model to the client
5. The client renders graphical model

#### Processing Edit Operations

When a user performs an edit operation in the diagram:

1. The client sends an operation request to the server
2. The server invokes the registered [edit operation handler]({{< ref "modelOperations" >}}), which modifies the underlying source model directly
3. The server applies the [graphical model factory]({{< ref "gmodel" >}}) to the modified source model to create a new graphical model
4. The server sends the created graphical model to the client
5. The client re-renders the diagram according to the new version of the graphical model

As can be seen in the steps above for loading and editing source models, both processes share many steps and are based on the same three custom implementations for particular source models.
Thus, by providing these three implementations, any source model format can be supported with GLSP.

Please note that GLSP provides [generic base implementations]({{< ref "integrations" >}}) for typical source model types.

### Source Model Storage and Model State

Every GLSP server needs to provide an implementation of the interface `SourceModelStorage`.
Implementations of this interface are responsible for loading source models from a specific resource, such as an EMF model, a JSON file, or a database, into the GLSP server's model state.

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
protected override bindModelState(): BindingTarget<ModelState> {
   return MyModelState;
}
```

</details>
</br>

Now as we have registered our model state implementation, we can look at the source model loading.
First, we need to make the GLSP server aware of your `SourceModelStorage` implementation, so you have to bind your implementation of the `SourceModelStorage` interface in the server’s DI module:

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
protected bindSourceModelStorage(): BindingTarget<SourceModelStorage> {
   return MySourceModelStorage;
}
```

</details>
</br>

The registered implementation of the source model storage needs to provide two functionalities:

1. Loading source models, based on the parameters that are contained in the [RequestModelAction](/documentation/protocol/#241-requestmodelaction), and adding them into the session’s model state.
   The implementation mostly depends on where you need to load your source model(s) from and what kind of model(s) you are dealing with (files, XML, JSON, EMF, a database, etc.).
2. Saving the current version of the source model from the `GModelState` back into its original resource (files, XML, EMF, database, etc.). This method is invoked when the client sends a [SaveModelAction](/documentation/protocol/#251-savemodelaction).

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
         throw new GLSPServerException("An error occurred while saving the model.", e);
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

The GLSP Workflow example is an example, in which in which the source model is a JSON file that contains the GModel directly.
In such a scenario, you can use the plain GModelState and the JsonFileGModelStorage.

Once the source model has been loaded into the model state, the GLSP server invokes the configured `GModelFactory` to derive the graphical model from the source model and issues model update for the client.

---

➡️ Let's look at how the source model is translated into a [graphical model]({{< ref "gmodel" >}}) next!
