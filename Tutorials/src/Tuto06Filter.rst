.. _tuto06:

********************************************
[*Tuto06Filter*] Apply a filter on an image
********************************************

This tutorial explains how to apply a filter on an image. Here, the filter applied on the image is a threshold. The tutorial also implements a flip filter, that won't be described in this tutorial. The code for this second filter is available in the repository.

.. figure:: ../media/tuto06Filter.png
    :scale: 80
    :align: center


Prerequisites
===============

Before reading this tutorial, you should have seen:
 * :ref:`tuto05`


Structure
=============


Properties.cmake
------------------

This file describes the project information and requirements:

.. code-block:: cmake

    set( NAME Tuto06Filter )
    set( VERSION 0.1 )
    set( TYPE APP )
    set( DEPENDENCIES  )
    set( REQUIREMENTS
        dataReg
        servicesReg
        gui
        guiQt
        ioVTK
        uiIO
        visuVTKQt
        vtkSimpleNegato
        opImageFilter # bundle containing the action to performs a threshold
        fwlauncher
        appXml
    )

    bundleParam(appXml PARAM_LIST config PARAM_VALUES FilterConfig)


.. note::

    The Properties.cmake file of the application is used by CMake to compile the application but also to generate the
    ``profile.xml``: the file used to launch the application.


plugin.xml
------------

This file is in the ``rc/`` directory of the application. It defines the services to run.

.. code-block:: xml

    <plugin id="Tuto06Filter" version="@PROJECT_VERSION@">

        <requirement id="dataReg" />
        <requirement id="servicesReg" />
        <requirement id="visuVTKQt" />

        <extension implements="::fwServices::registry::AppConfig">
            <id>FilterConfig</id>
            <config>

                <!-- Image declaration: -->
                <!-- This is the source image for the filtering. -->
                <object uid="myImage1" type="::fwData::Image" />
                <!-- This is the output image for the filtering. -->
                <object uid="myImage2" type="::fwData::Image" />

                <!-- Windows & Main Menu -->
                <service uid="myFrame" type="::gui::frame::SDefaultFrame">
                    <gui>
                        <frame>
                            <name>Filter</name>
                            <icon>Tuto06Filter-0.1/tuto.ico</icon>
                            <minSize width="720" height="600" />
                        </frame>
                        <menuBar />
                    </gui>
                    <registry>
                        <menuBar sid="myMenuBar" start="yes" />
                        <view sid="myDefaultView" start="yes" />
                    </registry>
                </service>

                <service uid="myMenuBar" type="::gui::aspect::SDefaultMenuBar">
                    <gui>
                        <layout>
                            <menu name="File" />
                            <menu name="Filter" />
                        </layout>
                    </gui>
                    <registry>
                        <menu sid="menuFile" start="yes" />
                        <menu sid="menuFilter" start="yes" />
                    </registry>
                </service>

                <service uid="myDefaultView" type="::gui::view::SDefaultView">
                    <gui>
                        <layout type="::fwGui::LineLayoutManager">
                            <view proportion="1" />
                            <view proportion="1" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="RenderingImage1" start="yes" />
                        <view sid="RenderingImage2" start="yes" />
                    </registry>
                </service>

                <!-- Menus -->
                <service uid="menuFile" type="::gui::aspect::SDefaultMenu">
                    <gui>
                        <layout>
                            <menuItem name="Open image file" shortcut="Ctrl+O" />
                            <separator />
                            <menuItem name="Quit" specialAction="QUIT" shortcut="Ctrl+Q" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="actionOpenImageFile" start="yes" />
                        <menuItem sid="actionQuit" start="yes" />
                    </registry>
                </service>

                <service uid="menuFilter" type="::gui::aspect::SDefaultMenu">
                    <gui>
                        <layout>
                            <menuItem name="Compute Image Filter" />
                            <menuItem name="Toggle vertical image flipping" />
                        </layout>
                    </gui>
                    <registry>
                        <menuItem sid="actionImageFilter" start="yes" />
                        <menuItem sid="actionImageFlipper" start="yes" />
                    </registry>
                </service>

                <!-- Actions -->
                <service uid="actionQuit" type="::gui::action::SQuit" />
                <service uid="actionOpenImageFile" type="::gui::action::SStarter" >
                    <start uid="readerPathImageFile" />
                </service>

                <!--
                    Filter action:
                    This action applies a threshold filter. The source image is 'myImage1' and the
                    output image is 'myImage2'.
                    The two images are declared below.
                 -->
                <service uid="actionImageFilter" type="::opImageFilter::action::SThreshold">
                    <in key="source" uid="myImage1" />
                    <inout key="target" uid="myImage2" />
                </service>

                <!--
                    Filter action:
                    This action applies a flip filter. The source image is 'myImage1' and the
                    output image is 'myImage2'.
                    The two images are declared below.
                 -->
                <service uid="imageFlipper" type="::opImageFilter::SFlip">
                    <in key="source" uid="myImage1" />
                    <out key="target" uid="myImage2" />
                </service>

                <service uid="actionImageFlipper" type="::gui::action::SSlotCaller">
                    <slots>
                        <slot>imageFlipper/flipAxisY</slot>
                    </slots>
                </service>

                <!-- Renderer declaration: -->

                <!--
                    Renderer and reader of the 1st Image:
                    This is the source image for the filtering.
                -->
                <service uid="RenderingImage1" type="::vtkSimpleNegato::SRenderer" autoConnect="yes" >
                    <in key="image" uid="myImage1" />
                </service>

                <service uid="readerPathImageFile" type="::uiIO::editor::SIOSelector">
                    <inout key="data" uid="myImage1" />
                    <type mode="reader" />
                </service>

                <!--
                    Rendered for the 2nd Image:
                    This is the output image for the filtering.
                -->
                <service uid="RenderingImage2" type="::vtkSimpleNegato::SRenderer" autoConnect="yes" >
                    <in key="image" uid="myImage2" />
                </service>

                <start uid="myFrame" />
                <start uid="imageFlipper" />
                <start uid="RenderingImage2" />

            </config>
        </extension>
    </plugin>


