---
title: directed-cubical 0.1.2.0 
date: 2014-06-08
tags: 
  - haskell
---

I finally got around to writing some
[code](http://hackage.haskell.org/package/directed-cubical) to output finite
directed cubical complexes in VTK format. Initially I was trying to get it to
work with the [XTK project](http://github.com/xtk/X) in hopes of getting some
slick WebGL-based presentations, but XTK's parser for VTK polydata files only
supports triangles at the moment (rather than general polygons). I was able to
partially work around this by rendering only the triangulations of 2-skeleta,
but then XTK only recognized the 2-simplices and not the 1-simplices which were
essential to the correct visualization of the corner-reduction algo. 

At this point I punted and wrote the code to output files compatible with
[ParaView](http://www.paraview.org) (and the specification, of course). ParaView
gave me a quick assurance that the bug was with XTK and not me, and then I was
able to generate some [cool animations](http://www.math.uwo.ca/~mmisamor/animations).  
