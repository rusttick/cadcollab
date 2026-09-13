This issue is based on a discussion with @pierreporte and @Creymore regarding collaboration annotations.  The collaboration annotation discussion is summarized in https://github.com/FreeCAD/FreeCAD/issues/25682 and revolved around whether collaboration annotations should be versioned or not.  This led to an explanation how this relates to PDM systems.

@pierreporte describes the PDM workflow as having two parallel structures:
- models that have links between them (an assembly has a link to a part or other subassemblies)
- items that represent parts or assemblies and are described by documents (essentially files) where 3D models can be documents, a drawing can be a document, specifications can be documents

Lens currently only manages (versions of) files but items are critical for a PDM.  To make sure we all talk about the same things, we defined the terms that are in use in this context:

- **model**: a design that defines geometry in 3D that can be realized in real life
- **link**: a reference to (the geometry of) a model.  In FreeCAD a link is much more versatile but I think the definition is correct for this context.
- **part**: a single well-defined component that cannot be broken down
- **assembly**: a collection of parts or assemblies joined together to form one piece
- **sub assembly**: an assembly that is a member of another assembly
- **item**: a representation of a part or assembly in terms of documents
- **document**: a single piece of documentation for an item that can take various forms such as models, drawings, specifications.  It consists of the primary content and possibly attachments.
- **drawing**: A 2D representation of a 3D model, often used for specifications for manufacturing
- **specification**: requirements for realization of the model
- **file**: in this context a document is a file
- **version**: a specific state of a document
- **container**: a folder in which items or documents are stored
- **primary content**: the main file that defines the document, for example a PDF.
- **attachment**: supporting files that contribute to the primary contents, for example a word document that results in the PDF.

A PDM (Product Data Management) system is mainly a collaboration tool that provides a central place for CAD models and related information to ensure that all collaborators have the last version.  It is typically a central server on premise and limited to one organization.

For PDMs, item management is very important.  Since an item consists of documents, changing a document, changes its version which changes the item version.  The goal of a PDM is tracking these kinds of changes which is sometimes a legal requirement, for example in aerospace through EN 9100 compliance.  It is not required to track intermediate changes, so you have control over when to release a new version.

Since PDMs are collaboration tools, they often offer ways to discuss items.  These discussions do not increase version numbers and are often not a document.  Most likely, PDM systems do not store them in a file, but simply on the platform.  Since FreeCAD/Lens wants to share this kind of information, it makes sense to include these kinds of discussions in a file in some form, possibly in the FreeCAD file itself or as a separate file similar to BCF.

This led us to a new term:

- **meta-document**: a document that does not contribute to the specification of an item but allows discussion of the item.

At this stage, we can conclude that Lens currently does not support the notion of an item.  One possibility to have that is storing an item as a sub-directory in a workspace or the workspace as well and attach version information to it on the server.

In addition, Lens has goals that are not typical for PDMs, namely the goal to make FreeCAD designs available to FreeCAD users and even to users that don't use FreeCAD.  The idea is that Lens can host designs that users can configure/parameterize themselves and download a FreeCAD, STL, or STEP file for further processing.

Parameterization of parts is something that PDMs typically don't provide since one item is only for one object and the unique ID that identifies an object cannot be parameterized as that item should have a different ID then.  Typically in PDMs variants of parts have their own model, document, and drawing.

This led to the insight that items in traditional PDMs are very static and that for compliance and provenance reasons, they take the pragmatic route to store each variant as a separate item.  However, this does not necessarily have to be the case if the ID can encode a specific set of parameters and the system ensures that the ID always results in the same item.  The documents could in principle be generated on the fly, although it may be difficult to provide guarantees for this kind of functionality.

So, I think we have to come to the conclusion that PDMs and Lens share functionality, managing CAD files and versions, but Lens does currently not offer enough to call it a PDM system.  In addition, Lens has other goals beyond the goals of a PDM, for example sharing and configuring CAD files to FreeCAD users and even beyond FreeCAD users.
