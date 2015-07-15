dof | type | common name | manupulations | use
----------------------------------------------
3 | Linear, rigid-body | translation only or constrained procrustes | translations |  images from the same subject
6 | Linear, rigid-body | rigid-body or ordinary procrustes | translations and rotations | images from the same subject, who may have moved, or from the same subject acquired on different days
7 | Linear, rigid-body with global rescale | similarity | translations, rotations, and one global scaling factor | images from the same subject acquired at different times; rodent-to-rodent; rough registration of image to template
9 | Linear, affine | 9-degree affine | translations, rotations, and scaling each of the three axes independently | one subject to another, better registration of subject to template
12 | Linear, affine | 12-degree affine | translations, rotations, and scaling at different places on each of the three axes | one subject to another, best (linear) registration of subject to template
----------------------------------------------
  
