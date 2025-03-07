.. _doc_spatial_shader:

Spatial shaders
===============

Spatial shaders are used for shading 3D objects. They are the most complex type of shader Godot offers.
Spatial shaders are highly configurable with different render modes and different rendering options
(e.g. Subsurface Scattering, Transmission, Ambient Occlusion, Rim lighting etc). Users can optionally
write vertex, fragment, and light processor functions to affect how objects are drawn.

Render modes
^^^^^^^^^^^^

+-------------------------------+------------------------------------------------------------------------+
| Render mode                   | Description                                                            |
+===============================+========================================================================+
| **blend_mix**                 | Mix blend mode (alpha is transparency), default.                       |
+-------------------------------+------------------------------------------------------------------------+
| **blend_add**                 | Additive blend mode.                                                   |
+-------------------------------+------------------------------------------------------------------------+
| **blend_sub**                 | Subtractive blend mode.                                                |
+-------------------------------+------------------------------------------------------------------------+
| **blend_mul**                 | Multiplicative blend mode.                                             |
+-------------------------------+------------------------------------------------------------------------+
| **depth_draw_opaque**         | Only draw depth for opaque geometry (not transparent).                 |
+-------------------------------+------------------------------------------------------------------------+
| **depth_draw_always**         | Always draw depth (opaque and transparent).                            |
+-------------------------------+------------------------------------------------------------------------+
| **depth_draw_never**          | Never draw depth.                                                      |
+-------------------------------+------------------------------------------------------------------------+
| **depth_prepass_alpha**       | Do opaque depth pre-pass for transparent geometry.                     |
+-------------------------------+------------------------------------------------------------------------+
| **depth_test_disabled**       | Disable depth testing.                                                 |
+-------------------------------+------------------------------------------------------------------------+
| **sss_mode_skin**             |                                                                        |
+-------------------------------+------------------------------------------------------------------------+
| **cull_back**                 | Cull back-faces (default).                                             |
+-------------------------------+------------------------------------------------------------------------+
| **cull_front**                | Cull front-faces.                                                      |
+-------------------------------+------------------------------------------------------------------------+
| **cull_disabled**             | Culling disabled (double sided).                                       |
+-------------------------------+------------------------------------------------------------------------+
| **unshaded**                  | Result is just albedo. No lighting/shading happens in material.        |
+-------------------------------+------------------------------------------------------------------------+
| **wireframe**                 | Geometry draws using lines.                                            |
+-------------------------------+------------------------------------------------------------------------+
| **diffuse_lambert**           | Lambert shading for diffuse (default).                                 |
+-------------------------------+------------------------------------------------------------------------+
| **diffuse_lambert_wrap**      | Lambert wrapping (roughness dependent) for diffuse.                    |
+-------------------------------+------------------------------------------------------------------------+
| **diffuse_burley**            | Burley (Disney PBS) for diffuse.                                       |
+-------------------------------+------------------------------------------------------------------------+
| **diffuse_toon**              | Toon shading for diffuse.                                              |
+-------------------------------+------------------------------------------------------------------------+
| **specular_schlick_ggx**      | Schlick-GGX for specular (default).                                    |
+-------------------------------+------------------------------------------------------------------------+
| **specular_blinn**            | Blinn for specular (compatibility).                                    |
+-------------------------------+------------------------------------------------------------------------+
| **specular_phong**            | Phong for specular (compatibility).                                    |
+-------------------------------+------------------------------------------------------------------------+
| **specular_toon**             | Toon for specular.                                                     |
+-------------------------------+------------------------------------------------------------------------+
| **specular_disabled**         | Disable specular.                                                      |
+-------------------------------+------------------------------------------------------------------------+
| **skip_vertex_transform**     | VERTEX/NORMAL/etc. need to be transformed manually in vertex function. |
+-------------------------------+------------------------------------------------------------------------+
| **world_vertex_coords**       | VERTEX/NORMAL/etc. are modified in world coordinates instead of local. |
+-------------------------------+------------------------------------------------------------------------+
| **ensure_correct_normals**    | Use when non-uniform scale is applied to mesh.                         |
+-------------------------------+------------------------------------------------------------------------+
| **shadows_disabled**          | Disable computing shadows in shader.                                   |
+-------------------------------+------------------------------------------------------------------------+
| **ambient_light_disabled**    | Disable contribution from ambient light and radiance map.              |
+-------------------------------+------------------------------------------------------------------------+
| **shadow_to_opacity**         | Lighting modifies the alpha so shadowed areas are opaque and           |
|                               | non-shadowed areas are transparent. Useful for overlaying shadows onto |
|                               | a camera feed in AR.                                                   |
+-------------------------------+------------------------------------------------------------------------+
| **vertex_lighting**           | Use vertex-based lighting.                                             |
+-------------------------------+------------------------------------------------------------------------+
| **particle_trails**           | Enables the trails when used on particles geometry.                    |
+-------------------------------+------------------------------------------------------------------------+
| **alpha_to_coverage**         |                                                                        |
+-------------------------------+------------------------------------------------------------------------+
| **alpha_to_coverage_and_one** |                                                                        |
+-------------------------------+------------------------------------------------------------------------+

