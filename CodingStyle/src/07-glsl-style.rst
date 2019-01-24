Glsl coding
============

Source and files
-----------------

.. rule :: Files extensions

    Implementation files use the extension ``.glsl``.

.. rule :: version

    Always specify the GL target version, we only allow versions up to 410 for maximum portability.

.. rule:: Document the code

    The code must be documented, especially the mathematical parts of the code.

Naming conventions
------------------

.. rule :: File

    The file's name should end with the shader's type, ``_VP`` for vertex programs, ``_TC`` for tessellation control programs, ``_FE`` for tessellation evalutaion programs, ``_GP`` for geometry programs and ``_FP`` for fragment programs, ``_VP`` for vertex programs and ``_GP`` for geometry programs.

.. rule :: Function and method names

    Functions and methods names must be written in camel case.

.. recommendation :: Correct naming of functions

    Try as much as possible to help the users of your code by using comprehensive names.

.. rule :: Variable naming

    #. Variable names must start with their storage qualifier :

        - Uniform variables must begin with ``u_``.
        - Function parameters must begin with ``_``.
        - Vertex shader inputs must begin with ``v_in_``.
        - Vertex shader outputs must begin with ``v_out_``.
        - Tessellation control shader outputs must begin with ``tc_out_``.
        - Tessellation evaluation shader outputs must begin with ``te_out_``.
        - Geometry shader outputs must begin with ``g_out_``.
        - Fragment shader outputs must begin with ``f_out_``.

    #. The following characters must specify the type of the variable :

        - int         : i
        - bool        : b
        - uint        : ui
        - double      : d
        - float       : f
        - sampler2D   : s2
        - sampler3D   : s3
        - vec2        : f2
        - vec3        : f3
        - vec4        : f4
        - mat2        : m2
        - mat3        : m3
        - mat4        : m4
        - ...

    #. Vectors can end by some information to specify their type if it's possible ;

        - Position    : Pos
        - Origin      : Ori
        - Direction   : Dir
        - Color       : Col
        - Length      : Len
        - Size        : Size

    #. Vectors representing geometrical and matrices transform. You should specify, if possible, their coordinate system. We define the following notation for the most commonly used spaces: (You may read this `documentation <https://www.khronos.org/opengl/wiki/Vertex_Post-Processing#Perspective_divide>`_ if you don't know them) :

        - Texture space             : Ts
        - Model space               : Ms
        - World space               : Ws
        - View space                : Vs
        - Clip space                : Cs
        - Normalized device space   : Ns
        - Window space              : Ss

        For vectors, the coordinate system should be appended to the name. In the case of transform matrices, we need to define both, the destination and source spaces, these should be separated by an underscore with the source being last.

        e.g. m4Ss_Ms defines a matrix transforming model vertices to window space points (often known as the model-view-projection matrix)

    #. If the variable is normalized, the last character must be ``N``.

    .. code-block :: glsl

        uniform float u_fClippingNearLen;
        uniform float u_fClippingFarLen;

        /// Converts a position in OpenGL's normalized device coordinates (NDC) to the model space.
        vec3 ndcToModelSpacePosition(in vec3 _f3FragPos_Ns, in mat4 _m4Ms_Cs)
        {
            vec4 f4ClipPos_Cs;
            f4ClipPos_Cs.w   = (2 * u_fClippingNearLen * u_fClippingFarLen)  / (u_fClippingNearLen + u_fClippingFarLen + _f3FragPos_Ns.z * (u_fClippingNearLen - u_fClippingFarLen));
            f4ClipPos_Cs.xyz = _f3FragPos_Ns * f4ClipPos_Cs.w;

            return _m4Ms_Cs * f4ClipPos_Cs;
        }
