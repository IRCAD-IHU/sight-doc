Activities
==========

An activity is defined by the extension ``::fwActivities::registry::Activities``. It is used to launch an
:ref:`AppConfig<App-Config>` with the selected data, it will create a new data ``::fwMedData::ActivitySeries`` that
inherits from a ``fwMedData::Series``.

There is two way to launch an activity:
- from a selection of Series contained in a Vector
- from a activity wizard

**from a selection of Series contained in a Vector**:

The service ``::activities::action::SActivityLauncher`` allows to launch an activity from a selection of Series. Its
role is to create the specific Activity associated with the selected data. The Series are contained in a
``::fwData::Vector`` that can be filled by the user on clicking on the Series selection widget (``::uiMedDataQt::editor::SSelector``)

This action should be followed by the service ``guiQt::editor::SDynamicView`` : this service listens the action
signals and launchs the activity in a new tab.

**from the activity wizard**:

The editor ``::activities::editor::SCreateActivity`` and the action ``::activities::action::SCreateActivity`` propose
the list of the available activity for the application, and when the user select one of them it sends a signal with the activity identifier.
The activity wizard (``::uiMedDataQt::editor::SActivityWizard``) listen this signal and display a widget to set the required data.
The ``::fwMedData::ActivitySeries`` is created and can be launched by the ``guiQt::editor::SDynamicView``.

The process to create the activity with the different services works with signals and slots.


Activity series
----------------

The ``::fwMedData::ActivitySeries`` has a ``::fwData::Composite`` that contains all the data required by the activity.

.. code-block:: cpp

    class FWMEDDATA_CLASS_API ActivitySeries : public ::fwMedData::Series
    {
    public:

        /// Constructor
        FWMEDDATA_API ActivitySeries();

        /// Destructor
        FWMEDDATA_API virtual ~ActivitySeries();

        /// Defines shallow copy
        FWMEDDATA_API void shallowCopy( const ::fwData::Object::csptr &_source );

        /// Defines deep copy
        FWMEDDATA_API void cachedDeepCopy( const ::fwData::Object::csptr &_source, DeepCopyCacheType &cache );

        /**
         * @brief Data container
         * @{ */
        ::fwData::Composite::sptr getData () const;
        void setData(const ::fwData::Composite::sptr & val);
        /**  @} */

        /**
         * @brief Activity configuration identifier
         * @{ */
        const std::string &getActivityConfigId () const;
        void setActivityConfigId (const std::string &val);
        /**  @} */

    protected:

        /// Activity configuration identifier
        ConfigIdType m_activityConfigId;

        /// Data container
        ::fwData::Composite::sptr m_data;
    };


Example
--------

.. code-block:: xml

    <extension implements="::fwActivities::registry::Activities">
       <id>myActivityId</id>
       <title>3D Visu</title>
       <desc>Activity description ...</desc>
       <icon>Bundles/media_0-1/icons/icon-3D.png</icon>
       <requirements>
            <requirement name="param1" type="::fwData::Image" /> <!-- defaults : minOccurs = 1, maxOccurs = 1-->
            <requirement name="param2" type="::fwData::Mesh" maxOccurs="3" >
                <key>Item1</key>
                <key>Item2</key>
                <key>Item3</key>
            </requirement>
            <requirement name="param3" type="::fwData::Mesh" maxOccurs="*" container="vector" />
            <requirement name="imageSeries" type="::fwMedData::ImageSeries" minOccurs="0" maxOccurs="2" />
            <requirement name="modelSeries" type="::fwMedData::ModelSeries" minOccurs="1" maxOccurs="1">
                 <desc>Description of the required data....</desc>
                 <validator>::fwActivities::validator::ImageProperties</validator>
            </requirement>
            <requirement name="transformationMatrix" type="::fwData::TransformationMatrix3D" minOccurs="0" maxOccurs="1" create="true" />
           <!-- ...-->
       </requirements>
       <builder>::fwActivities::builder::ActivitySeries</builder>
       <validator>::fwActivities::validator::ImageProperties</validator>
       <appConfig id="myAppConfigId">
           <parameters>
               <parameter replace="registeredImageUid" by="@values.param1" />
               <parameter replace="orientation" by="frontal" />
               <!-- ...-->
           </parameters>
       </appConfig>
    </extension>


The activity parameters are (in the following order):

id
*****
The activity unique identifier.

title
*******
The activity title that will be displayed on the tab.

desc
******
The description of the activity. It is displayed by the SActivityLauncher when several activity can be launched
with the selected data.


icon
*****
The path to the activity icon. It is displayed by the SActivityLauncher when several activity can be launched
with the selected data.


requirements
*************
The list of the data required to launch the activity. This data must be selected in the vector (``::fwData::Vector``).

requirement:
    A required data.

    name:
        Key used to add the data in the activity Composite.

    type:
        The data type (ex: ``::fwMedData::ImageSeries``).

    minOccurs (optional, "1" by default):
        The minimum number of occurrences of this type of object in the vector.

    maxOccurs (optional, "1" by default):
        The maximum number of occurrences of this type of object in the vector.

    container (optional, "vector" or "composite", default: composite):
        Container used to contain the data if minOccurs or maxOccurs are not "1".
        If the container is "composite", you need to specify the "key" of each object in the composite.

    create (optional, default "false"):
        If true and (minOccurrs == 0 && maxOccurs == 1), the data will be automatically created if it is not present.

    desc (optional):
        description of the parameter

    validator (optional):
        validator to check if the associated data is well formed (inherited of ::fwAtivities::IObjectValidator)