Built-ins
^^^^^^^^^

Values marked as "in" are read-only. Values marked as "out" are for optional writing and will
not necessarily contain sensible values. Values marked as "inout" provide a sensible default
value, and can optionally be written to. Samplers are not subjects of writing and they are
not marked.

Global built-ins
^^^^^^^^^^^^^^^^

Global built-ins are available everywhere, including custom functions.

+-------------------+----------------------------------------------------------------------------------------+
| Built-in          | Description                                                                            |
+===================+========================================================================================+
| in float **TIME** | Global time, in seconds.                                                               |
+-------------------+----------------------------------------------------------------------------------------+
| in float **PI**   | A ``PI`` constant (``3.141592``).                                                      |
|                   | A ration of circle's circumference to its diameter and amount of radians in half turn. |
+-------------------+----------------------------------------------------------------------------------------+
| in float **TAU**  | A ``TAU`` constant (``6.283185``).                                                     |
|                   | An equivalent of ``PI * 2`` and amount of radians in full turn.                        |
+-------------------+----------------------------------------------------------------------------------------+
| in float **E**    | A ``E`` constant (``2.718281``). Euler's number and a base of the natural logarithm.   |
+-------------------+----------------------------------------------------------------------------------------+

Vertex built-ins
^^^^^^^^^^^^^^^^

Vertex data (``VERTEX``, ``NORMAL``, ``TANGENT``, ``BITANGENT``) are presented in local
model space. If not written to, these values will not be modified and be passed through
as they came.

They can optionally be presented in world space by using the *world_vertex_coords* render mode.

Users can disable the built-in modelview transform (projection will still happen later) and do
it manually with the following code:

.. code-block:: glsl

    shader_type spatial;
    render_mode skip_vertex_transform;

    void vertex() {
        VERTEX = (MODELVIEW_MATRIX * vec4(VERTEX, 1.0)).xyz;
        NORMAL = normalize((MODELVIEW_MATRIX * vec4(NORMAL, 0.0)).xyz);
        // same as above for binormal and tangent, if normal mapping is used
    }

Other built-ins, such as UV, UV2 and COLOR, are also passed through to the fragment function if not modified.

Users can override the modelview and projection transforms using the ``POSITION`` built-in. When ``POSITION`` is used,
the value from ``VERTEX`` is ignored and projection does not happen. However, the value passed to the fragment shader
still comes from ``VERTEX``.

For instancing, the INSTANCE_CUSTOM variable contains the instance custom data. When using particles, this information
is usually:

* **x**: Rotation angle in radians.
* **y**: Phase during lifetime (0 to 1).
* **z**: Animation frame.

This allows you to easily adjust the shader to a particle system using default particles material. When writing a custom particle
shader, this value can be used as desired.

