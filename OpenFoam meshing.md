# OpenFoam Meshing

### Introduction

By default OpenFoam defines a mesh of **arbitrary polyhedral** cells in 3-D, bounded by arbitrary polygonal faces, i.e. the cells can have an **unlimited number of faces where, for each face, there is no limit on the number of edges nor any restriction on its alignment**. Recall the mesh generation for SD7003 was done and put under the **polyMesh directory.** 

As a result, **polyMesh** offers **great freedom** in mesh generation and manipulation in particular when the geometry of the domain is **complex or changes over time.** However, the **price/cost** this generality is that it can be difficult to convert meshes generated using **conventional tools.** 

To deal with this issue, OpenFoam library provides **cellShape tools** to manage conventional mesh formats based on sets of **pre-defined cell shapes**.



The **conditions** that a mesh **must satisfy** are:

1. **Points/vertices**: 
   + defined by a **vector** in units of **meters (m)**
   + compiled into a list (its position is represented by a **label**, e.g. 0, 1, 2... based on **C**++ sign convention)
   + each **point/vertex** is referred to by a **label**, e.g. 0, 1, 2, ....
   + one position in the list ONLY contain **one point**, and one point must be at **at least one face**

2. **Faces**
   + an **ordered** list of **points/vertices**
   + **internal faces** connect two cells (**never** be more than two), **normal points into cell**
   + **boundary faces**, **normal points outside of the computational domain**
3. **Edges**
   + two neighbouring points are connected by an **edge**
4. **Cells/blocks**
   + a **list of faces** in arbitrary order
   + must **completely** cover the computational domain
   + must **NOT** overlap one another
   + every cell must be **convex** and its cell centre **inside** the cell
   + every cell must **be closed** (refer to manual for geometrical closedness and topological closedness)
5. **Boundary**
   + a **list of patches which is a list of face labels, only boundary faces No internal faces**

Two **meshing utilities** provided in OpenFoam:

1. **blockMesh**

   mesh generation using blockMeshDict

2. **snappyHexMesh**

   mesh generation using snappyHexMesh

**Refer to manual for detail information.**



