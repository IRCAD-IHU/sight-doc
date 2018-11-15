.. _Documentation:

Documentation
=============

.. rule:: Document the code

    The code must be documented with **Doxygen**, an automated tool to generate documentation.

.. rule:: Location of the documentation

    Every documentation that can be useful to a user must be placed inside the header files. Thus a user of a module can
    find the declaration of a class and its documentation at the same place. Inside the implementation file, the
    documentation will give more details about algorithms.

    Moreover, every documentation must be placed next to the entity it is refering to, in order to help searching inside
    the code.

.. recommendation:: Lightweight documentation

    Inside a documentation block, only use necessary tags. This will avoid to overload the documentation and makes it
    readable. By the way, empty tags will be presented inside the generated documentation and will be useless.
    Just use an empty line to make a separation inside a documentation block.

    Don't indicate parameter types when using ``@param`` directive. This is useless since it will duplicate information
    of the function prototype.
    Also, prefer the use of ``///`` whenever possible.

Example 1 : Bad documentation block

    .. code-block:: cpp

        /**
        * @brief        A very short description.
        *
        * A longer description, giving more details about the documented piece
        * of code.
        *********************************************
        * @param
        *********************************************
        * @return
        *********************************************
        * @exception
        *********************************************
        * @todo
        *********************************************

Example 2 : Good documentation block

    .. code-block:: cpp

        /**
        * @brief        A very short description.
        *
        * A longer description, giving more details about the documented piece
        * of code.
        */

Example 3 : Function documentation

    .. code-block:: cpp

        class Sample
        {
        public:
            /**
            * Retrieve the thing.
            *
            * @return       The thing value.
            */
            const std::string& getThing( void ) const;
            /**
            * @brief        Set the thing.
            *
            * @param        thing   :  The new thing.
            */
            void setThing( const std::string& thing );

        private:
            /// stored thing
            std::string     m_thing;
        };

.. recommendation:: Structured documentation

    Doxygen provides a default structure when you generate the documentation. However, when dealing with a big
    documented entity, it is often recommended to use the group feature (``@name``). With this feature you will build a
    logical view of the class interfaces.

.. rule:: Document service

    The service must be properly documented.

    This should include first a brief description, then a long description if necessary.

    .. code-block:: cpp

        /**
         * @brief This is the short description.
         *
         * This is the long description.
         *

    After that the signals and slots must be documented in two distinct sections.

    .. code-block:: cpp

        /**
         * ...
         * @section Signals Signals
         * - \b signal2(::fwData::Mesh::sptr) : Emitted when the mesh has changed.
         * - \b signal1(std::int64_t) : Emitted when ...
         *
         * @section Slots Slots
         * - \b modified() : Modify the data.
         */

    Last the xml configuration of the service must be described into a dedicated section.
    It should indicate first the input, input/outputs and outputs in three subsections. The type and the name of the data should appear along with a short description.
    A fourth subsection  describes the rest of the parameters, and tells if it they are optional or not.

    .. code-block:: cpp

        /**
         * ...
         * @section XML XML Configuration
         *
         * @code{.xml}
                <service type="::namespace::SService">
                    <in key="data1" uid="model" />
                    <inout key="data2" uid="mesh" />
                    <out key="data3" uid="image2" />
                    <out key="data4" uid="image1" />
                    <option1>12</option1>
                    <option2>12</option2>
                </service>
           @endcode
         * @subsection Input Input
         * - \b data1 [::fwMedData::ModelSeries]: blablabla.
         * @subsection In-Out In-Out
         * - \b data2 [::fwData::Mesh]: blablabla.
         * @subsection Output Output
         * - \b data3 [::fwData::Image]: blablabla.
         * - \b data4 [::fwData::Image]: blablabla.
         * @subsection Configuration Configuration
         * - \b option1 : first option.
         * - \b option2(optional) : second option.
         *
         */

    **The XML documentation is important**, it is parsed to register properly the service.
    The `Input`, `Output` and `InOut` sections must follow the defined format:

        \\- \\b ``key_name`` [``object_type``]: ``description``

    - *key_name*: the name of the key (used to retrieve the object in the service)
    - *object_type*: class of the object with the full namespace (don't forget the ``::``)
    - *description*: the purpose of this input/output
