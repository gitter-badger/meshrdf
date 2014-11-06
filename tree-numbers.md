---
title: Tree Numbers
layout: page
resource: true
categories:
- Data Model
---

### RDF Graph Diagram

The following diagram illustrates the problem of tree numbers, that impose multiple
overlapping hierarchical relationships between descriptors. In the diagram, every
arrow indicates a skos:broader relationship.

In this particular example, "Eye" belongs to two overlapping trees -- one that is
depicted in green and one in purple.

According to the green tree, "Eyebrow" has broader concept "Eye", which has broader
concept "Face". According to the purple tree, "Oculomotor Muscles" has broader concept
"Eye", which has broader concept "Sense Organ".

![Tree Numbers Produce Overlapping Hierarchies](images/BroaderRelations.png){: class="rdf-graph"}


Leaving aside the question of whether or not these particular relationships make sense, this is the
way the tree numbers currently exist in MeSH.

The problem comes when you try to use these relations to "walk up the tree".  If these properties
were transitive, then one could conclude that "Eyebrows" has broader concept "Sense Organ", or that
"Oculomotor Muscles" has broader concept "Face", but neither of those is true according to the MeSH trees.

The way that this is modeled in MeSH RDF is illustrated in the following graph.  Each tree number
is reified as a first-class resource.  `skos:broader` relations (which are not transitive) are asserted
between the `Descriptor`s, and `skos:broaderTransitive` relations (which, obviously, are) are asserted
between the `TreeNumber`s.

![Tree Numbers RDF Graph Model](images/BroaderRelationsWithTreeNodes.png){: class="rdf-graph"}


With this model, if a user wanted to find all the "broader" ancestors of Eyebrows (D005138) then he or
she could do so, using the SPARQL transitivity operator ("+"), with the following query:

```sparql
select ?ancestor
where {
    mesh:D005138 mesh:hasTreeNode ?treeNode .
    ?treeNode skos:broaderTransitive+ ?ancestorTreeNode .
    ?ancestor mesh:hasTreeNode ?ancestorTreeNode
}
```

### SPARQL

The following <span class='invoke-sparql'>SPARQL query</span> creates a table that shows the relationships between "Eye"
and each of the immediate broader and narrower concepts for each of its tree numbers:

```sparql
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>

SELECT *
from <http://id.nlm.nih.gov/mesh2014>
where {
  mesh:D005123 meshv:treeNumber ?tree_number .

  {
    select ?tree_number
           ?broader_tree_number ?broader_descriptor
           ?narrower_tree_number ?narrower_descriptor
    where {
      ?tree_number skos:broaderTransitive ?broader_tree_number .
      ?broader_descriptor meshv:treeNumber ?broader_tree_number .
      mesh:D005123 skos:broader ?broader_descriptor .

      ?narrower_tree_number skos:broaderTransitive ?tree_number .
      ?narrower_descriptor meshv:treeNumber ?narrower_tree_number .
      ?narrower_descriptor skos:broader mesh:D005123 .
    }
  }
}
```
<span class='invoke-sparql'>SPARQL query</span> query is very similar, but outputs
RDF (using the CONSTRUCT clause) and
reconstructs the graph that is shown in the figure above.

```sparql
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX meshv: <http://id.nlm.nih.gov/mesh/vocab#>

CONSTRUCT {
  mesh:D005123 meshv:treeNumber ?tree_number .
  ?tree_number skos:broaderTransitive ?broader_tree_number .
  ?broader_descriptor meshv:treeNumber ?broader_tree_number .
  mesh:D005123 skos:broader ?broader_descriptor .

  ?narrower_tree_number skos:broaderTransitive ?tree_number .
  ?narrower_descriptor meshv:treeNumber ?narrower_tree_number .
  ?narrower_descriptor skos:broader mesh:D005123 .
}
from <http://id.nlm.nih.gov/mesh2014>
where {
  mesh:D005123 meshv:treeNumber ?tree_number .

  {
    select ?tree_number
           ?broader_tree_number ?broader_descriptor
           ?narrower_tree_number ?narrower_descriptor
    where {
      ?tree_number skos:broaderTransitive ?broader_tree_number .
      ?broader_descriptor meshv:treeNumber ?broader_tree_number .
      mesh:D005123 skos:broader ?broader_descriptor .

      ?narrower_tree_number skos:broaderTransitive ?tree_number .
      ?narrower_descriptor meshv:treeNumber ?narrower_tree_number .
      ?narrower_descriptor skos:broader mesh:D005123 .
    }
  }
}
```

