# Three-Dimensional Visualization and Animation

## Mars Rover Mission Game

**Fig 1. General Overview of the Prototype**  
**G08**  
Filipa Cotrim 95572  
Filipe Henriques 95573  
Tiago Lopes 95679  

### Game Instructions

The Mars Rover Mission Game consists of a modelled rover that can be manually driven, completing missions. The general scene contains static objects (cones and cubes in the first prototype, and R2D2’s in the final one), as well as rolling balls. The game can be paused by pressing the “S” key and, if the user loses the game, it can be restarted by pressing the “R” key. The player starts the game with 3 lives and increases points the further it drives. If the rover collides with any of the static objects, its movement stops. On the other hand, if it hits a ball, it is set back to its initial position, also losing a life. The balls roll randomly across the Mars ground, bouncing back when colliding with static objects.

To drive the rover, the user must press:
- “Q” to move in an accelerated movement
- “A” to move in a deaccelerated movement (eventually moving backwards)
- “P” and “O” to change the direction angle either to the right or left, respectively.

Regarding the scenery of the game, the user can make changes to the overall lighting by enabling and disabling spotlights (“H” key), the directional light (“N” key), and point lights (“C” key). Additionally, the “F” and “D” keys enable and disable fog, and the “L” key enables the lens flare effect. In terms of cameras, the user can change the main ones by pressing “1”, “2”, or “3”, or choose the rearview by pressing “V”.

### Tasks Implementation – 1st Assignment

#### Graphic Modelling
At this stage of development, all the objects were rendered using simple three-dimensional geometric objects utilizing the `basic_geometry.cpp` library available, with the rover being the most complex one – composed of 10 cylinders and 2 cubes. Meshes were kept in different vector structs, and when it came to the drawing portion, all objects were scaled, translated, and/or rotated using `push/popMatrix`, keeping in mind that OpenGL matrices are multiplied to the right of the existing matrix (vectors are in column-major order).

#### Cameras
For the cameras, we created an array of a specific struct that holds both their position, target, and type (perspective and orthogonal). The first two cameras are static cameras with the same values but with different types, providing a top view of the space. The third camera follows the rover to give a third-person view. For this, we recalculate both the camera’s position and target based on the rover’s position and direction.

#### Game Elements’ Movements
When implementing each object’s movements, we tried to make it as realistic as possible. We altered the `myMesh` struct attributes (in the `geometry.h` file), adding the moving direction, velocity, position, and angle. This way, the distance traveled by the rover is based on its velocity at any given moment, with the velocity increasing as the game progresses, and the new position is calculated based on the current rover direction. The rolling balls' movement was implemented similarly to the rover’s, with the particularity that when it hits a static object, we normalize the new direction vector.

#### Lighting of the Scene
To illuminate the scene, we created all 3 kinds of lights: 1 directional light, 2 spotlights functioning as the headlights of the rover, and 6 point lights scattered across the map. For them to work, we had to pass multiple arguments to the shaders, which was done by implementing structs with arguments varying depending on the type of light (e.g., the spotlight struct had its cone direction added). In the shaders, the new `colorOut` was calculated using a Modified Blinn-Phong Shading Model, which is basically a sum of the model for each light.

#### Collision Detection
The best way to evaluate if two objects collided is by implementing the AABB detection method. Each mesh now has yet another attribute, the Bounding Box – a struct containing the edges’ values of the object, like it was bounded by an invisible rectangle. Even though the box has no rotation, for each calculation of collision, we calculate the edges’ new positions based on the rover angle. This issue only pertains to the rover since it is the only object whose rotation affects the values of the box. This method detects a collision when two bounding boxes enter each other's regions, more specifically, if the shape that determines the first object is in some way inside the shape of the second object.

#### Transparency
Transparency, as well as other features mentioned next, was implemented through a blending technique. Thus, the need to enable it (`glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA); glEnable(GL_BLEND);`). With this, desired objects instead of having a solid color, have a color combination of their own colors with the ones from the objects behind them. This was implemented in the fragment shader (`colorOut = vec4(max(intensity*texel.rgb + spec, 0.1*texel.rgb), texel.a)`). The amount of transparency of the objects is defined by the alpha value (with 0.0 meaning fully transparent and 1.0 meaning an opaque object). We also made sure the depth buffer was read-only (by calling `glDepthMask(GL_FALSE)`) so that only opaque values are being stored.

#### Fog Effect
For fog, the function chosen for better results was the exponential one (in the fragment shader), dependent on the distance of the camera to the point. Still in the fragment shader, we use the “mix” function as a means to interpolate the final color between fog color and the fragment color using the exponential fog density.

### Tasks Implementation – 2nd Assignment

#### Billboards
The static textured rectangles are used to replace meshes as a fast way of rendering. For the billboard background to be transparent, we ensure to discard the fragments whose alpha values are equal to 0 while blending all other. This, of course, requires enabling the blending function and the alpha channel. As for the billboard rotation according to the camera position (more precisely, the rover position), we made use of the `l3dBillboard.cpp` library provided.

#### Camera Rear View
This task was the most challenging one. We managed to implement a well-functioning rear-view camera initially, but when adding the shadows and reflections feature, the stencil buffer wasn’t working properly. We tried using two different render scenes, one where the stencil draws in the active camera and another where it draws on the rear view, using separate masks. However, we did not get the desired result. Therefore, we adapted this camera and instead of using a quad displayed simultaneously as the other camera, we displayed it as an extra camera view (by pressing the “V” key).