Filter service
---------------

Here, the filter service is inherited from ``::fwGui::IActionSrv``,
which allows to use this service as an action, in this case as a button.
The member function  ``updating()`` is called when clicking on the button.
However you can inherit from another type (like ``::fwServices::IOperator``) if you do not need this behavior.

This  ``updating()`` function retrieves the two images and applies the threshold algorithm.

The ``::fwData::Image`` contains a buffer for pixel values, it is stored as a ``void *`` to allows several types of
pixel (uint8, int8, uint16, int16, double, float ...). To use the image buffer, we need to cast it to the image pixel type.
For that, we use the ``::fwTools::Dispatcher`` class which it allows to invoke a template functor according to the image type. This is
particularly useful when using template based libraries like `ITK <https://itk.org/>`_.

.. code-block:: cpp

    void SThreshold::updating()
    {
        ThresholdFilter::Parameter param; // filter parameters: threshold value, image source, image target

        ::fwData::Object::csptr input                  = this->getInput< ::fwData::Object >(s_IMAGE_INPUT);
        ::fwMedData::ImageSeries::csptr imageSeriesSrc = ::fwMedData::ImageSeries::dynamicConstCast(input);
        ::fwData::Image::csptr imageSrc                = ::fwData::Image::dynamicConstCast(input);
        ::fwData::Object::sptr output;

        // Get source/target image
        if(imageSeriesSrc)
        {
            param.imageIn                                  = imageSeriesSrc->getImage();
            ::fwMedData::ImageSeries::sptr imageSeriesDest = ::fwMedData::ImageSeries::New();

            ::fwData::Object::DeepCopyCacheType cache;
            imageSeriesDest->::fwMedData::Series::cachedDeepCopy(imageSeriesSrc, cache);
            imageSeriesDest->setDicomReference(imageSeriesSrc->getDicomReference());

            ::fwData::Image::sptr imageOut = ::fwData::Image::New();
            imageSeriesDest->setImage(imageOut);
            param.imageOut = imageOut;
            output         = imageSeriesDest;
        }
        else if(imageSrc)
        {
            param.imageIn                  = imageSrc;
            ::fwData::Image::sptr imageOut = ::fwData::Image::New();
            param.imageOut                 = imageOut;
            output                         = imageOut;
        }
        else
        {
            FW_RAISE("Wrong type: source type must be an ImageSeries or an Image");
        }

        param.thresholdValue = m_threshold;

        /*
        * The dispatcher allows to apply the filter on any type of image.
        * It invokes the template functor ThresholdFilter using the image type.
        */
        ::fwTools::DynamicType type = param.imageIn->getPixelType(); // image type

        // Invoke filter functor
        ::fwTools::Dispatcher< ::fwTools::IntrinsicTypes, ThresholdFilter >::invoke( type, param );

        this->setOutput(s_IMAGE_OUTPUT, output);
    }


