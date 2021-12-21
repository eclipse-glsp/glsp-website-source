+++
fragment = "content"
weight = 100

title = "Graphical Model Generation"

[sidebar]
  sticky = true

+++

After the initial loading of the source model – and also after each change of the original source model – the GLSP server generates a graphical model from the source model in order to define what is to be rendered on the client.


The generation of the graphical model from the original model source is the responsibility of the `GModelFactory`.
Therefore, the `GModelFactory` obtains the source model from the model state and generates a new graphical model from it.
Implementations of the `GModelFactory` are by nature very specific to the source model and the diagram type.
Thus, in almost every GLSP editor project, a custom `GModelFactory` implementation is provided.


The only exception are GLSP editors that directly operate on GModels; that is, the GModel is persisted and loaded directly by the model source loader.
In such cases, no transformation from the model source and the GModel needs to be provided as the model source already is the GModel.
Thus, the so-called NullImpl of the `GModelFactory` can be used.
An example for such a use case is provided in the GLSP Workflow example.

For all other use cases, an implementation of the `GModelFactory` needs to be provided and registered in the server DI module as follows.

<details open><summary>Java GLSP Server</summary>

```java
  @Override
   protected Class<? extends GModelFactory> bindGModelFactory() {
      return MyModelFactory.class;
   }
```

</details>

<details><summary>Node GLSP Server</summary>

```ts
    protected configure(bind: interfaces.Bind, unbind: interfaces.Unbind,
    isBound: interfaces.IsBound, rebind: interfaces.Rebind): void {
        super.configure(bind, unbind, isBound, rebind);
        bind(GModelFactory).to(MyModelFactory).inSingletonScope();
    }
```

</details>
</br>

For the sake of an example, let’s assume that the model source is a simple list of named entities.
Each entity should be visualized as a node with a label, which indicates its name.
 Then the corresponding `ModelFactory` could look as follows.

<details open><summary>Java GLSP Server</summary>

```java
public class MyModelFactory implements GModelFactory {

   @Inject
   protected MyModelState modelState;

   @Override
   public void createGModel() {

      List<Entity> entities = modelState.getModel().getEntities();

      List<GModelElement> entityNodes = entities.stream().map(entity -> //
      new GNodeBuilder("node:entity")
         .layout("vbox")
         .add(new GLabelBuilder()
            .text(entity.getName())
            .build())
         .build())
         .collect(Collectors.toList());

      GGraph newModel = new GGraphBuilder()
         .id("entity-graph")
         .addAll(entityNodes)
         .build();

      modelState.setRoot(newModel);
   }
}
```

</details>

<details><summary>Node GLSP Server</summary>

```ts
@injectable()
export class MyModelFactory implements GModelFactory {
    @inject(MyModelState)
    protected modelState: MyModelState;

    createModel(): void {
        const entities = this.modelState.getModel().getEntities();

        const entityNodes = entities.map(entity =>
            new GNodeBuilder(GNode).id('node:entity').layout('vbox')
            .add(new GLabelBuilder(GLabel)
                .text(entity.name)
                .build())
            .build()
        );

        const newModel = new GGraphBuilder(GGraph)
            .id('entity-graph')
            .addChildren(...entityNodes)
            .build();

        this.modelState.root = newModel;
    }
}
```

</details>
</br>

In the `createGModel()` method the entities are retrieved from the model state, as they have been added there by the model source loader (see  [Model Source Loader]({{< ref "modelLoading" >}})).
Then a new GNode is created for each entity.
Finally all new nodes are added as children of a newly created GGraph and the graphical root element in the model state is updated.

Note that we have used the `GModelBuilder` API in this example to construct new graphical elements.
This builder API offers a convenient way to construct new graphical model elements in a concise and fluent fashion.
It is the preferred method and should be used over plain constructor creation.
