.. _tuto06:

********************************************
[*Tuto06Filter*] Apply a filter on an image
********************************************

This tutorial explains how to apply a filter on an image. Here, the filter applied on the image is a threshold.
The tutorial also implements a flip filter, that won't be described in this tutorial.
The code for this second filter is available in the repository.

.. figure:: ../media/tuto06Filter.png
    :scale: 80
    :align: center


Prerequisites
===============

Before reading this tutorial, you should have seen:

 * :ref:`tuto05`
 * :ref:`BufferObjects`


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
        opImageFilter # module containing the action to performs a threshold
        fwlauncher
        appXml
    )

    moduleParam(appXml PARAM_LIST config PARAM_VALUES FilterConfig)


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

                <!-- ************************************ data ************************************ -->

                <!-- This is the source image for the filtering. -->
                <object uid="myImage1" type="::fwData::Image" />
                <!-- This is the output image for the filtering. "deferred" defines that the image is not created at the
                     beginning, but will be created by a service. -->
                <object uid="myImage2" type="::fwData::Image" src="deferred" />

                <!-- ************************************* GUI ************************************ -->

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
                            <orientation value="horizontal" />
                            <view proportion="1" />
                            <view proportion="1" />
                        </layout>
                    </gui>
                    <registry>
                        <view sid="renderingImage1" start="yes" />
                        <!-- As the output image is deferred, the service cannot be started at the beginning. -->
                        <view sid="renderingImage2" start="no" />
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

                <!-- ******************************** actions ************************************* -->

                <!-- Action to quit the application -->
                <service uid="actionQuit" type="::gui::action::SQuit" />

                <!-- Action to open image file: call update on image reader service -->
                <service uid="actionOpenImageFile" type="::gui::action::SSlotCaller" >
                    <slots>
                        <slot>imageReader/update</slot>
                    </slots>
                </service>

                <!-- Action to apply threshold filter: call update on threshold filter service -->
                <service uid="actionImageFilter" type="::gui::action::SSlotCaller" >
                    <slots>
                        <slot>imageFilter/update</slot>
                    </slots>
                </service>

                <!-- Action to apply flip filter: call 'flipAxisY' slot on flip service -->
                <service uid="actionImageFlipper" type="::gui::action::SSlotCaller">
                    <slots>
                        <slot>imageFlipper/flipAxisY</slot>
                    </slots>
                </service>

                <!-- ************************************ reader ********************************** -->

                <!-- Reader of the input image -->
                <service uid="imageReader" type="::uiIO::editor::SIOSelector">
                    <inout key="data" uid="myImage1" />
                    <type mode="reader" />
                </service>

                <!-- ************************************ operators ******************************* -->

                <!--
                    Threshold filter:
                    Applies a threshold filter. The source image is 'myImage1' and the
                    output image is 'myImage2'.
                    The two images are declared above.
                 -->
                <service uid="imageFilter" type="::opImageFilter::SThreshold">
                    <in key="source" uid="myImage1" />
                    <out key="target" uid="myImage2" />
                    <config>
                        <threshold>50</threshold>
                    </config>
                </service>

                <!--
                    Flip filter:
                    Applies a flip filter. The source image is 'myImage1' and the
                    output image is 'myImage2'.
                    The two images are declared above.
                 -->
                <service uid="imageFlipper" type="::opImageFilter::SFlip">
                    <in key="source" uid="myImage1" />
                    <out key="target" uid="myImage2" />
                </service>


                <!-- ************************************ renderers ******************************* -->

                <!--
                    Renderer of the 1st Image:
                    This is the source image for the filtering.
                -->
                <service uid="renderingImage1" type="::vtkSimpleNegato::SRenderer" autoConnect="yes" >
                    <in key="image" uid="myImage1" />
                </service>

                <!--
                    Rendered for the 2nd Image:
                    This is the output image for the filtering.
                -->
                <service uid="renderingImage2" type="::vtkSimpleNegato::SRenderer" autoConnect="yes" >
                    <in key="image" uid="myImage2" />
                </service>

                <!-- ************************************* starts ********************************* -->

                <start uid="myFrame" />
                <start uid="imageReader" />
                <start uid="imageFilter" />
                <start uid="imageFlipper" />
                <!-- start the service using a deferred image -->
                <start uid="renderingImage2" />

            </config>
        </extension>
    </plugin>


Filter service
---------------

Here, the filter service is inherited from a ``::fwServices::IOperator``

This  ``updating()`` function retrieves the two images and applies the threshold algorithm.

The ``::fwData::Image`` contains a buffer for pixel values, it is stored as a ``void *`` to allows several types of
pixels (uint8, int8, uint16, int16, double, float ...). To use the image buffer, we need to cast it to the image pixel
type. For that, we use the ``::fwTools::Dispatcher`` class which it allows to invoke a template functor according to the
image type. This is particularly useful when using template based libraries like `ITK <https://itk.org/>`_.

The image type is defined by the ``::fwTools::Type``, this class allows to serialize the image type as a string and to
retrieve the type information as sizeof, signed or not, ...
The Dispatcher allows to associate each ``Type`` to a real type as std::uint8_t, std::int8_t, std::uint16_t,... float,
double.

.. code-block:: cpp

    void SThreshold::updating()
    {
        ThresholdFilter::Parameter param; // filter parameters: threshold value, image source, image target

        auto input                  = this->getLockedInput< ::fwData::Object >(s_IMAGE_INPUT);
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

        // get image type
        ::fwTools::Type type = param.imageIn->getType();

        /* The dispatcher allows to apply the filter on any type of image.
         * It invokes the template functor ThresholdFilter using the image type.
         * - template parameters:
         *   - ::fwTools::SupportedDispatcherTypes defined all the supported type of the functor, here all the type
         *     supported by ::fwTools::Type(std::int8_t, std::uint8_t, std::int16_t, std::uint16_t, std::int32_t,
         *     std::uint32_t, std::int64_t, std::uint64_t, float, double)
         *   - ThresholdFilter: functor struct or class
         * - parameters:
         *   - type: ::fwTools::Type of the image
         *   - param: struct containing the functor parameters (here the input and output images and the threshold value)
         */
        ::fwTools::Dispatcher< ::fwTools::SupportedDispatcherTypes, ThresholdFilter >::invoke( type, param );

        // register the output image to be accessible by the other service from the XML configuration
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
            imageOut->resize(); // Allocate the image buffer

            // Get iterators on image buffers
            auto it1          = imageIn->begin<PIXELTYPE>();
            const auto it1End = imageIn->end<PIXELTYPE>();
            auto it2          = imageOut->begin<PIXELTYPE>();
            const auto it2End = imageOut->end<PIXELTYPE>();

            const PIXELTYPE maxValue = std::numeric_limits<PIXELTYPE>::max();

            // Fill the target buffer considering the thresholding
            for(; it1 != it1End && it2 != it2End; ++it1, ++it2 )
            {
                * it2 = ( *it1 < thresholdValue ) ? 0 : maxValue;
            }
        }
    };

.. note::

    The SFlip service uses the same principle as the SThreshold service. The code makes further use of templates to
    enable the filter to work 1, 2 and 3 dimension images. Furthermore, the main code is implemented in the
    imageFilterOp library, which is then called from the SFlip service.


Run
=========

To run the application, launch the following command in the install or build directory:

.. code::

    bin/tuto06filter
