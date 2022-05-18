+++
fragment = "content"
weight = 100

title = "Graphical Model Rendering & Styling"

[sidebar]
  sticky = true
+++

### Rendering

The input of the diagram rendering on the client is the GModel that has been generated on the server from the source model (see [Graphical Model]({{< ref "gmodel" >}})) and sent to the client via a [SetModelAction](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md#242-setmodelaction) or [UpdateModelAction](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md#243-updatemodelaction).
The client is then responsible for rendering the GModel.

In order to render the received graphical model, each graphical element type needs to be associated with a view on the client.
A view defines how a specific type of graphical element shall be transformed into a corresponding SVG representation.
The derived SVG elements are then rendered on the canvas of the diagram widget.

To define a new view, we have to create a class that implements the `IView` interface and register it for a specific type that is used in the graphical model.
As an example, let’s configure that the view named `SLabelView` is used for all elements with the type “label:custom”.
Therefore, we first need to create a dependency injection module, named `customDiagramModule` below, and configure the SLabelView for the graphical model element type “label:custom” using the `configureModelElement()` utility function:

```ts
const customDiagramModule = new ContainerModule(
  (bind, unbind, isBound, rebind) => {
    const context = { bind, unbind, isBound, rebind };
    configureModelElement(context, "label:custom", SLabel, SLabelView);
  }
);
```

The `configureModeElement()` function takes the inversify binding context, the graphical model type, its model class and its associated view as input. Under the hood this function sets up the necessary bindings so that the _GLSP client_ knows that

- Graphical model elements (received from the GLSP Server) with type ‘label:custom’ are deserialized to instances of `SLabel`
- Graphical model element with type ‘label:custom’ are rendered with the `SLabelView`

In order to be effective, we need to load the module `customDiagramModule` defined above in the diagram DI container, aka the root "di.config.ts" of your diagram implementation.
With that, every element of type “label:custom” will be rendered with the view implementation `SLabelView`.

Views themselves are typically implemented with [JSX](https://www.typescriptlang.org/docs/handbook/jsx.html), which simplifies the definition of SVG elements in Typescript. Therefore, the following generic imports are required in any module declaring a view to enable declaration of svg elements with JSX:

```ts
/** @jsx svg */
import { VNode } from "snabbdom";
import { RenderingContext, svg } from ‘@eclipse-glsp/client’;
```

In addition, make sure that the following options are set in the `tsconfig.json` file of your project:

```json
{
  "compilerOptions": {
    "jsx":"react",
    "reactNamespace":"JSX"
}
```

With that, we can implement a view as follows:

```tsx
@injectable()
export class SLabelView extends ShapeView {
  render(
    label: Readonly<SLabel>,
    context: RenderingContext
  ): VNode | undefined {
    if (!isEdgeLayoutable(label) && !this.isVisible(label, context)) {
      return undefined;
    }
    const vnode = <text class-sprotty-label={true}>{label.text}</text>;
    const subType = getSubType(label);
    if (subType) {
      setAttr(vnode, "class", subType);
    }
    return vnode;
  }
}
```

Every view has to implement the `render(`) method.
The `render(`) method takes the graphical model element as input and returns the corresponding SVG element as virtual DOM node.
The viewer queries all registered views and creates a new virtual DOM which is then used to patch the current DOM of the diagram widget.

Note that the `SLabelView` also checks whether the given element is visible and skips the SVG generation if the element is not visible in the diagram canvas.
This check is optional but it’s highly recommended to implement it in your custom views as it heavily improves the rendering performance.

The following sections give an overview of available default views in Sprotty and GLSP and how to configure them:

#### Default Sprotty Views

##### CircularNodeView

A `CircularNodeView` creates a round shape with a radius computed from the shape's size (by default it computes the radius by the minimum of the shape's width or height and divides that by 2).
The computation of the radius can be overriden and adapted to custom needs.

```ts
configureModelElement(
  context,
  DefaultTypes.NODE_CIRCLE,
  CircularNode,
  CircularNodeView
);
```

##### DiamondNodeView

A `DiamondNodeView` creates a rhombus shape based on the shape's size.

```ts
configureModelElement(
  context,
  DefaultTypes.NODE_DIAMOND,
  DiamondNode,
  DiamondNodeView
);
```

##### ExpandButtonView

The `ExpandButtonView` renders a SVG element in the shape of a triangle that allows exapandable parent elements to trigger expansion, for example to display further element information.

```ts
configureModelElement(
  context,
  DefaultTypes.BUTTON_EXPAND,
  SButton,
  ExpandButtonView
);
```

##### PreRenderedView

The `PreRenderedView` visualizes a previously rendered piece of svg code as a separate SVG element.

```ts
configureModelElement(
  context,
  DefaultTypes.PRE_RENDERED,
  PreRenderedElement | ShapedPreRenderedElement,
  PreRenderedView
);
```

##### RectangularNodeView

A `RectangularNodeView` creates a rectangular shape based shape's size.

```ts
configureModelElement(
  context,
  DefaultTypes.NODE_RECTANGLE,
  RectangularNode,
  RectangularNodeView
);
```

##### SGraphView

The `SGraphView` renders the base svg canvas for an SModel and triggers the rendering of its children.