+----------------------------------------+--------------------------------------------------------+
| Built-in                               | Description                                            |
+========================================+========================================================+
| in vec2 **VIEWPORT_SIZE**              | Size of viewport (in pixels).                          |
+----------------------------------------+--------------------------------------------------------+
| in mat4 **INV_CAMERA_MATRIX**          | World space to view space transform.                   |
+----------------------------------------+--------------------------------------------------------+
| in mat4 **CAMERA_MATRIX**              | View space to world space transform.                   |
+----------------------------------------+--------------------------------------------------------+
| in mat4 **INV_PROJECTION_MATRIX**      | Clip space to view space transform.                    |
+----------------------------------------+--------------------------------------------------------+
| in bool **OUTPUT_IS_SRGB**             | ``true`` when calculations happen in sRGB color space  |
|                                        | (``true`` in GLES2, ``false`` in GLES3).               |
+----------------------------------------+--------------------------------------------------------+
| in int **INSTANCE_ID**                 | Instance ID for instancing.                            |
+----------------------------------------+--------------------------------------------------------+
| in vec4 **INSTANCE_CUSTOM**            | Instance custom data (for particles, mostly).          |
+----------------------------------------+--------------------------------------------------------+
| in int **VIEW_INDEX**                  |                                                        |
+----------------------------------------+--------------------------------------------------------+
| in int **VIEW_MONO_LEFT**              |                                                        |
+----------------------------------------+--------------------------------------------------------+
| in int **VIEW_RIGHT**                  |                                                        |
+----------------------------------------+--------------------------------------------------------+
| inout vec3 **VERTEX**                  | Vertex in local coordinates.                           |
+----------------------------------------+--------------------------------------------------------+
| inout vec3 **NORMAL**                  | Normal in local coordinates.                           |
+----------------------------------------+--------------------------------------------------------+
| inout vec3 **TANGENT**                 | Tangent in local coordinates.                          |
+----------------------------------------+--------------------------------------------------------+
| inout vec3 **BINORMAL**                | Binormal in local coordinates.                         |
+----------------------------------------+--------------------------------------------------------+
| out vec4 **POSITION**                  | If written to, overrides final vertex position.        |
+----------------------------------------+--------------------------------------------------------+
| inout vec2 **UV**                      | UV main channel.                                       |
+----------------------------------------+--------------------------------------------------------+
| inout vec2 **UV2**                     | UV secondary channel.                                  |
+----------------------------------------+--------------------------------------------------------+
| inout vec4 **COLOR**                   | Color from vertices.                                   |
+----------------------------------------+--------------------------------------------------------+
| out float **ROUGHNESS**                | Roughness for vertex lighting.                         |
+----------------------------------------+--------------------------------------------------------+
| inout float **POINT_SIZE**             | Point size for point rendering.                        |
+----------------------------------------+--------------------------------------------------------+
| inout mat4 **MODELVIEW_MATRIX**        | Model space to view space transform (use if possible). |
+----------------------------------------+--------------------------------------------------------+
| inout mat3 **MODELVIEW_NORMAL_MATRIX** |                                                        |
+----------------------------------------+--------------------------------------------------------+
| inout mat4 **WORLD_MATRIX**            | Model space to world space transform.                  |
+----------------------------------------+--------------------------------------------------------+
| inout mat3 **WORLD_NORMAL_MATRIX**     |                                                        |
+----------------------------------------+--------------------------------------------------------+
| inout mat4 **PROJECTION_MATRIX**       | View space to clip space transform.                    |
+----------------------------------------+--------------------------------------------------------+
| inout uvec4 **BONE_INDICES**           |                                                        |
+----------------------------------------+--------------------------------------------------------+
| inout vec4 **BONE_WEIGHTS**            |                                                        |
+----------------------------------------+--------------------------------------------------------+
| out vec4 **CUSTOM0**                   |                                                        |
+----------------------------------------+--------------------------------------------------------+
| out vec4 **CUSTOM1**                   |                                                        |
+----------------------------------------+--------------------------------------------------------+
| out vec4 **CUSTOM2**                   |                                                        |
+----------------------------------------+--------------------------------------------------------+
| out vec4 **CUSTOM3**                   |                                                        |
+----------------------------------------+--------------------------------------------------------+

Fragment built-ins
^^^^^^^^^^^^^^^^^^

The default use of a Godot fragment processor function is to set up the material properties of your object
and to let the built-in renderer handle the final shading. However, you are not required to use all
these properties, and if you don't write to them, Godot will optimize away the corresponding functionality.

