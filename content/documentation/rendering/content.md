+++
fragment = "content"
weight = 100

title = "Model Rendering & Styling"

[sidebar]
  sticky = true
+++

### Model Rendering

The input of the diagram rendering on the client is the GModel that has been generated on the server from the source model (see [Graphical Model Generation]({{< ref "modelGeneration" >}})) and sent to the client via a [SetModelAction](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md#242-setmodelaction) or [UpdateModelAction](https://github.com/eclipse-glsp/glsp/blob/master/PROTOCOL.md#243-updatemodelaction).
The client is then responsible for rendering the graphical model.

In order to render the received graphical model, each graphical element type needs to be associated with a view on the client.
A view defines how a specific type of graphical element shall be transformed into a corresponding SVG representation.
The derived SVG elements are then rendered on the canvas of the diagram widget.

To define a new view, we have to create a class that implements the `IView` interface and register it for a specific type that is used in the graphical model.
As an example, let’s configure that the view named `SLabelView` is used for all elements with the type “label:custom”.
Therefore, we first need to create a dependency injection module, named `customDiagramModule` below, and configure the SLabelView for the graphical model element type “label:custom” using the `configureModelElement()` utility function:

```ts
const customDiagramModule= new ContainerModule((bind, unbind, isBound, rebind) => {
    const context = { bind, unbind, isBound, rebind };
    configureModelElement(context, 'label:custom', SLabel, SLabelView);
});
```

The `configureModeElement()` function takes the inversify binding context, the graphical model type, its model class and its associated view as input. Under the hood this function sets up the necessary bindings so that the _GLSP client_ knows that

- Graphical model elements (received from the GLSP Server) with type ‘label:custom’ are deserialized to instances of `SLabel`
- Graphical model element with type ‘label:custom’ are rendered with the `SLabelView`

In order to be effective, we need to load the module `customDiagramModule` defined above in the diagram DI container, aka the root "di.config.ts" of your diagram implementation.
With that, every element of type “label:custom” will be rendered with the view implementation `SLabelView`.

Views themselves are typically implemented with [JSX](https://www.typescriptlang.org/docs/handbook/jsx.html), which simplifies the definition of SVG elements in Typescript. Therefore, the following generic imports are required in any module declaring a view to enable declaration of svg elements with JSX:


```ts
/** @jsx svg */
import { VNode } from 'snabbdom';
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
    render(label: Readonly<SLabel>, context: RenderingContext): VNode | undefined {
        if (!isEdgeLayoutable(label) && !this.isVisible(label, context)) {
            return undefined;
        }
        const vnode = <text class-sprotty-label={true}>{label.text}</text>;
        const subType = getSubType(label);
        if (subType) {
            setAttr(vnode, 'class', subType);
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
  "type":"label:custom",
  "cssClasses":[
    "my-custom-class"
  ]
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
import '../css/diagram.css';

const customDiagramModule= new ContainerModule((bind,unbind, isBound,rebind)=>{
…
  });
```