#### Particles
The particles implemented simulate and give some impact to the collision with the rolling balls. When that happens, many small quads are created at the place of collision. They simulate different trajectories for each particle while also giving the impression of being affected by wind and gravity.

#### Lens Flare
The lens flare effect is only enabled if the spotlight and directional light are turned off for a more realistic view. For this technique, we followed the example provided by the teacher, implementing the rendering of the lens elements in a separate function. Firstly, we turned off the depth-test so that the flare would be represented in front of the scene. Then, since we had the projected position of light on the viewport, we wanted to position the elements along a line from this position to across the center of the screen. We wanted the elements to appear bigger and more opaque the nearer they were to the center, and smaller and more translucent the farther they were. To do so, we computed how far off-center the flare source was, the destination, the position interpolated along the line, and the width of the screen (for the elements to be proportioned).

#### Shadows and Reflections
For the implementation of planar shadows, we made the floor semi-transparent to work as a mirror (an ideal reflective planar surface). We took into consideration that the shadows and reflections should only be seen if the camera is positioned above the ground. We enabled stencil testing and wrote 1 where the reflection and shadow are to be drawn and proceeded to draw the quad/mirror, never rendering it into the color buffer. Then, we rendered the reflected geometry, mirroring the position of light. For the rendering of shadows, we had to enable the blending function and disable the depth test to force the shadow geometry to be rendered behind the floor, darkening the color stored in the color buffer.

#### Cube Mapping/Sky Box
To implement the skybox, we used cube mapping. The cube encompasses the scene while providing the illusion of a faraway 3D background behind the scenery. We made sure to turn off depth masking so that the box is behind all the other elements, and we also discarded the translation component of the camera in the view matrix.

#### Environment Mapping
In this task, we implemented a floating sphere that acts like a mirror of the environment, but due to the lack of lighting, the sphere might appear black from some angles or because of the chosen skybox. However, the reflection is there and it moves as you move around it.

#### Assimp Objects
To make all objects more visually pleasing, instead of using the previously implemented 3D geometries, we imported the desired models into the application. This method allowed us to use more interesting models that we wouldn’t have been able to define manually, such as normals, vertices, or texture coordinates. The Assimp Model library “Assimp” loads all the model’s data into data structures we can use to extract their geometries, and subsequently, the textures. This model was then used to create the relevant transformations (scaling, rotation, translation) for its proper placement.  

### Conclusion
The Mars Rover Mission Game’s progression has seen great advances from its original prototype to the current state with a variety of extra features, from realistic graphics and effects (fog, lens flares, particles) to more engaging user input mechanics, providing both excitement and challenging gameplay.

# Tasks Implementation – 2nd Assignment

## Billboards
Billboards, implemented as static textured rectangles, are used to replace meshes for efficient rendering. To achieve transparency in the billboard background, fragments with an alpha value of 0 are discarded in the fragment shader, while others are blended. This requires enabling both the blending function and the alpha channel. The billboard's rotation, aligned with the camera position (specifically, the rover’s position), is handled through the `l3dBillboard.cpp` library.

## Camera Rear View
This task was particularly challenging. Initially, we implemented a functional rear-view camera, but when shadows and reflections were added, the stencil buffer did not work as expected. We attempted to use two render scenes—one for the active camera and another for the rear view—each with separate masks. However, this approach did not yield the desired results. Ultimately, we adapted the system, replacing the quad display with an additional camera view, which could be toggled by pressing the “V” key.

## Particles
Particles simulate the impact of collisions with the RollingBalls. Upon a collision, small quads are generated at the collision point, each following distinct trajectories influenced by wind and gravity.

## Lens Flare
The lens flare effect is activated when both the spotlight and directional light are turned off for realism. The flare elements are rendered in a separate function. To ensure proper positioning, we compute the distance from the flare’s light source to the center of the screen, with the elements appearing larger and more opaque closer to the center, and smaller and more translucent farther away. This is achieved by interpolating along a line from the light source to the center and adjusting the element sizes based on this distance.

## Shadows and Reflections
For planar shadows, we set the floor to be semi-transparent, acting as an ideal reflective surface. We ensure the reflections and shadows are visible only when the camera is positioned above the ground. Stencil testing is enabled to mark areas for reflection and shadow, and the mirror quad is drawn without rendering to the color buffer. Reflections are rendered by mirroring the geometry relative to the light's position. To draw shadows, we disable the depth test and enable blending to ensure shadows appear behind the floor and darken the color buffer.

## Cube Mapping/Sky Box
The skybox is implemented using cube mapping, where a cube surrounds the scene, creating the illusion of a distant 3D background. Depth masking is turned off to ensure the skybox stays behind all other elements. Additionally, we discard the camera’s translation component in the view matrix.

## Environment Mapping
A floating sphere is implemented to reflect the environment, though due to a lack of proper lighting, it may appear black from certain angles or due to the chosen skybox. Despite this, the reflection remains accurate and moves as the camera changes position.

## Assimp Objects
To enhance the visual quality of objects, we transitioned from using manually defined 3D geometries to importing models via the Assimp library. This allowed for the use of more detailed models, with Assimp handling normals, vertices, and texture coordinates. The library loads model data into structures and recursively renders nodes containing indices to data stored in a scene object. A global Assimp scene is created for each imported object, along with an importer instance for each model. A flag is passed through all relevant functions to identify the object being rendered. These improvements significantly enhanced the game’s visual appeal.