builder
********

**The builder is only used when the activity series is created from a selection of Series**.
Implementation of the activity builder. The default builder is ``::fwActivities::builder::ActivitySeries`` :
it creates the ``::fwMedData::ActivitySeries`` and adds the required data in its composite with de defined key.

The builder ``::fwActivities::builder::ActivitySeriesInitData`` allows, in addition to what the default builder does,
to create data when minOccurs == 0 and maxOccurs == 0.

validators (optional)
**********************
It defines the list of validators. If you need only one validator, you don't need the "validators" tag (only "validator").

validator (optional):
    It allows to validate if the selected required objects are correct for the activity.

    For example, the validator ``::fwActivities::validator::ImageProperties`` checks that all the selected images
    have the same size, spacing and origin.


appConfig
**********
It defines the AppConfig to launch and its parameters

id:
    Identifier of the AppConfig

parameters:
    List of the parameters required by the AppConfig

parameter:
    Defines a parameter

    replace:
        Name of the parameter as defined in the AppConfig
    by:
        Defines the string that will replace the parameter name. It should be a simple string (ex.
        frontal) or define a sesh@ path (ex. @values.myImage). The root object of the sesh@ path is the
        composite contained in the ActivitySeries.


Validators
------------

There is three types of validator :

Pre-build validator
********************

**This type of validator is only used when the activity series is created from a selection of Series**.
This type of validators checks if the current selection of data is correct to build the activity. It inherits of
::fwActivities::IValidator and must implement the methods:

.. code-block:: cpp

    ValidationType validate(
           const ::fwActivities::registry::ActivityInfo& activityInfo,
           SPTR(::fwData::Vector) currentSelection ) const;

Activity validator
*******************

This type of validator checks if the ::fwMedData::ActivitySeries is correct to launch its associated activity.
It inherits of ::fwActivities::IActivityValidator and must implement the method:

.. code-block:: cpp

    ValidationType validate(const CSPTR(::fwMedData::ActivitySeries) &activity ) const;

The validator ::fwActivities::validator::DefaultActivity is applied if no other validator is defined. It checks if
all the required objets are present in the series and if all the parameters delivered to the AppConfig are present.

It provides some method useful to implement your own validator.

Object validator
****************

This type of validator checks if the required object is well formed. It can check a single object or a Vector or
a Composite containing one type of object. It inherits of ::fwActivities::IObjectValidator and must implement the
method:

.. code-block:: cpp

    ValidationType validate(const CSPTR(::fwData::Object) &currentData ) const;


Wizard
--------

Services are available to create/launch activities :

SActivityLauncher
******************

This action allows to launch an activity according to the selected data.

.. figure:: ../media/SActivityLauncher.png
    :scale: 60
    :align: center


SCreateActivity
*****************

There is an action or an editor (``::activities::action::SCreateActivity`` or
``::activities::editor::SCreateActivity``). This services display the available activities according to the configuration.

When the activity is selected, the service sends a signal with the activity identifier. It should works with the
::uiMedData::editor::SActivityWizard that creates or updates the activitySeries.

.. code-block:: xml

    <service uid="action_newActivity" type="::activities::action::SCreateActivity">
        <!-- Filter mode 'include' allows all given activity id-s.
             Filter mode 'exclude' allows all activity id-s excepted given ones. -->
        <filter>
            <mode>include</mode>
            <id>2DVisualizationActivity</id>
            <id>3DVisualizationActivity</id>
            <id>VolumeRenderingActivity</id>
        </filter>
    </service>

filter (optional):
    it allows to filter the activity that can be proposed.

mode: 'include' or 'exclude'.
    Defines if the activity in the following list are proposed (include) or not (exclude).

id:
    id of the activity


SActivityWizard
*****************

This editor allows to select the data required by an activity in order to create the ActivitySeries.
This editor displays a tab widget (one tab by data). It works on a ::fwMedData::SeriesDB and adds the created activity
series into the seriesDB.

.. figure:: ../media/SActivityWizard.png
    :scale: 60
    :align: center

Example
********

To launch the activity, you will need to connect the services in you AppConfig:

.. code-block:: xml

    <extension implements="::fwServices::registry::AppConfig">
        <id>myExample</id>
        <config>

            <object uid="seriesDB" type="::fwMedData::SeriesDB" />
            <!-- ... -- >

            <!-- Editor to select an activity. -->
            <service uid="activitySelector" type="::activities::editor::SCreateActivity" />

            <service uid="activityCreator" type="::uiMedDataQt::editor::SActivityWizard" >
                <inout key="seriesDB" uid="seriesDB" />
                <ioSelectorConfig>SDBReaderIOSelectorConfig</ioSelectorConfig>
            </service>

            <service uid="dynamicView" type="::guiQt::editor::SDynamicView" autoConnect="yes">
                <mainActivity id="myMainActivity" closable="false" />
                <inout key="SERIESDB" uid="seriesDB" />
                <parameters>
                    <parameter replace="ICON_PATH" by="${appIconPath}" />
                </parameters>
            </service>

            <!-- Display the gui allowing to create a ::fwMedData::ActivitySeries with the required data for
                 the selected activity. -->
            <connect>
                <signal>selector/activityIDSelected</signal>
                <slot>activityCreator/createActivity</slot>
            </connect>

            <!-- Launch the activity when it is created. -->
            <connect>
                signal>activityCreator/activityCreated</signal>
                <slot>dynamicView/launchActivity</slot>
            </connect>

        </config>
    </extension>
