# BIOFRAG Habitat Fragment Delineation
An R function to implement the habitat fragment delineation method and patch shape metric used by the BIOFRAG project (https://biofrag.wordpress.com/). Adapted from the original methodology created by M. Pfeifer, V. Lefebvre and R. Ewers (https://github.com/VeroL/BioFrag). 

## Background ##

Habitat fragments are most simply defined as isolated areas of suitable habitat, encompassed by a relatively inhospitable anthropogenic matrix. However, patches of habitat may often be connected to eachother by habitat corridors, which are of smaller diameter than the patches themselves. Whether two connected patches of habitat should be classified as one entity or two seperate fragments arguably depends on the level of functional connectivity provided by the connecting corridor/s. 

The level of functional connectivity that habitat corridors provide is largely determined by their width and degree of degradation, with narrow, degraded corridors often supporting only a depauperate biotic assemblage, similar to that of the matrix. However, traditional patch delineation methods, e.g., connected components labelling, group all connected habitat features into one fragment, regardless of the size of connecting corridors. Delineating habitat patches in this way could lead to the grouping of areas of habitat with limited functional connectivity and, in turn, result in misleading inference on the effects of any derived fragmentation metrics on biodiversity. 

This delineation procedure aims to overcome this potential issue by enabling researchers to incorporate ecological knowledge on taxa's edge related habitat preferences and habitat corridor use. 

### A Note on Terminology ###

The terms 'Patch' and 'Fragment' are often used interchangeably in fragmentation ecology and this could thus lead to some confusion in this vignette. I will use a dumbbell as a metaphor to hopefully clarify how I refer to 'patches', 'fragments' and 'corridors' - this terminology is admittedly not the most clear, but hopefully if you use this methodology you can just say 'We delineated habitat fragments using...' and not have to get into how elements are considered before and after delineation. 

Consider a dumbbell shaped piece of habitat. The narrow bar/grip is a 'habitat corridor' and the bulbous weights on either end are 'habitat patches' - the key difference between the elements is their width/diameter (A in the figure below). Then, once a delineation procedure has been applied and the habitat patches have either been grouped or seperated, I refer to the resultant classified elements as 'fragments' (B in the figure below).  
<br/>
<p align="center">
<img src="https://user-images.githubusercontent.com/92942535/221236002-71d07f41-bca7-4bd2-8e24-6812b9f0486c.png" width="700" height="250">
<br/>

## The Method ##

The method is based upon the watershed transformation commonly used in image segmentation, with a few prior steps added to improve ecological relevancy. These steps are centred around the extent of edge effects experienced by organisms residing within habitat patches; that is, the within-habitat distance from a matrix-habitat border over which abiotic and biotic conditions differ from those in continuous natural habitat. This in turn leads to the concept of 'core' habitat - habitat which is far enough from the matrix that it is not subject to edge effects, i.e., its biotic and abiotic conditions may be considered similar to those of natural, continuous habitat.

As in traditional delineation methods, this method defines entirely isolated habitat patches as independent fragments. However, the benefit of this procedure is that  habitat patches connected by corridors below a user-defined width are seperated into independent fragments, while patches connected by corridors above this width are grouped into a single fragment. 

As the extent of edge effects often vary among taxa, it may be neccesary to conduct multiple delineations of the same landscape when conducting multi-taxa analyses.

The process consists of 6 key steps (see below figure for visual demonstration):
  - A) If required, a categorical land cover map for the focal landscape is converted to a binary habitat raster, where 1 = Habitat and 0 = Matrix.
  - B) The binary habitat raster is then converted to a distance matrix, where cell values represent the distance between each habitat cell and the nearest non-habitat cell (i.e., distance to habitat edge).
  - C) A distance threshod is applied, whereby all habitat cells holding a value greater than the user-defined edge-effect distance are assigned the same value (the edge effect distance in units of raster cells, plus 2).
  - D) All local maxima below the pre-defined edge effect distance are flattened. This is equivalent to a H-Maxima transform and prevents small subsidiary areas of habitat that do not contain core habitat from being distinguished from larger, connected patches.
  - E) The resultant distance matrix is inverted (i.e., multiplied by -1), so that cell values decrease with increasing distance from the matrix.
  - F) The marker-controlled watershed transformation is applied. Conceptually, this fills the landscape with water, treating cell values as altitudes, and identifies as distinct elements (i.e., habitat fragments) areas where the water pools. 
<br/>
<br/>
<p align="center">
<img src="https://user-images.githubusercontent.com/92942535/221204121-6f1c0896-a48a-437f-a505-bc33534ca3bd.png" width="700" height="900">
</p>
<br/>
<br/>

## Outputs ##
 
In the resulting output, independent habitat fragments are classified according to four criteria:

  1) Habitat patches that are entirely isolated are classed as independent fragments.
  2) Patches that contain core habitat and are connected by corridors that do not contain core habitat are seperated into 2+ independent fragments.
  3) Patches that are connected by corridors that do contain core habitat are grouped into one fragment.
  4) Patches that are not entirely isolated but do not themselves contain core habitat are grouped with the largest connected patch (i.e., into a single fragment). 

BIOFRAGr provides two outputs:

  1) A raster where each cell holds the number of the identified fragment to which it belongs
  2) An sf data frame holding polygons for each identified fragment