+----------------------------------------+--------------------------------------------------------------------------------------------------+
| Built-in                               | Description                                                                                      |
+========================================+==================================================================================================+
| in vec2 **VIEWPORT_SIZE**              | Size of viewport (in pixels).                                                                    |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in vec4 **FRAGCOORD**                  | Coordinate of pixel center in screen space. ``xy`` specifies position in window, ``z``           |
|                                        | specifies fragment depth if ``DEPTH`` is not used. Origin is lower-left.                         |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in bool **FRONT_FACING**               | ``true`` if current face if front face.                                                          |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in vec3 **VIEW**                       | Vector from camera to fragment position (in view space).                                         |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in vec2 **UV**                         | UV that comes from vertex function.                                                              |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in vec2 **UV2**                        | UV2 that comes from vertex function.                                                             |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in vec4 **COLOR**                      | COLOR that comes from vertex function.                                                           |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in vec2 **POINT_COORD**                | Point Coordinate for drawing points with POINT_SIZE.                                             |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in bool **OUTPUT_IS_SRGB**             | ``true`` when calculations happen in sRGB color space (``true`` in GLES2, ``false`` in GLES3).   |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in mat4 **WORLD_MATRIX**               | Model space to world space transform.                                                            |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in mat3 **WORLD_NORMAL_MATRIX**        |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in mat4 **INV_CAMERA_MATRIX**          | World space to view space transform.                                                             |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in mat4 **CAMERA_MATRIX**              | View space to world space transform.                                                             |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in mat4 **PROJECTION_MATRIX**          | View space to clip space transform.                                                              |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in mat4 **INV_PROJECTION_MATRIX**      | Clip space to view space transform.                                                              |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in vec3 **VERTEX**                     | Vertex that comes from vertex function (default, in view space).                                 |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in int **VIEW_INDEX**                  |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in int **VIEW_MONO_LEFT**              |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in int **VIEW_RIGHT**                  |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| sampler2D **SCREEN_TEXTURE**           | Built-in Texture for reading from the screen. Mipmaps contain increasingly blurred copies.       |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| in vec2 **SCREEN_UV**                  | Screen UV coordinate for current pixel.                                                          |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| sampler2D **NORMAL_ROUGHNESS_TEXTURE** |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| sampler2D **DEPTH_TEXTURE**            | Built-in Texture for reading depth from the screen. Must convert to linear using INV_PROJECTION. |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **DEPTH**                    | Custom depth value (0..1).                                                                       |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| inout vec3 **NORMAL**                  | Normal that comes from vertex function (default, in view space).                                 |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| inout vec3 **TANGENT**                 | Tangent that comes from vertex function.                                                         |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| inout vec3 **BINORMAL**                | Binormal that comes from vertex function.                                                        |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out vec3 **NORMAL_MAP**                | Set normal here if reading normal from a texture instead of NORMAL.                              |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **NORMAL_MAP_DEPTH**         | Depth from variable above. Defaults to 1.0.                                                      |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out vec3 **ALBEDO**                    | Albedo (default white).                                                                          |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **ALPHA**                    | Alpha (0..1); if written to, the material will go to the transparent pipeline.                   |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **ALPHA_SCISSOR_THRESHOLD**  | If written to, values below a certain amount of alpha are discarded.                             |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **ALPHA_HASH_SCALE**         |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **ALPHA_ANTIALIASING_EDGE**  |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out vec2 **ALPHA_TEXTURE_COORDINATE**  |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **METALLIC**                 | Metallic (0..1).                                                                                 |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **SPECULAR**                 | Specular. Defaults to 0.5, best not to modify unless you want to change IOR.                     |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **ROUGHNESS**                | Roughness (0..1).                                                                                |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **RIM**                      | Rim (0..1). If used, Godot calculates rim lighting.                                              |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **RIM_TINT**                 | Rim Tint, goes from 0 (white) to 1 (albedo). If used, Godot calculates rim lighting.             |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **CLEARCOAT**                | Small added specular blob. If used, Godot calculates Clearcoat.                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **CLEARCOAT_GLOSS**          | Gloss of Clearcoat. If used, Godot calculates Clearcoat.                                         |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **ANISOTROPY**               | For distorting the specular blob according to tangent space.                                     |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out vec2 **ANISOTROPY_FLOW**           | Distortion direction, use with flowmaps.                                                         |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **SSS_STRENGTH**             | Strength of Subsurface Scattering. If used, Subsurface Scattering will be applied to object.     |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out vec4 **SSS_TRANSMITTANCE_COLOR**   |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **SSS_TRANSMITTANCE_DEPTH**  |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **SSS_TRANSMITTANCE_BOOST**  |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| inout vec3 **BACKLIGHT**               |                                                                                                  |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **AO**                       | Strength of Ambient Occlusion. For use with pre-baked AO.                                        |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out float **AO_LIGHT_AFFECT**          | How much AO affects lights (0..1; default 0).                                                    |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out vec3 **EMISSION**                  | Emission color (can go over 1,1,1 for HDR).                                                      |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out vec4 **FOG**                       | If written to, blends final pixel color with FOG.rgb based on FOG.a.                             |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out vec4 **RADIANCE**                  | If written to, blends environment map radiance with RADIANCE.rgb based on RADIANCE.a.            |
+----------------------------------------+--------------------------------------------------------------------------------------------------+
| out vec4 **IRRADIANCE**                | If written to, blends environment map IRRADIANCE with IRRADIANCE.rgb based on IRRADIANCE.a.      |
+----------------------------------------+--------------------------------------------------------------------------------------------------+

