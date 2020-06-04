.. _BufferObjects:

Objects containing buffers
===========================

In ``sight``, we manage different type of large buffers, as in meshes and images. As the image can be of different types
(signed or not, integers/floating), we use a generic class ``::fwData::Array`` that stores a buffer as a ``void*`` and
provides methods to access the buffer.

Mesh and Image use arrays to store their information (buffer, points, normals), but they are not accessible, so we
provide an API to access the data.

Array
-------

The array can be multi-dimensional, the number of dimensions is defined by the number of elements in the size vector.
It will perform the allocation, reallocation, destruction of the buffer.
The array contains a buffer stored as a ``void*``, its type is defined by ``::fwTools::Type`` that provides the basic
types ([u]int8, [u]int16, [u]int32, [u]int64, float and double).

Usage
*******

Allocation
~~~~~~~~~~~~~

The array buffer is allocated using the ``resize()`` methods. You can get the allocated size using ``getSizeInBytes()``.

.. warning::

    The allocated size can be different from the array size: it can happen if you called ``resize(..., false)``. It may
    be useful when you don't want to reallocate the image too often, but you need to be sure to allocate enough memory.

To resize the array, you must define the Type ([u]int[8|16|32|64], double, float) and the size of the buffer. You can
use ```setType(type)`` and ``resize(size)`` or directly call ``resize(size, type)``.

Buffer access
~~~~~~~~~~~~~~~

You can access buffer values using ``at<type>(const size_t& offset)`` or ``at<type>({x, y, z})`` methods.

.. warning::

    These methods may be slow if you use it intensively (a cast of the buffer is performed each time) and should
    not be used to parse the entire buffer (it is better to use iterators).

You can also retrieve the entire buffer  with ``getBuffer()`` but be careful no check will be done when you use it.

.. warning::
    The array must be locked for dump before accessing the buffer. It prevents the buffer to be dumped on the disk.

Iterators
~~~~~~~~~~~

To parse the buffer from beginning to end, the proper way is to use iterators. The iterator check (in debug) that you
don't iterate outside of the buffer.

The iteration depends on the given format. The format can be the buffer type ([u]int[8|16|32|64], double, float), but
can also be a simple struct like:

.. code-block:: cpp

    struct Color {
        std::uint8_t r;
        std::uint8_t g;
        std::uint8_t b;
        std::uint8_t a;
    };

This struct allows to parse the array as an RGBA buffer (RGBARGBARGBA....).

To get an iterator on the array, use ``begin<FORMAT>()`` and ``end<FORMAT>()`` methods.

.. warning::

    The iterator does not assert that the array type is the same as the given format. It only asserts (in debug) that the iterator does not iterate outside of the buffer bounds).

Example
********

.. code-block:: cpp

    const std::int16_t value = 19;
    ::fwData::Array::sptr array = ::fwData::Array::New();
    array->resize({1920, 1080}, ::fwTools::Type::s_INT16);
    auto iter          = array->begin<std::int16_t>();
    const auto iterEnd = array->end<std::int16_t>();
    for (; iter != iterEnd; ++iter)
    {
        *iter = value;
    }

.. tip::

    While these two examples will produce the same results, their performance will be different.

    Using iterators: high performance

    .. code-block:: cpp


        auto iter          = array->begin<std::int16_t>();
        const auto iterEnd = array->end<std::int16_t>();

        for (; iter != iterEnd; ++iter)
        {
            value = *iter;
        }

    Using ``at<std::int16_t>({x, y, z})`` : low performance

    .. code-block:: cpp


        const auto size = array->getSize();
        for (size_t z=0 ; z<size[2] ; ++z)
        {
            for (size_t y=0 ; y<size[1] ; ++y)
            {
                for (size_t x=0 ; x<size[0] ; ++x)
                {
                    value = array->at<std::int16_t>({x, y, z});
                }
            }
        }

Image
-------

An image contains an buffer (stored in an Array) and is defined by some parameters (size, spacing, pixel type, ...)
The buffer type is defined by ``::fwTools::Type`` that provides the basic types ([u]int8, [u]int16, [u]int32, [u]int64,
float and double).

The image size is a 3D size_t array but the third dimension can be 0 for a 2D image.
The image ``PixelFormat`` represents the buffer organization in components (GRAY_SCALE: 1 component, RGB and BGR: 3
components, RGBA and BGRA: 4 components).

Usage
********

Allocation
~~~~~~~~~~~~

The image buffer is allocated using the ``resize()`` methods. You can get the allocated size using ``getSizeInBytes()``
and ``getAllocatedSizeInBytes()``.

.. warning::

    The allocated size can be different from the image size: it can happen if you called ``setSize()`` without calling
    ``resize()``. It may be useful when you don't want to reallocate the image too often, but you need to be sure to
    allocate enough memory.

To resize the image, you must define the Type ([u]int[8|16|32|64], double, float), the size and the pixel
format of the buffer. You can use ``setSize(size)``, ``setType(type)`` and  ``setPixelFormalt(format)`` or directly call
``resize(size, type, format)``.

Buffer access
~~~~~~~~~~~~~~~

You can access voxel values using ``at<type>(IndexType id)`` or ``at<type>(IndexType x, IndexType y, IndexType z)``
methods.

.. warning::
    These methods may be slow if you use it intensively (a cast of the buffer is performed each time) and should not be used
    to parse the entire buffer (it is better to use iterators).

You can also use ``getPixelAsString()`` to retrieve the value as a string (useful for displaying information).

.. warning::

    The image must be locked for dump before accessing the buffer. It prevents the buffer to be dumped on the disk.


Iterators
~~~~~~~~~~

To parse the buffer from beginning to end, the proper way is to use iterators. The iterator checks (in debug) that you
don't iterate outside of the buffer.

The iteration depends on the given format. The format can be the buffer type ([u]int[8|16|32|64], double, float), but
can also be a simple struct like:

.. code-block:: cpp

    struct Color {
        std::uint8_t r;
        std::uint8_t g;
        std::uint8_t b;
        std::uint8_t a;
    };

This struct allows to parse the image as an RGBA buffer (RGBARGBARGBA....).

To get an iterator on the image, use ``begin<FORMAT>()`` and ``end<FORMAT>()`` methods.

.. warning::

    The iterator does not assert that the image type is the same as the given format. It only asserts (in debug) that
    the iterator does not iterate outside of the buffer bounds).


Example
********

.. code-block:: cpp

    ::fwData::Image::sptr img = ::fwData::Image::New();
    img->resize(1920, 1080, 0, ::fwTools::Type::s_UINT8, ::fwData::Image::PixelFormat::RGBA);
    auto iter    = img->begin<Color>();
    const auto iterEnd = img->end<Color>();
    for (; iter != iterEnd; ++iter)
    {
        iter->r = val1;
        iter->g = val2;
        iter->b = val2;
        iter->a = val4;
    }

.. tip::

    While these two examples will produce the same results, their performance will be different.

    Using iterators: high performance

    .. code-block:: cpp


        auto iter          = image->begin<std::int16_t>();
        const auto iterEnd = image->end<std::int16_t>();

        for (; iter != iterEnd; ++iter)
        {
            value = *iter;
        }

    Using ``at<std::int16_t>({x, y, z})`` : low performance

    .. code-block:: cpp


        const auto size = image->getSize2();
        for (size_t z=0 ; z<size[2] ; ++z)
        {
            for (size_t y=0 ; y<size[1] ; ++y)
            {
                for (size_t x=0 ; x<size[0] ; ++x)
                {
                    value = array->at<std::int16_t>(x, y, z);
                }
            }
        }


Mesh
-------

The ``::fwData::Mesh`` represents a geometric structure composed of points, lines, triangles, quads or polygons.

Structure
***********

The mesh structure contains some information stocked in ``::fwData::Array``:

m_points
    Contains point coordinates (x,y,z)

m_cellTypes
    Contains cell type (TRIAN or QUAD for the moment)
m_cellData
    Contains point indexes in m_points used to create cells: 3 indexes are necessary to create a triangle cell, 4 for
    quad cell.
m_cellDataOffsets
    Contains indexes relative to m_cellData, to retrieve the first point necessary to the cell creation.

And some additional arrays to store the mesh attributes (normals, texture coordinates and colors for points and
cells).

Example
~~~~~~~~

- m_nbPoints = number of mesh points  * 3
- m_points = [ x0, y0, z0, x1, y1, z1, x2, y2, z2, x3, y3, z3, ... ]
- m_nbCells = number of mesh cells
- m_cellTypes.size = m_nbCells
- m_cellTypes = [TRIANGLE, TRIANGLE, QUAD, QUAD, TRIANGLE ... ]
- m_cellDataOffsets.size = m_nbCells
- m_cellDataOffsets = [0, 3, 6, 10, 14, ... ] (offset shifting in  m_cellData = +3 if triangle cell rr +4 if quad cell)
- m_cellsDataSize = m_nbCells * <nb_points_per_cell> (m_nbCells * 3 if only triangle cell)
- m_cellData = [0, 1, 2, 0, 1, 3, 0, 1, 3, 5... ] ( correspond to point id )

Gets the points coordinates of the third cell:

.. code-block::

    m_cellTypes[2] => cell type = QUAD
    m_cellDataOffsets[2] => index in m_cellData of cell definition = 6
    index of p1 = m_cellData[6] = 0
    index of p2 = m_cellData[6+1] = 1
    index of p3 = m_cellData[6+2] = 3
    index of p4 = m_cellData[6+3] = 5
    p1 = [ x0=m_points[0]  y0 z0 ] ( 0 * 3 = 0 )
    p2 = [ x1=m_points[3]  y1 z1 ] ( 1 * 3 = 3 )
    p3 = [ x3=m_points[9]  y3 z3 ] ( 3 * 3 = 9 )
    p4 = [ x5=m_points[15] y5 z5 ] ( 5 * 3 = 15 )

There are other arrays to stock normal by points, normal by edges, color by points or color by cells, to short :

- Normal arrays contains normal vector (x,y,z)
- normals.size = number of mesh points (respc cells)
- normals = [ x0, y0, z0, x1, y1, z1, x2, y2, z2, x3, y3, z3, ... ]
- Color arrays contains RGBA colors
- colors.size = number of mesh points (respc cells) * 4
- colors = [ r0, g0, b0, a0, r1, g1, b1, a1, ... ]

Usage
******

Allocation
~~~~~~~~~~~~

The two methods ``reserve(...)`` and ``resize(...)`` allocate the mesh arrays. The difference between the two methods is
that ``resize(...)`` modifies the number of points and cells.

The ``pushPoint()`` and ``pushCell()`` methods add new points or cells, they increment the number of points/cells and
allocate more memory if needed. It is recommended to call ``reserve()`` method before it if you know the number of
points and cells, it avoids allocating more memory than needed.
You can call ``adjustAllocatedMemory()`` to reduce the allocated memory to the real number of points and cells.

The ``setPoint()`` and ``setCell()`` methods change the value of a point/cell at a given index.

**Example with resize(), setPoint() and setCell():**

.. code-block:: cpp

   ::fwData::Mesh::sptr mesh = ::fwData::Mesh::New();

   mesh->resize(NB_POINTS, NB_CELLS, CELL_TYPE, EXTRA_ARRAY);
   const auto lock = mesh->lock(); // prevents the buffers from being dumped on the disk

   for (size_t i = 0; i < NB_POINTS; ++i)
   {
       const std::uint8_t val                               = static_cast<uint8_t>(i);
       const ::fwData::Mesh::ColorValueType color[4]        = {val, val, val, val};
       const float floatVal                                 = static_cast<float>(i);
       const ::fwData::Mesh::NormalValueType normal[3]      = {floatVal, floatVal, floatVal};
       const ::fwData::Mesh::TexCoordValueType texCoords[2] = {floatVal, floatVal};
       const size_t value                                   = 3*i;
       mesh->setPoint(i, static_cast<float>(value), static_cast<float>(value+1), static_cast<float>(value+2));
       mesh->setPointColor(i, color);
       mesh->setPointNormal(i, normal);
       mesh->setPointTexCoord(i, texCoords);
   }

   for (size_t i = 0; i < NB_CELLS; ++i)
   {
       mesh->setCell(i, i, i+1, i+2);

       const ::fwData::Mesh::ColorValueType val             = static_cast< ::fwData::Mesh::ColorValueType >(i);
       const ::fwData::Mesh::ColorValueType color[4]        = {val, val, val, val};
       const float floatVal                                 = static_cast<float>(i);
       const ::fwData::Mesh::NormalValueType normal[3]      = {floatVal, floatVal, floatVal};
       const ::fwData::Mesh::TexCoordValueType texCoords[2] = {floatVal, floatVal};
       mesh->setCellColor(i, color);
       mesh->setCellNormal(i, normal);
       mesh->setCellTexCoord(i, texCoords);
   }


**Example with reseve(), pushPoint() and pushCell():**

.. code-block:: cpp

   ::fwData::Mesh::sptr mesh = ::fwData::Mesh::New();

   mesh->reserve(NB_POINTS, NB_CELLS, CELL_TYPE, EXTRA_ARRAY);
   const auto lock = mesh->lock(); // prevents the buffers from being dumped on the disk

   for (size_t i = 0; i < NB_POINTS; ++i)
   {
       const std::uint8_t val                               = static_cast<uint8_t>(i);
       const ::fwData::Mesh::ColorValueType color[4]        = {val, val, val, val};
       const float floatVal                                 = static_cast<float>(i);
       const ::fwData::Mesh::NormalValueType normal[3]      = {floatVal, floatVal, floatVal};
       const ::fwData::Mesh::TexCoordValueType texCoords[2] = {floatVal, floatVal};
       const size_t value                                   = 3*i;
       const auto id =
           mesh->pushPoint(static_cast<float>(value), static_cast<float>(value+1), static_cast<float>(value+2));
       mesh->setPointColor(id, color);
       mesh->setPointNormal(id, normal);
       mesh->setPointTexCoord(id, texCoords);
   }

   for (size_t i = 0; i < NB_CELLS; ++i)
   {
       const auto id = mesh->pushCell(i, i+1, i+2);

       const ::fwData::Mesh::ColorValueType val             = static_cast< ::fwData::Mesh::ColorValueType >(i);
       const ::fwData::Mesh::ColorValueType color[4]        = {val, val, val, val};
       const float floatVal                                 = static_cast<float>(i);
       const ::fwData::Mesh::NormalValueType normal[3]      = {floatVal, floatVal, floatVal};
       const ::fwData::Mesh::TexCoordValueType texCoords[2] = {floatVal, floatVal};
       mesh->setCellColor(id, color);
       mesh->setCellNormal(id, normal);
       mesh->setCellTexCoord(id, texCoords);
   }

.. warning::

    The mesh must be locked for dump before accessing the points or cells. It prevents the arrays to be dumped on the
    disk.

Iterators
~~~~~~~~~~

To access the mesh points and cells, you should use the following iterators:

 - ``::fwData::iterator::PointIterator``: to iterate through mesh points
 - ``::fwData::iterator::ConstPointIterator``: to iterate through mesh points read-only
 - ``::fwData::iterator::CellIterator``: to iterate through mesh cells
 - ``::fwData::iterator::ConstCellIterator``: to iterate through mesh cells read-only

**Example to iterate through points:**

.. code-block:: cpp

   ::fwData::Mesh::sptr mesh = ::fwData::Mesh::New();
   mesh->resize(25, 33, ::fwData::Mesh::CellType::TRIANGLE);
   auto iter    = mesh->begin< ::fwData::iterator::PointIterator >();
   const auto iterEnd = mesh->end< ::fwData::iterator::PointIterator >();
   float p[3] = {12.f, 16.f, 18.f};

  for (; iter != iterEnd; ++iter)
  {
      iter->point->x = p[0];
      iter->point->y = p[1];
      iter->point->z = p[2];
  }


**Example to iterate through cells:**

.. code-block:: cpp

   ::fwData::Mesh::sptr mesh = ::fwData::Mesh::New();
   mesh->resize(25, 33, ::fwData::Mesh::CellType::TRIANGLE);
   auto iter         = mesh->begin< ::fwData::iterator::ConstCellIterator >();
   const auto endItr = mesh->end< ::fwData::iterator::ConstCellIterator >();

   auto itrPt = mesh->begin< ::fwData::iterator::ConstPointIterator >();
   float p[3];

   for(; iter != endItr; ++iter)
   {
       const auto nbPoints = iter->nbPoints;

       for(size_t i = 0 ; i < nbPoints ; ++i)
       {
           auto pIdx = static_cast< ::fwData::iterator::ConstCellIterator::difference_type >(iter->pointIdx[i]);

           ::fwData::iterator::ConstPointIterator pointItr(itrPt + pIdx);
           p[0] = pointItr->point->x;
           p[1] = pointItr->point->y;
           p[2] = pointItr->point->z;
       }
   }


``pushCell()`` and ``setCell()`` may not be very efficient, you can use CellIterator to define the cell. But be careful
to properly define all the cell attributes.

**Example of defining cells using iterators:**

.. code-block:: cpp

   ::fwData::Mesh::sptr mesh = ::fwData::Mesh::New();
   mesh->resize(25, 33, ::fwData::Mesh::CellType::QUAD);
   auto it          = mesh->begin< ::fwData::iterator::CellIterator >();
   const auto itEnd = mesh->end< ::fwData::iterator::CellIterator >();

   const auto cellType = ::fwData::Mesh::CellType::QUAD;
   const size_t nbPointPerCell = 4;

   size_t count = 0;
   for (; it != itEnd; ++it)
   {
       // define the cell type and cell offset
       (*it->type)   = cellType;
       (*it->offset) = nbPointPerCell*count;

       // /!\ define the next offset to be able to iterate through point indices
       if (it != itEnd-1)
       {
           (*(it+1)->offset) = nbPointPerCell*(count+1);
       }

       // define the point indices
       for (size_t i = 0; i < 4; ++i)
       {
           ::fwData::Mesh::CellValueType ptIdx = val;
           it->pointIdx[i] = ptIdx;
       }
   }
