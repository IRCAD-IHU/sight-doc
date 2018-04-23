.. _serviceCreation:

*************************
How to create a service ?
*************************

Implementation
===============
A service is a C++ class inherited from ``::fwServices::IService``. It will implement at least the following method:
- configuring(): parse the configuration (usually from the XML)
- starting(): initialize the service (create the gui, retrieve/initialize the data, ...)
- updating(): process
- stopping(): clear all (clear the gui, release the data, ...)

These methods are called by the *configure()*, *start()*, *update()* and *stop()* methds of the base class ``IService``.

For the example, we will create a service ``SMesher`` in a bundle ``operators``. The service will have a 
``::fwData::Image`` as input and a ``::fwData::Mesh`` as output. 

The header file ``SMesher.hpp`` should be in the folder ``<src_dir>/bundles/operators/include/operators``:

.. code-block:: cpp

    #pragma once

    #include "operators/config.hpp"

    #include <fwData/Image.hpp>
    #include <fwData/Mesh.hpp>

    #include <fwServices/IOperator.hpp>

    namespace operators
    {

    /**
     * @brief Generate a ModelSeries from an ImageSeries
     *
     * @section XML XML Configuration
     *
     * @code{.xml}
         <service type="::operators::SMesher">
             <in key="image" uid="..." />
             <out key="mesh" uid="..." />
             <generateNormals>true</generateNormals>
         </service>
       @endcode
     * @subsection Input Input
     * - \b image [::fwData::Image]: image used to generate the mesh.
     * @subsection Output Output
     * - \b mesh [::fwData::Mesh]: generated mesh.
     * @subsection Configuration Configuration
     * - \b generateNormals (optional, default: false): if true, the mesh normals will be generated.
     */
    class OPERATORS_CLASS_API SMesher : public ::fwServices::IOperator
    {
    public:
        fwCoreServiceClassDefinitionsMacro((SMesher)(::fwServices::IOperator));

        /// Constructor.
        OPERATORS_API SMesher() noexcept;

        /// Destructor.
        OPERATORS_API virtual ~SMesher() noexcept;

    protected:

        /// Parse the configuration to retrieve 'generateNormals' option.
        OPERATORS_API void configuring() override;

        /// Do nothing.
        OPERATORS_API void starting() override;

        /// Generate the mesh.
        OPERATORS_API void updating() override;

        /// Unregister the output
        OPERATORS_API void stopping() override;

    private:

        /// Option to generate or not the normals.
        bool m_generateNormals;
    };

    } // namespace operators


The file ``operators/config.hpp`` is automatically generated, it provides ``OPERATORS_CLASS_API`` and ``OPERATORS_API``
that allow to expose the methods.

**The doxygen section of the service is very important** (see :ref:`Documentation` Rule: 43), it is parsed by cmake to register 
properly the service.

In the source file ``SMesher.cpp`` should be in the folder ``<src_dir>/bundles/operators/src/operators``:

.. code-block:: cpp


    #include "operators/SMesher.hpp"

    #include <fwData/Image.hpp>
    #include <fwData/Mesh.hpp>

    namespace operators
    {

    static const ::fwServices::IService::KeyType s_IMAGE_INPUT = "image";
    static const ::fwServices::IService::KeyType s_MESH_OUTPUT = "mesh";

    //-----------------------------------------------------------------------------

    SMesher::SMesher() noexcept :
        m_generateNormals(false)
    {

    }

    //-----------------------------------------------------------------------------

    SMesher::~SMesher() noexcept
    {
    }

    //-----------------------------------------------------------------------------

    void SMesher::configuring()
    {
        const ConfigType config = this->getConfigTree();
        m_generateNormals = config.get<bool>("generateNormals", true);
    }

    //------------------------------------------------------------------------------

    void SMesher::starting()
    {

    }

    //------------------------------------------------------------------------------

    void SMesher::updating()
    {
        // retrieve the image
        ::fwData::Image::csptr image = this->getInput< ::fwData::Image >(s_IMAGE_INPUT);
        SLM_ASSERT("Input '" + s_IMAGE_INPUT + "' is not defined", image);

        ::fwData::Mesh::sptr mesh = ::fwData::Mesh::New();

        // generate the mesh
        // ...
        
        if (m_generateNormals)
        {
            // ...
        }

        // set the output mesh to be available in the configuration
        this->setOutput(s_MESH_OUTPUT, mesh);
    }

    //------------------------------------------------------------------------------

    void SMesher::stopping()
    {
        // unregister output mesh
        this->setOutput(s_MESH_OUTPUT, nullptr);
    }

    }// namespace operators


Usage
========

This service is defined in xml configuration like:

.. code-block:: xml
    
    <extension implements="::fwServices::registry::AppConfig">
    <!-- ..... -->

    <object uid="image" type="::fwData::Image" />
    <object uid="generatedMesh" type="::fwData::Mesh" src="deferred" />

    <!-- ..... -->

     <service uid="mesher" type="::operators::SMesher">
         <in key="image" uid="image" />
         <out key="mesh" uid="generatedMesh" />
         <generateNormals>true</generateNormals>
     </service>

     <!-- ..... -->
     
     <start uid="mesher" />
     <update uid="mesher" />


Connection
===========

It should be necessary to reimplement ``getAutoConnections()``, if you want to automatically connect the input data 
signals to the service. In our example, we want to call ``update()`` method when the image is modified.

.. code-block:: cpp

    IService::KeyConnectionsMap SMesher::getAutoConnections() const
    {
        KeyConnectionsMap connections;
        
        connections.push(s_IMAGE_INPUT, ::fwData::Image::s_MODIFIED_SIG, s_UPDATE_SLOT);
        
        return connections;
    }

To make this connection, you have to add ``autoConnect="yes"`` in the XML declaration of the service.

.. code-block:: xml

    <service uid="mesher" type="::operators::SMesher">
        <in key="image" uid="image" autoConnect="yes" />
        <out key="mesh" uid="generatedMesh" />
        <generateNormals>true</generateNormals>
    </service>
    

.. tip::

    If you have some problem to use your service in your application, see :ref:`serviceNotFound`.