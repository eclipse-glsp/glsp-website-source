+++
fragment = "content"
weight = 100

title = "Client-side Layouting"

[sidebar]
  sticky = true
+++

We distinguish between micro and macro layout.
Usually, the server is responsible for the macro layout, that is the arrangement of the main model elements (i.e. nodes and edges).
This layout is defined already in the graphical model by means of coordinates.
In turn, the client is responsible for the micro layout, that is the positioning and size computation of elements within a container element such as nodes.
The client side (i.e. micro) layout can be configured in the graphical model, but will be applied on the client during the rendering phase.
</br>

### Layout Container

Graphical elements that support client-side layouting of contained elements offer a `layout` property which defines the type of layouter that should be used.
In addition, the behavior of the layouter can be configured with a set of `layout options`.

For an example let’s have a look at the following `GNode`:

<details open><summary> Java GLSP Server</summary>

```java
new GNodeBuilder()
   .layout(GConstants.Layout.VBOX)
   .layoutOptions(new GLayoutOptions()
      .hAlign(GConstants.HAlign.Center))
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
  .layout("vbox")
  .addLayoutOption("hAlign", "center")
  .add(GLabel.builder().text("label1").build())
  .add(GLabel.builder().text("label2").build())
  .build();
```

</details>
</br>

This node contains two label elements that should be layouted vertically (from top to bottom) using the `vbox` layouter.
In addition, we specify options for that layouter by defining that the children should be centered horizontally.

#### Default Layouters

In general, layouters can be applied to elements that are compartments, in order to layout the containers based on the sizes of their children.
There are four built-in layout types that can be used: `hbox`, `vbox`, `freeform` and `stack`.

##### `hbox` Layout

