+++
fragment = "content"
weight = 100

title = "Getting Started"

[sidebar]
  sticky = true
+++

GLSP is architected to be very flexible and provides several implementation options:

* 🖥️ **Server**<br/>
   GLSP servers can be written either in Java or with Typescript based on nodejs.
* 🗂️ **Source Model**<br/>
   You can use any format or framework for managing your source model.
   For the most common choices, GLSP provides dedicated base modules, such as for [EMF](https://www.eclipse.org/modeling/emf), [emf.cloud](https://www.eclipse.org/emfcloud), or GModel-JSON, which contains the graphical model directly.
* 🖼️ **Tool Platform**<br/>
   Diagram editors can used in multiple tool platforms or used in plain web applications or in an Electron app.
   While GLSP editors can be integrated into any web application, GLSP provides dedicated integration components for seamlessly deploying a GLSP editor inside of [Eclipse Theia](https://www.theia-ide.org), [VS Code](https://code.visualstudio.com), or [Eclipse RCP](https://www.eclipse.org/ide).

Due to GLSP's architecture, you can even change any of those options above later on, without impacting other parts of your implementation, or support multiple variants, e.g. VS Code and Eclipse RCP, while sharing almost all of your server and client code.

To get you started quickly, GLSP provides project templates for the most popular choices.
Thus, please clone the [glsp-examples repository](https://github.com/eclipse-glsp/glsp-examples) and switch to the folder [`project templates`](https://github.com/eclipse-glsp/glsp-examples/tree/master/project-templates):

```bash
git clone https://github.com/eclipse-glsp/glsp-examples.git
cd glsp-examples/project-templates
```

Now select your preferred server language, source model format, and platform integration (see list below).
Switch to the respective folder and follow its readme file.

* [🖥️ Java ● 🗂️ EMF ● 🖼️ Theia -- `java-emf-theia`](https://github.com/eclipse-glsp/glsp-examples/tree/master/project-templates/java-emf-theia)
* [🖥️ Node ● 🗂️ Custom JSON ● 🖼️ Theia -- `node-json-theia`](https://github.com/eclipse-glsp/glsp-examples/tree/master/project-templates/node-json-theia)
* [🖥️ Node ● 🗂️ Custom JSON ● 🖼️ VS Code -- `node-json-vscode`](https://github.com/eclipse-glsp/glsp-examples/tree/master/project-templates/node-json-vscode)

If you don't find your preferred combination, please raise a question in the [Github discussions](https://github.com/eclipse-glsp/glsp/discussions).
If you need help on deciding which combination is right for you, please refer to the [integrations]({{< relref  "integrations" >}}) page or look at our [support options]({{< relref  "support" >}}).

#### What next?

Once you are up and running based on the project template above, we recommend to start working on the following aspects next:

* ➡️ Add your custom [source model]({{< relref  "sourceModel" >}}) instead of using the example model!
* ➡️ Define the diagram elements to be generated from the source model into the [graphical model]({{< relref  "gmodel" >}})!
* ➡️ Make the diagram look the way you want by adjusting the [diagram rendering and styling]({{< relref  "rendering" >}})!
* ➡️ Look at the [workflow example](https://github.com/eclipse-glsp/glsp-examples/tree/master/workflow) to explore the implementation of more advanced editor features
