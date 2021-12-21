+++
fragment = "content"
weight = 100

title = "Extending the Graphical Model"

[sidebar]
  sticky = true
+++

GLSP provides a set of default graphical model element classes that can be used to construct the graphical model and already cover a large set of use cases.
For advanced use cases the existing base model elements can be customized or additional elements can be introduced.
As an example, let’s have a look at the custom WeightedEdge element introduced by the [GLSP Workflow example](https://github.com/eclipse-glsp/glsp-examples).

## GLSP Client
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

## Node GLSP Server

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

## Java GLSP Server

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

## Generic Args

Every graphical model element type has a generic “args” property, which can be used to store additional properties as key-value pairs.
These arguments can be used as a more lightweight alternative to extending the graphical model classes, especially if only simple extensions are needed.