```ts
configureModelElement(context, DefaultTypes.GRAPH, GLSPGraph, SGraphView);
```

##### SLabelView

The `SLabelView` renders a text element that contains the given label text.

```ts
configureModelElement(context, DefaultTypes.LABEL, SLabel, SLabelView);
```

##### SRoutingHandleView

A `SRoutingHandleView` renders a circle shaped element that servers as routing point for routable elements (e.g. Edges).
Its position is computed by either a register `EdgeRouterRegistry` or passed routing arguments.

```ts
configureModelElement(
  context,
  DefaultTypes.ROUTING_POINT,
  SRoutingHandle,
  SRoutingHandleView
);
```

#### Default GLSP Views

##### GEdgeView

A `GEdgeView` renders a line element which is routed by the `EdgeRouterRegistry`.
The view also triggers the rendering of additional elements (such as mouse handles) and edge children (such as edge labels or routing points).

```ts
configureModelElement(context, DefaultTypes.EDGE, SEdge, GEdgeView);
```

##### GIssueMarkerView

A `GIssueMarkerView` renders a issue marker on top of shapes that have a validation issue.
These issue markers are elements in the shape of an information, warning or error icon based on the severity of the issue.

```ts
configureModelElement(
  context,
  DefaultTypes.ISSUE_MARKER,
  SIssueMarker,
  GIssueMarkerView
);
```

##### RoundedCornerNodeView

A `RectangularNodeView` creates a rectangular shape based shape's size and computes and renders the corners in a rounded way, based on the given options (i.e. `getClipPathInsets`).

```ts
configureModelElement(context, DefaultTypes.NODE, SNode, RoundedCornerNodeView);
```

##### StructureCompartmentView

```ts
configureModelElement(
  context,
  "struct",
  SCompartment,
  StructureCompartmentView
);
```

##### ForeignObjectView

The `ForeignObjectView` renders a box , which is usually contained by a default node view, which takes care of resizing and moving of the element.
The view then renders the provided code snippet, e.g. a `<pre>` element.

```ts
configureModelElement(
  context,
  DefaultTypes.FOREIGN_OBJECT,
  ForeignObjectElement,
  ForeignObjectView,
  {
    disable: [selectFeature, moveFeature],
  }
);
```

An example use case for using a `ForeignObjectView` is for example a multiline-text-box.
Therefore we would create a custom text node (which extends the `ForeignObjectElement`) which also implements the `EditableLabel` interface.

```ts
export class MultiLineTextNode
  extends ForeignObjectElement
  implements EditableLabel
{
  readonly is;
  text = "";
  set bounds(bounds: Bounds) {
    /* ignore set bounds, always use the parent's bounds */
  }
  get bounds(): Bounds {
    if (isBoundsAware(this.parent)) {
      return {
        x: this.position.x,
        y: this.position.y,
        width: this.parent.bounds.width,
        height: this.parent.bounds.height,
      };
    }
    return EMPTY_BOUNDS;
  }
  get code(): string {
    return `<pre>${this.text}</pre>`;
  }
  get namespace(): string {
    return "http://www.w3.org/1999/xhtml";
  }
  get editControlDimension(): Dimension {
    return {
      width: this.bounds.width - 4,
      height: this.bounds.height - 4,
    };
  }
}
```

To register this node type, we configure it with `ForeignObjectView`, disable `moveFeature` and `selectFeature` (as this handled by its parent node).
To be able to edit this multi-line comment node we need to enable the `editLabelFeature`:

```ts
configureModelElement(
  context,
  "comment-node",
  MultiLineTextNode,
  ForeignObjectView,
  { disable: [moveFeature, selectFeature], enable: [editLabelFeature] }
);
```

</br></br>

### Styling

The style of the rendered SVG elements is controlled with plain CSS.
CSS classes can be declared directly in the corresponding view.
The `SLabelView`, for instance, adds the CSS class ‘sprotty-label’ to the generated SVG text element.

```tsx
const vnode = <text class-sprotty-label={true}>{label.text}</text>;
```

Graphical model elements also have a ‘cssClasses’ property which contains a list of CSS classes to be applied, in addition to the classes defined in the view.
For instance, the server could send the following graphical model element:

```json
{
  "id": "myCustomLabel",
  "type": "label:custom",
  "cssClasses": ["my-custom-class"]
}
```

Keeping our previous model configuration in mind, the corresponding SVG element now has two css classes applied: ‘sprotty-label’ and ‘my-custom-class’.

Based on those CSS classes, we can define CSS rules:

```css
.sprotty-label {
  fill: black;
  font-size: 100%;
}

.my-custom-class.sprotty-label {
  fill: red;
}
```

This simple style sheet declares that elements with the class ‘sprotty-label’’ should be rendered in black.
If “my-custom-class’’ is applied as well they are rendered in red.
To load this stylesheet it has to be imported somewhere in the project.
Typically this is done in the "di.config.ts” file as it’s the entry point of the diagram DI container.

```ts
import "../css/diagram.css";

const customDiagramModule= new ContainerModule((bind,unbind, isBound,rebind)=>{
…
  });
```

---

➡️ Now it's best to learn more about [client-side layouting]({{< ref "clientLayouting" >}}) next!
