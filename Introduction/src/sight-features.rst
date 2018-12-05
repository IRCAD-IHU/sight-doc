**************
Sight features
**************

-------------
Main features
-------------

 - Reader/Writer
    - DICOM reader/writer
        - PACS connection
        - 3D mesh segmentation reader/writer
        - DICOM filter for reader
    - VTK (images and meshes)
    - ITK
    - Atoms (our custom in-out data format)
 - Visualisation
    - 2D and 3D multi-planar reconstruction
    - volume rendering
    - 3D meshes


--------------------------
Augmented reality features
--------------------------

- webcam, network video and local video playing based on QtMultimedia_,
- mono and stereo camera calibration,
- ArUco_ optical markers tracking,
- openIGTLink_ support through clients and servers services,
- TimeLine data, allowing to store buffers of various data (video, matrices, markers, etc...).
  These can be used to synchronize these data accross time.

.. _QtMultimedia: http://doc.qt.io/qt-5/qtmultimedia-index.html
.. _ArUco: https://sourceforge.net/projects/aruco/
.. _openIGTLink: http://openigtlink.org/

-------------
Ogre features
-------------

- regular surfacic meshes rendering for reconstruction,
- 2D and 3D negato medical image display with transfer function support,
- Order-independent transparency (several techniques implemented such as Depth-peeling,
  Weighted-blended order independent transparency, and Hybrid Transparency),
- customizable shaders and parameters edition.