The [`HBoxLayouterExt`](https://github.com/eclipse-glsp/glsp-client/blob/master/packages/client/src/features/layout/hbox-layout.ts) layouts children of a container in a horizontal (left to right) direction.
This layouter provides additional layout options (i.e. preferred sizes and grab options) via `HBoxLayoutOptionsExt` which defaults values can be overridden.

- The layout options `prefWidth` and `prefHeight` define the preferred size of the container element (for example if all children would be deleted, this size is applied to the container element).
- The special layout options `hGrab` and `vGrab` (boolean flags) indicate, if the remaining size (horizontally or vertically) can be grabbed by children.

<details open><summary>Example Usage Java GLSP Server</summary>

This example creates a compartment of the default type `DefaultTypes.COMPARTMENT` using the `hbox` layout.
It adds two children, one icon compartment (with custom type `"icon"`) and one label.
The layout options define a horizontal gap of `3` between the children of the compartment.

```java
   new GCompartmentBuilder()
         .type(DefaultTypes.COMPARTMENT)
         .layout(GConstants.Layout.HBOX)
         .layoutOptions(new GLayoutOptions().hGap(3))
         .add(new GCompartmentBuilder()
            .type("icon")
            .build())
         .add(new GLabelBuilder()
            .text("label")
            .build())
         .build();
   .build();
```

</details>

<details ><summary> Node GLSP Server</summary>

```ts
GCompartment.builder()
  .builder()
  .layout("hbox")
  .addLayoutOption("hGap", 3)
  .add(GCompartment.builder().type("icon").build())
  .add(GLabel.builder().text("label").build())
  .build();
```

</details>

According to the layout options, the children of the container are layouted and as concluding step, the final bounds of the container are computed based on the maximum bounds of its children.

##### `vbox` Layout</br>

The [`VBoxLayouterExt`](https://github.com/eclipse-glsp/glsp-client/blob/master/packages/client/src/features/layout/vbox-layout.ts) layouts children of a container in a vertical (top to bottom) direction.
This layouter provides additional layout options via `VBoxLayoutOptionsExt` which defaults values can be overridden.
These additional layout options are the same as for the `hbox` layout.

According to the layout options, the children of the container are layouted and as concluding step, the final bounds of the container are computed based on the maximum bounds of its children.

<details open><summary>Example Usage Java GLSP Server</summary>

This example creates a compartment of the default type `DefaultTypes.COMPARTMENT` using the `vbox` layout.
It adds two children labels, which are aligned left horizontally.

```java
   new GCompartmentBuilder()
         .type(DefaultTypes.COMPARTMENT)
         .layout(GConstants.Layout.VBOX)
         .layoutOptions(new GLayoutOptions().hAlign(GConstants.HAlign.LEFT))
         .add(new GLabelBuilder()
            .text("First child label")
            .build())
         .add(new GLabelBuilder()
            .text("Second child label")
            .build())
         .build();
   .build();
```

</details>

<details ><summary> Node GLSP Server</summary>

```ts
GCompartment.builder()
  .builder()
  .layout("vbox")
  .addLayoutOption("hAlign", "left")
  .add(GLabel.builder().text("First child label").build())
  .add(GLabel.builder().text("Second child label").build())
  .build();
```

</details>

##### `freeform` Layout</br>

The [`FreeFormLayouter`](https://github.com/eclipse-glsp/glsp-client/blob/master/packages/client/src/features/layout/freeform-layout.ts) layouts children of a container according to their explicit `x/y` positions inside the parent container (relative position).
This layouter uses the default `AbstractLayoutOptions` and provides default values (e.g. `0` padding), which of course can also be overridden.

Again here, the children of the container are layouted according to their explicit positions inside the container and as concluding step, the final bounds of the container are computed based on the maximum bounds of its children.

<details open><summary>Example Usage Java GLSP Server</summary>

This example creates a compartment of the custom type `"comp:structure"` using the `freeform` layout.
It adds one child node, at the relative position `(5, 5)` of the parent.

```java
   GDimension containerPrefSize = GraphUtil.dimension(20 /*width*/, 10 /*height*/);
   GDimension childSize = GraphUtil.dimension(22 /*width*/, 7 /*height*/);
   GPoint childPosition = GraphUtil.point(5 /*x*/, 5 /*y*/);

   Map<String, Object> layoutOptions = new HashMap<>();
   layoutOptions.put(H_GRAB, true);
   layoutOptions.put(V_GRAB, true);
   layoutOptions.put(GLayoutOptions.KEY_PREF_WIDTH, containerPrefSize.getWidth());
   layoutOptions.put(GLayoutOptions.KEY_PREF_HEIGHT, containerPrefSize.getHeight());
   new GCompartmentBuilder()
      .type("comp:structure")
      .layout(GConstants.Layout.FREEFORM)
      .layoutOptions(layoutOptions)
      .add(
         new GNodeBuilder(DefaultTypes.NODE)
            .position(childPosition)
            .size(childSize)
            .build()
      )
      .build();
```

</details>

<details ><summary> Node GLSP Server</summary>

```ts
const containerPrefSize = { width: 20, height: 10 };
const childSize = { width: 22, height: 7 };
const childPosition = { x: 5, y: 5 };

const layoutOptions = {
  ["hGrab"]: true,
  ["vGrab"]: true,
  ["prefWidth"]: containerPrefSize.width,
  ["prefHeight"]: containerPrefSize.height,
};
return GCompartment.builder()
  .type("comp:structure")
  .layout("freeform")
  .addLayoutOptions(layoutOptions)
  .add(
    GNode.builder().type("node").position(childPosition).size(childSize).build()
  )
  .build();
```

</details>

##### `stack` Layout</br>

The [`StackLayouter`](https://github.com/eclipse/sprotty/blob/master/packages/sprotty/src/features/bounds/stack-layout.ts) layouts children of a container according to their absolute position.
This layouter provides additional layout options (i.e. padding factor, horizontal and vertical alignment) via `StackLayoutOptions` which defaults values can be overridden.

The children of the container are layouted according to their explicit positions. Based on that, the maximum height and width are computed and are used as bounds for the container.

##### Common Layout Options

- Vertical alignment `GConstants.VAlign`: `TOP` | `CENTER` | `BOTTOM`
- Horizontal alignment `GConstants.HAlign`: `LEFT` | `CENTER` | `RIGHT`
- Preferred height / width `GLayoutOptions.KEY_PREF_WIDTH` | `GLayoutOptions.KEY_PREF_HEIGHT`

For more details, please see [`GLayoutOptions`](https://github.com/eclipse-glsp/glsp-server/blob/master/plugins/org.eclipse.glsp.graph/src/org/eclipse/glsp/graph/builder/impl/GLayoutOptions.java)
and [`GConstants`](https://github.com/eclipse-glsp/glsp-server/blob/master/plugins/org.eclipse.glsp.graph/src/org/eclipse/glsp/graph/util/GConstants.java).

#### Custom Layouter

Additional custom layouters can be contributed by creating a layouter class that extends the `AbstractLayouter` and optionally provide custom options that extend the `AbstractLayoutOptions`.

In the following example we show a simple custom layouter that extends the `VBoxLayouterExt` and provides a custom layout option,
which increases all children sizes by this value in both width and height, the default value is `50px`.

<details open><summary>GLSP Client</summary>

```ts
export interface MyCustomLayoutOptions extends VBoxLayoutOptionsExt {
  enlargeSizeBy: number;
}

@injectable()
export class MyCustomLayouter extends VBoxLayouterExt {
  static override KIND = "myCustomLayout";

  protected override getChildrenSize(
    container: SParentElement & LayoutContainer,
    containerOptions: MyCustomLayoutOptions,
    layouter: StatefulLayouter
  ): Dimension {
    const result = super.getChildrenSize(container, containerOptions, layouter);
    return {
      width: result.width + containerOptions.enlargeSizeBy,
      height: result.height + containerOptions.enlargeSizeBy,
    };
  }

  protected override getDefaultLayoutOptions(): MyCustomLayoutOptions {
    return {
      enlargeSizeBy: 50,
      ...super.getDefaultLayoutOptions(),
    };
  }

  protected override spread(
    a: MyCustomLayoutOptions,
    b: MyCustomLayoutOptions
  ): MyCustomLayoutOptions {
    return { ...a, ...b };
  }
}
```

</details>

Then the new custom layout needs to bound in the diagram module (`di.config.ts`):

<details open><summary>GLSP Client</summary>

```ts
const myDiagramModule = new ContainerModule((bind, unbind, isBound, rebind) => {
   rebind(VBoxLayouter).to(MyCustomLayouter);
}
```

</details>

For independent implementations that extend the `AbstractLayout`, it will be bound as follows:

<details open><summary>GLSP Client</summary>

```ts
const myDiagramModule = new ContainerModule((bind, unbind, isBound, rebind) => {
   configureLayout({ bind, isBound }, MyCustomLayouter.KIND, MyCustomLayouter);
}
```

</details>

Then the custom layouter is ready to use:

<details open><summary>Example Usage Java GLSP Server</summary>

```java
   new GCompartmentBuilder()
         .type(DefaultTypes.COMPARTMENT)
         .layout("myCustomLayout")
         .layoutOptions(Map.of("enlargeSizeBy", 15))
         .add(new GLabelBuilder()
            .text("label")
            .build())
         .build();
   .build();
```

</details>

<details ><summary> Node GLSP Server</summary>

```ts
GCompartment.builder()
  .builder()
  .layout("myCustomLayout")
  .addLayoutOption("enlargeSizeBy", 15)
  .add(GLabel.builder().text("label").build())
  .build();
```

</details>

</details>
</br>

### Edge Layout

Graphical elements that are typically rendered in combination with an edge such as labels have an `edgeLayout` property.
This can be used to describe how the element should be aligned with the edge.

For example let's have a look at the following `GLabel`:

<details open><summary> Java GLSP Server</summary>

```java
new GLabelBuilder()
   .edgePlacement(new GEdgePlacementBuilder()
      .side("top")
      .position(0.5)
      .build())
   .add(new GLabelBuilder().text("MyLabel").build())
   .build();
```

</details>
<details ><summary> Node GLSP Server</summary>

```ts
GLabel.builder()
  .edgePlacement({ side: "top", position: 0.5, rotate: false, offset: 0 })
  .add(new GLabelBuilder(GLabel).text("MyLabel").build())
  .build();
```

</details>
</br>

The label above specifies the property `edgePlacement`.
If this label is added as a child of an edge, it will be placed above the edge.
The position is defined with 0.5 (i.e. 50 percent of the edge length) which means the label should be placed in the center of the edge.
After the edge routes have been rendered the client will query all active edge placements and adjust the label position accordingly.
