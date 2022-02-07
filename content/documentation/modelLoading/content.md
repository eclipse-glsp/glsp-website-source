+++
fragment = "content"
weight = 100

title = "Model Source Loading & Model State"

[sidebar]
  sticky = true
+++

To be able to render the diagram, the GLSP client first requests the graphical model from the server.
The model loading process is initiated with a [RequestModelAction](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md#241-requestmodelaction).
Typically this is the first action that the GLSP client sends after initializing the diagram widget.

With the [RequestModelAction](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md#241-requestmodelaction), the client can provide a set of custom arguments (client options).
At the bare minimum these client options have to provide the required information so that the _GLSP server_ can identify and load the correct model source.
In most cases this will be, for instance, a URI of the model to be loaded.

To handle such a request model action, the _GLSP server_ will perform the following steps:

1. Invoke the configured `ModelSourceLoader` to load the model from an arbitrary model source into the model state.
2. Invoke the configured `GModelFactory` to transform the model state into the graphical model (GModel; see [GModelFactory]({{< ref "modelGeneration" >}})) that describes the diagram to be rendered.
3. Invoke the model submission to send the graphical model to the client.

In this section, we’ll look at step 1, in which the configured `ModelSourceLoader` implementation is invoked to load the source model from the arbitrary model source into the model state.

The `ModelState` is the central stateful object within a client session that represents the information about the current state of the original source model.
All other services and handlers may access the model state to obtain the required information about the model in order to perform their respective tasks.

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

Now as we have registered our model state implementation, we can look at the model source loading.
First, we need to make the GLSP server aware of your model source loader, you have to register your implementation of the `ModelSourceLoader` interface in the server’s DI module:

<details open><summary>Java GLSP Server</summary>

```java
  @Override
  protected Class<? extends ModelSourceLoader> bindSourceModelLoader() {
    return MyModelLoader.class;
  }
```

</details>

<details><summary>Node GLSP Server</summary>

```ts
  protected configure(bind: interfaces.Bind, unbind: interfaces.Unbind,
   isBound: interfaces.IsBound, rebind: interfaces.Rebind): void {
      super.configure(bind, unbind, isBound, rebind);
      bind(ModelSourceLoader).to(MyModelLoader);
  }
```

</details>
</br>

The registered implementation of the model loader needs to load the source model and add it to the session’s model state, based on the parameters that are contained in the [RequestModelAction](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md#241-requestmodelaction).
Consequently, the implementation mostly depends on where you need to load your source model(s) from and what kind of model(s) you are dealing with (files, XML, JSON, EMF, database, etc.).

<details open><summary>Java GLSP Server</summary>

```java
public class MyModelLoader implements ModelSourceLoader {

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

}
```

</details>

<details opn><summary>Node GLSP Server</summary>

```ts
@injectable()
export class MyModelLoader implements ModelSourceLoader {

    @inject(MyModelState)
    protected modelState: MyModelState;

    loadSourceModel(action: RequestModelAction): MaybePromise<void> {
      const uri = action.options!["sourceURI"];
      // load your model from file, EMF, database, etc.
      const model = …;
      // add information needed about your model into the model state
      this.modelState.model=model;
    }
}
```

</details>
</br>

In the GLSP Workflow example there is also an example, in which the source model is a JSON file that contains the GModel directly.
In such a scenario, you can use the plain GModelState and the JsonFileGModelLoader.

Once the source model has been loaded into the model state, the GLSP server invokes the configured `GModelFactory` to derive the graphical model from the source model and issues model update for the client. (see [Graphical Model Generation]({{< ref "modelGeneration" >}}))