### MeSH RDF

```
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix mesh: <http://id.nlm.nih.gov/mesh/> .
@prefix meshv:  <http://id.nlm.nih.gov/mesh/vocab#> .

# Eye
mesh:D005123 skos:broader mesh:D005145, mesh:D012679 ;
             meshv:treeNumber mesh:A09.371 ,
                              mesh:A01.456.505.420 .

# Eyebrows
mesh:D005138 skos:broader mesh:D005123 ;
             meshv:treeNumber mesh:A01.456.505.420.338 .

# Face
mesh:D005145 meshv:treeNumber mesh:A01.456.505 .

# Oculomotor Muscles
mesh:D009801 skos:broader mesh:D005123 ;
             meshv:treeNumber mesh:A09.371.613 .

# Sense Organs
mesh:D012679 meshv:treeNumber  mesh:A09 .

# Relations among TreeNumbers
mesh:A01.456.505.420  skos:broaderTransitive  mesh:A01.456.505 .
mesh:A09.371  skos:broaderTransitive  mesh:A09 .
mesh:A01.456.505.420.338  skos:broaderTransitive  mesh:A01.456.505.420 .
mesh:A09.371.613  skos:broaderTransitive  mesh:A09.371 .
```

### MeSH XML

```xml
<DescriptorRecordSet LanguageCode="eng">

  <DescriptorRecord DescriptorClass="1">
    <DescriptorUI>D005123</DescriptorUI>
    <DescriptorName>
      <String>Eye</String>
    </DescriptorName>
    ...
    <TreeNumberList>
      <TreeNumber>A01.456.505.420</TreeNumber>
      <TreeNumber>A09.371</TreeNumber>
    </TreeNumberList>
    ...
  </DescriptorRecord>

  <DescriptorRecord DescriptorClass="1">
    <DescriptorUI>D005145</DescriptorUI>
    <DescriptorName>
      <String>Face</String>
    </DescriptorName>
    ...
    <TreeNumberList>
      <TreeNumber>A01.456.505</TreeNumber>
    </TreeNumberList>
    ...
  </DescriptorRecord>

  <DescriptorRecord DescriptorClass="1">
    <DescriptorUI>D005138</DescriptorUI>
    <DescriptorName>
      <String>Eyebrows</String>
    </DescriptorName>
    ...
    <TreeNumberList>
      <TreeNumber>A01.456.505.420.338</TreeNumber>
      <TreeNumber>A17.360.296</TreeNumber>
    </TreeNumberList>
    ...
  </DescriptorRecord>

  <DescriptorRecord DescriptorClass="1">
    <DescriptorUI>D009801</DescriptorUI>
    <DescriptorName>
      <String>Oculomotor Muscles</String>
    </DescriptorName>
    ...
    <TreeNumberList>
      <TreeNumber>A02.633.567.700</TreeNumber>
      <TreeNumber>A09.371.613</TreeNumber>
    </TreeNumberList>
    ...
  </DescriptorRecord>

  <DescriptorRecord DescriptorClass="1">
    <DescriptorUI>D012679</DescriptorUI>
    <DescriptorName>
      <String>Sense Organs</String>
    </DescriptorName>
    ...
    <TreeNumberList>
      <TreeNumber>A09</TreeNumber>
    </TreeNumberList>
    ...
  </DescriptorRecord>

</DescriptorRecordSet>
```
