+++
fragment = "content"
weight = 100

title = "Client-side layouting"

[sidebar]
  sticky = true
+++

We distinguish between micro and macro layout.
Usually, the server is responsible for the macro layout, that is the arrangement of the main model elements (i.e. nodes and edges).
This layout is defined already in the graphical model by means of coordinates.
In turn, the client is responsible for the micro layout, that is the positioning and size computation of elements within a container element such as nodes.
The client side (i.e. micro) layout can be configured in the graphical model, but will be applied on the client during the rendering phase.
</br>

## Layout Container

Graphical elements that support client-side layouting of contained elements offer a `layout` property which defines the type of layouter that should be used. 
In addition, the behavior of the layouter can be configured with `layout options`.

For an example let’s have a look at the following `GNode`:

<details open><summary> Java GLSP Server</summary>

```java
new GNodeBuilder()
   .layout(“vbox”)
   .layoutOptions(new GLayoutOptions()
      .hAlign(“center”))
   .add(new GLabelBuilder()
      .text("label1")
      .build())
   .add(new GLabelBuilder()
      .text("label2")
      .build())
   .build();
```

</details>
<details ><summary> Node GLSP Server</summary>

```ts
GNode.builder()
    .layout('vbox')
    .addLayoutOption('hAlign', 'center')
    .add(new GLabelBuilder(GLabel).text('label1').build())
    .build();
```

</details>
</br>

This node contains two label elements that should be layouted.
There are four built-in layout types that can be used: “hbox”,”vbox”,”stack” and “freeform”.
In the example node we define that the labels should be layouted with the vbox layouter.
In addition, we specify options for that layouter by defining that the children should be centered horizontally.

Additional layouters can be contributed by creating a layouter class that implements the `ILayout` interface.

<details open><summary>GLSP Client</summary>

```ts
export interface ILayout {
    layout(container: SParentElement & LayoutContainer,
           layouter: StatefulLayouter): void
}
```

</details>

Then the new custom layout needs to bound in the diagram module ("di.config.ts"):

<details open><summary>GLSP Client</summary>

```ts
const myDiagramMdoule = new ContainerModule((bind, unbind, isBound, rebind) => {
    bind(TYPES.Layouter).to(MyCustomLayouter);
}
```


</details>
</br>

## Edge Layout

Graphical elements that are typically rendered in combination with an edge such as labels have an `edgeLayout` property.
This can be used to describe how the element should be aligned with the edge.

For example let's have a look at the following `GLabel`:

<details open><summary> Java GLSP Server</summary>

```java
new GLabelBuilder() //
   .edgePlacement(new GEdgePlacementBuilder()//
      .side(“top”)//
      .position(0.5)//
      .build())//
   .add(new GLabelBuilder().text("MyLabel").build())
   .build();
```

</details>
<details ><summary> Node GLSP Server</summary>

```ts
GLabel.builder()
    .edgePlacement({ side: 'top', 
                    position: 0.5,
                    rotate: false,
                    offset: 0 })
    .add(new GLabelBuilder(GLabel).text('MyLabel').build())
    .build();
```


</details>
</br>

The label above specifies the property `edgePlacement`.
If this label is added as a child of an edge, it will be placed above the edge.
The position is defined with 0.5 (i.e. 50 percent of the edge length) which means the label should be placed in the center of the edge.
After the edge routes have been rendered the client will query all active edge placements and adjust the label position accordingly.
