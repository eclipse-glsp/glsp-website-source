+++
fragment = "content"
title = "Documentation"
weight = 100
+++

The GLSP provides extensible components to enable the development of diagram editors including edit functionality in (distributed) web-applications via a client-server protocol. By applying the architectural pattern of the language server protocol (LSP) to graphical languages and diagram modeling tools, it aims to achieve the same modularity and flexibility for diagram editors that has been enabled by the LSP for textual languages.

Textual languages, as supported by the LSP, significantly differ from graphical languages due to their graph-based structure, due to their way of being visualized in an editor, as well as due to  the nature of editing commands users can perform. Thus, it is impractical to directly reuse the language server protocol and language server or client frameworks. Instead, dedicated protocols and frameworks tailored to the specific requirements of graphical languages and diagram editors are required.

The main goal of GLSP is to encapsulate as much knowledge as possible about the graphical language on the server in the sense that available node and edge types, available operations, or validation is performed on the server, while the client-side editor is only responsible for rendering the diagram, providing the editing tools for the edit operations defined by the server, as well as visual feedback while editing. The actual manipulation of the underlying graph model is eventually delegated to the server, which after performing the manipulation, just sends an update to the client, which will then update the diagram rendering.

Therefore, the Graphical Language Server Platform aims at providing the following components:

  * Java-based server framework for building specific standalone diagram servers
  * Client-server protocol allowing the server to communicate available node and edge types, allowed editing operations, as well as for the client to invoke editing operations that are to be performed on the server
  * Frameworks for building diagram clients for different platforms
