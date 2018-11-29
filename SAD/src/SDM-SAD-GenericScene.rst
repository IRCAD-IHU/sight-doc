.. _generic_scene:

Generic Scene
==============

Overview
------------------------

A generic scene in Sight is a feature to visualize elements like meshes or images in a scene.
The scene is based on VTK. The generic scene is universal and therefore applicable for diverse visualization tasks.
It can be seen as fusion of a negatoscope and the 3D model visualization.

Manager
------------------------

The ``SRender`` is the manager service of the VTK scene. Its main task is to instantiate a VTK context (``vtkRender`` and ``vtkRenderWindow``).
In addition, it configures the rendering properties and describes a list of *adaptors*, which are dedicated services that render Sight data into this rendering context.

.. code-block:: xml

    <service uid="generiSceneUID" type="::fwRenderVTK::SRender" >
        <scene renderMode="auto|timer|none" offScreen="imageKey" width="1920" height="1080">
            <renderer id="myRenderer" layer="0" background="0.0" />
            <vtkObject id="transform" class="vtkTransform" />
            <picker id="negatodefault" vtkclass="fwVtkCellPicker" />

            <adaptor uid="meshAdaptor" />
            <adaptor uid="imageAdaptor" />

        </scene>
        <fps>30</fps><!-- used if renderMode=="timer" -->
    </service>


renderMode (optional, "auto" by default)
    This attribute is forwarded to all adaptors. For each adaptor, if renderMode="auto",  the scene is automatically
    rendered after doStart, doUpdate, doSwap, doStop and m_vtkPipelineModified=true. If renderMode="timer" the scene is
    rendered at N frame per seconds (N is defined by **fps** tag). If renderMode="none" you should call 'render' slot to
    render the scene.

offScreen (optional):
    Key of the image used for off screen render

width (optional, "1280" by default):
    Width for off screen render

height (optional, "720" by default):
    Height for off screen render

renderer
    Defines a renderer. At least one renderer is mandatory, but there can be multiple renderer on different layers.

    - **id** (mandatory): the identifier of the renderer
    - **layer** (optional): defines the layer of the vtkRenderer. This is only used if there are layered renderers.
    - **background** (optional): the background color of the rendering screen.

    The color value can be defined as a grey level value (ex . 1.0 for white) or as a hexadecimal value (ex : \#ffffff for white).

vtkObject
    Represents a vtk object. It is usually used for vtkTransform or vtkImageBlend.

   - **id** (mandatory): the identifier of the vtkObject
   - **class** (mandatory): the classname of the vtkObject to create. For example vtkTransform, vtkImageBlend, ...

picker
    Represents a picker.

    - **id** (mandatory): the identifier of the picker
    - **vtkclass** (optional, by default vtkCellPicker): the classname of the picker to create.

adaptor
    Defines the adaptors to display in the scene.

    - **uid** (mandatory): the uid of the adaptor service

Adaptor
-------------

An adaptor (inherited from ``::fwRenderVTK::IAdaptor``) is a service to manipulate or display a sight data.
Services representing an adaptor are managed by a generic scene (``::fwRenderVTK::SRender``).
The adaptors are the gateway between Sight objects and VTK objects.
To respect the principles of the framework, adaptors are kept as generic as possible.
Therefore they are reusable in other applications or even adaptors as sub-services.

As usual, an adaptor needs to implement the methods ``configuring``, ``starting``, ``stopping``, and ``updating``.


.. code-block:: cpp

    class MyAdaptor : public ::fwRenderVTK::IAdaptor
    {

    public:

        fwCoreServiceClassDefinitionsMacro ( (MyAdaptor)(::fwRenderVTK::IAdaptor) );

    protected:

        /// Parse the adaptor "config" tag
        void configuring() override;

        /// Initialize the vtk pipeline (actor, mapper, ...)
        void starting() override;

        /// Clear the vtk pipeline
        void stopping() override;

        /// Update the pipeline from the current object
        void updating() override;
    };

To ease the configuration and the link with the ``::fwRenderVTK::SRender``, the ``configuring`` and ``starting``
should contain this minimal code:

.. code-block:: cpp

    void SMesh::configuring()
    {
        this->configureParams();
        ...
    }

    void SMesh::starting()
    {
        this->initialize();

        ...

        // Request ::fwRenderVTK::SRender to trigger a rendering when it is ready
        this->requestRender();
    }


Adaptors are configured and started like other services in the xml since **Sight 12.0.0**.

.. code-block:: xml

    <service uid="meshAdaptor" type="::visuVTKAdaptor::SMesh" autoConnect="yes">
        <in key="mesh" uid="meshUID" />
        <config renderer="default" picker="" uvgen="sphere" />
    </service>

    ...

    <start uid="meshAdaptor" />