Light built-ins
^^^^^^^^^^^^^^^

Writing light processor functions is completely optional. You can skip the light function by setting
render_mode to ``unshaded``. If no light function is written, Godot will use the material
properties written to in the fragment function to calculate the lighting for you (subject to
the render_mode).

To write a light function, assign something to ``DIFFUSE_LIGHT`` or ``SPECULAR_LIGHT``. Assigning nothing
means no light is processed.

The light function is called for every light in every pixel. It is called within a loop for
each light type.

Below is an example of a custom light function using a Lambertian lighting model:

.. code-block:: glsl

    void light() {
        DIFFUSE_LIGHT += clamp(dot(NORMAL, LIGHT), 0.0, 1.0) * ATTENUATION * ALBEDO;
    }

If you want the lights to add together, add the light contribution to ``DIFFUSE_LIGHT`` using ``+=``, rather than overwriting it.

.. warning::

    The ``light()`` function won't be run if the ``vertex_lighting`` render mode
    is enabled, or if
    **Rendering > Quality > Shading > Force Vertex Shading** is enabled in the
    Project Settings. (It's enabled by default on mobile platforms.)

+-----------------------------------+----------------------------------------------------+
| Built-in                          | Description                                        |
+===================================+====================================================+
| in vec2 **VIEWPORT_SIZE**         | Size of viewport (in pixels).                      |
+-----------------------------------+----------------------------------------------------+
| in vec4 **FRAGCOORD**             | Coordinate of pixel center in screen space.        |
|                                   | ``xy`` specifies position in window, ``z``         |
|                                   | specifies fragment depth if ``DEPTH`` is not used. |
|                                   | Origin is lower-left.                              |
+-----------------------------------+----------------------------------------------------+
| in mat4 **WORLD_MATRIX**          | Model space to world space transform.              |
+-----------------------------------+----------------------------------------------------+
| in mat4 **CAMERA_MATRIX**         | View space to world space transform.               |
+-----------------------------------+----------------------------------------------------+
| in mat4 **INV_CAMERA_MATRIX**     | World space to view space transform.               |
+-----------------------------------+----------------------------------------------------+
| in mat4 **PROJECTION_MATRIX**     | View space to clip space transform.                |
+-----------------------------------+----------------------------------------------------+
| in mat4 **INV_PROJECTION_MATRIX** | Clip space to view space transform.                |
+-----------------------------------+----------------------------------------------------+
| in vec3 **NORMAL**                | Normal vector, in view space.                      |
+-----------------------------------+----------------------------------------------------+
| in vec2 **UV**                    | UV that comes from vertex function.                |
+-----------------------------------+----------------------------------------------------+
| in vec2 **UV2**                   | UV2 that comes from vertex function.               |
+-----------------------------------+----------------------------------------------------+
| in vec3 **VIEW**                  | View vector, in view space.                        |
+-----------------------------------+----------------------------------------------------+
| in vec3 **LIGHT**                 | Light Vector, in view space.                       |
+-----------------------------------+----------------------------------------------------+
| in vec3 **LIGHT_COLOR**           | Color of light multiplied by energy.               |
+-----------------------------------+----------------------------------------------------+
| in float **ATTENUATION**          | Attenuation based on distance or shadow.           |
+-----------------------------------+----------------------------------------------------+
| in vec3 **SHADOW_ATTENUATION**    |                                                    |
+-----------------------------------+----------------------------------------------------+
| in vec3 **ALBEDO**                | Base albedo.                                       |
+-----------------------------------+----------------------------------------------------+
| in vec3 **BACKLIGHT**             |                                                    |
+-----------------------------------+----------------------------------------------------+
| in float **METALLIC**             | Metallic.                                          |
+-----------------------------------+----------------------------------------------------+
| in float **ROUGHNESS**            | Roughness.                                         |
+-----------------------------------+----------------------------------------------------+
| in bool **OUTPUT_IS_SRGB**        | ``true`` when calculations happen in sRGB color    |
|                                   | space (``true`` in GLES2, ``false`` in GLES3).     |
+-----------------------------------+----------------------------------------------------+
| out vec3 **DIFFUSE_LIGHT**        | Diffuse light result.                              |
+-----------------------------------+----------------------------------------------------+
| out vec3 **SPECULAR_LIGHT**       | Specular light result.                             |
+-----------------------------------+----------------------------------------------------+
| out float **ALPHA**               | Alpha (0..1); if written to, the material will go  |
|                                   | to the transparent pipeline.                       |
+-----------------------------------+----------------------------------------------------+
