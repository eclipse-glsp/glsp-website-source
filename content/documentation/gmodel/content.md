+++
fragment = "content"
weight = 100

title = "Graphical Model"

[sidebar]
  sticky = true

+++

The graphical model is a serializable description of the diagram to be visualized on the client.
It is the central communication artifact between client and server.
The server creates the graphical model from an [arbitrary source model]({{< ref "sourceModel" >}}) by invoking a so-called [`GModelFactory`](#Graphical-Model-Factory) and sends the graphical model to the client.
Thus, the client doesn't need to know from which source model it has been generated and how to manipulate the source model.
However, the client interprets the graphical model in order to [render a visualization of the diagram]({{< ref "rendering" >}}) described in the graphical model.

### Graphical Model Structure

The graphical model is composed of elements and edges.
Each element or edge has a unique identifier and a type.
The graphical model elements are organized in a tree, as defined by the parent-child relationship between elements, with a single root element.
The graphical model library consists of several common base classes, such as nodes, ports, labels, compartments, but can be extended with additional properties or even new types, if needed.

The graphical model is typically composed of the following elements.

- **GModelRoot**: Each graphical model must have exactly one root
  - **GShapeElement**: A graphical element is represented by a shape with visual bounds (position and size).
  Note that such elements can be nested based on their parent-child relationship.
  There are a the following concrete sub-types of shapes:
    - **GNode**: Representation of a logical diagram node
    - **GPort**: Ports are typically children of nodes and serve as connection points for edges
    - **GLabel**: Representation of a text label
    - **GCompartment**: A generic container element used for element grouping
  - **GEdge**: A diagram edge that connects a source element and a target element (typically nodes or ports).

#### SModel: Graphical model on the client

The default GLSP client uses [Sprotty](https://github.com/eclipse/sprotty), an SVG-based diagramming framework, to render diagrams.
Sprotty uses a model to represent a diagram too -- the so-called `SModel`.
The graphical model of GLSP is based on the SModel and, thus, can be seen as a compatible extension of the Sprotty model.

As a naming convention, GLSP uses the S-prefix for model elements on the client to conform to the naming of the Sprotty model.
Thus, a node of the graphical model on the GLSP client is called `SNode`, whereas it is called `GNode` on the server, the same is true for `SEdge` / `GEdge`, etc.
Semantically, those elements, however, are equivalent and are exchanged transparently between the server and the client via JSON-RPC.

#### GModel: Graphical model on the server

The [Java-based GLSP server](https://github.com/eclipse-glsp/glsp-server) uses the [EMF](https://www.eclipse.org/modeling/emf) to represent and manage the graphical model internally.
Note that this is just an internal way of representing the `GModel` at runtime but doesn’t mean that adopters need to represent their original source models with EMF too.
The GLSP server uses EMF in order to reuse its model management and editing capabilities, its command stack and command-based editing.
Therefore, the graphical model is described as an Ecore model and the corresponding Java classes are automatically generated from this model.
Using [GSON](https://github.com/google/gson), the GModel is then serialized and deserialized to JSON before it is sent via JSON-RPC to the client.

The [node-based GLSP server](https://github.com/eclipse-glsp/glsp-server-node) provides a graph model library, which defines the graph model types, such as `GNode`, `GEdge`, etc. alongside a builder API to make creating instances more convenient.
However, as the node-based GLSP server and the GLSP client are both based on ES6, this graph library is based on graph model definitions that are used on the client.

### Graphical Model Factory

After the initial loading of the source model – and also after each change of the source model – the GLSP server generates a graphical model from the source model in order to define what is to be rendered on the client.

The generation of the graphical model from the original source model is the responsibility of the `GModelFactory`.
Therefore, the `GModelFactory` obtains the source model from the model state and generates a new graphical model from it.
Implementations of the `GModelFactory` are by nature very specific to the source model and the diagram type.
Thus, in almost every GLSP editor project, a custom `GModelFactory` implementation is provided.

The only exception are GLSP editors that directly operate on GModels; that is, the GModel is persisted and loaded directly by the registered implementation of the source model storage.
In such cases, no transformation from the source model to the GModel needs to be provided as the source model already is the GModel.
Thus, the so-called `NullImpl` of the `GModelFactory` can be used.
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

For the sake of an example, let’s assume that the source model is a simple list of named entities.
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

In the `createGModel()` method the entities are retrieved from the model state, as they have been added there by the source model storage (see [Source Model Storage]({{< ref "sourceModel" >}})).
Then a new GNode is created for each entity.
Finally all new nodes are added as children of a newly created GGraph and the graphical root element in the model state is updated.

Note that we have used the `GModelBuilder` API in this example to construct new graphical elements.
This builder API offers a convenient way to construct new graphical model elements in a concise and fluent fashion.
It is the preferred method and should be used over plain constructor creation.

### Extending the Graphical Model

GLSP provides a set of default graphical model element classes that can be used to construct the graphical model and already cover a large set of use cases.
For advanced use cases the existing base model elements can be customized or additional elements can be introduced.
As an example, let’s have a look at the custom WeightedEdge element introduced by the [GLSP Workflow example](https://github.com/eclipse-glsp/glsp-examples).

#### GLSP Client

A WeightedEdge is a special edge that has an optional “probability” property.
We can define such an element by simply subclassing the `SEdge` class:

```ts
export class WeightedEdge extends SEdge {
   probability?: string;
}
```

And then the new WeightedEdge type has to be configured in the diagram module (`di.config.ts`).

```ts
const workflowDiagramModule = new ContainerModule((bind, unbind, isBound, rebind) => {
    ...
    configureModelElement(context, 'edge:weighted', WeightedEdge, WorkflowEdgeView);
    ...
}
```

</br>

#### Node GLSP Server

For the node _GLSP server_ the new `WeightedEdge` type can be declared similar to the _GLSP client_ by subclassing the `GEdge` class.

```ts
export class WeightedEdge extends GEdge {
    probability?: string;
}
```

To use the builder API for `WeightedEdge` creation we also have to implement a `WeightedEdgeBuilder` that extends the default `GEdgeBuilder`.

```ts
export class WeightedEdgeBuilder<E extends WeightedEdge = WeightedEdge> extends GEdgeBuilder<E> {
    probability(probability: string): this {
        this.proxy.probability = probability;
        return this;
    }
}
```

</br>

#### Java GLSP Server

When using the _Java GLSP server_, a new Ecore model that extends the default "graph.ecore" model has to be created to declare new model elements.
For more details, please have a look at the "workflow-graph.ecore" model in the [GLSP Workflow example](https://github.com/eclipse-glsp/glsp-examples). 
Once the `WeightedEdge` is specified in the Ecore model, the corresponding source code has to be generated.
Now the `GraphExtension` API can be used to configure the "workflow-graph.ecore" for the workflow diagram language.
A class that implements the the corresponding interface has to created:

```java
public class WFGraphExtension implements GraphExtension {

   @Override
   public EPackage getEPackage() { return WfgraphPackage.eINSTANCE; }

   @Override
   public EFactory getEFactory() { return WfgraphFactory.eINSTANCE; }

}
```

And then configured in the `WorkflowDiagramModule`:

```java
@Override
protected Class<? extends GraphExtension> bindGraphExtension() {
    return WFGraphExtension.class;
}
```

To use the builder API for `WeightedEdge` creation we also have to implement a `WeightedEdgeBuilder` that extends the default `AbstractGEdgeBuilder`.
</br></br>

#### Generic Args

Every graphical model element type has a generic “args” property, which can be used to store additional properties as key-value pairs.
These arguments can be used as a more lightweight alternative to extending the graphical model classes, especially if only simple extensions are needed.

---

➡️ Let's look at how the graphical model is [rendered on the client]({{< ref "rendering" >}}) next!