The functor is a *structure* containing a *sub-structure* for the parameters (inputs and outputs) and a template
method ``operator(parameters)``.

.. code-block:: cpp

    /**
     * Functor to apply a threshold filter.
     *
     * The pixel with the value less than the threshold value will be set to 0, else the value is set to the maximum
     * value of the image pixel type.
     *
     * The functor provides a template method operator(param) to apply the filter
     */
    struct ThresholdFilter
    {
        struct Parameter
        {
            double thresholdValue; ///< threshold value.
            ::fwData::Image::csptr imageIn; ///< image source
            ::fwData::Image::sptr imageOut; ///< image target: contains the result of the filter
        };

        /**
         * @brief Applies the filter
         * @tparam PIXELTYPE image pixel type (uint8, uint16, int8, int16, float, double, ....)
         */
        template<class PIXELTYPE>
        void operator()(Parameter &param)
        {
            const PIXELTYPE thresholdValue = static_cast<PIXELTYPE>(param.thresholdValue);
            ::fwData::Image::csptr imageIn = param.imageIn;
            ::fwData::Image::sptr imageOut = param.imageOut;
            SLM_ASSERT("Sorry, image must be 3D", imageIn->getNumberOfDimensions() == 3 );
            imageOut->copyInformation(imageIn); // Copy image size, type... without copying the buffer
            imageOut->allocate(); // Allocate the image buffer

            ::fwDataTools::helper::ImageGetter imageInHelper(imageIn); // helper used to access the image source buffer
            ::fwDataTools::helper::Image imageOutHelper(imageOut); // helper used to access the image target buffer

            // Get image buffers
            const PIXELTYPE* buffer1 = (PIXELTYPE*)imageInHelper.getBuffer();
            PIXELTYPE* buffer2       = (PIXELTYPE*)imageOutHelper.getBuffer();

            // Get number of pixels
            const size_t NbPixels = imageIn->getSize()[0] * imageIn->getSize()[1] * imageIn->getSize()[2];

            // Fill the target buffer considering the thresholding
            for( size_t i = 0; i<NbPixels; ++i, ++buffer1, ++buffer2 )
            {
                * buffer2 = ( *buffer1 < thresholdValue ) ? 0 : std::numeric_limits<PIXELTYPE>::max();
            }
        }
    };

.. note::

    The SFlip service uses the same principle as the SThreshold service. The code makes further use of templates to enable the filter to work 1, 2 and 3 dimension images. Furthermore, the main code is implemented in the imageFilterOp library, which is then called from the SFlip service.


Run
=========

To run the application, launch the following command in the install or build directory:

.. code::

    bin/fwlauncher share/Tuto06Filter-0.1/profile.xml
