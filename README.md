# Terra Toy Prop Tool
This is the official documentation for the custom prop creation tools for Terra Toy. In this repository, you will find documentation on [PropScript](https://github.com/frozein/PropScript) (the programming language used to create props), examples of PropScript files, as well as an executable for packaging the source code into a format readable by the game. Please note that Terra Toy as well as PropScript are both in a beta state, so you may encounter bugs or unintuitive behavior, as well as a lack of features.

## Basic PropScript Documentation
PropScript has an overall syntax somewhat similar to Python's (so set your text editor's syntax highlighting to Python), with the exception that code blocks must be contained in curly braces `{}`. PropScript files have the extension `.ps`.
### Basics
- A single line (ended with a newline) is a single statement, there are no semicolons. If you wish to have a statement extend more than one line, enclose it in parenthesis `()`.
- Variables are defined and assigned as follows: "`myVar = statement`", where the type of "`statment`" determines the variable's type, you cannot declare a variable without assigning it a value.
- Functions are called as follows: "`my_function(param1, param2, ...)`"
- Single-line comments are prefixed  with a `#`, there are no multi-line comments
### Control Flow
```python
# If/Else Statements:
if statement
{
	...
}
else if statement
{
	...
}
else
{
	...
}

# The brackets can be omitted if the code inside is only a single statement:
if statement
	...
```
```python
# For loops:
for varName in range(minVal, maxVal)
{
	...

	break # exits out of the loop
	continue # skips the current iteration of the loop
}

# The brackets can be omitted if the code inside is only a single statement:
for varName in range(minVal, maxVal)
	...
```
```python
# Function definition:
func my_function(paramName1, paramName2, ...)
{
	...

	ret returnVal # returning a value is optional, "ret" with no value will exit the function
}
```

### Standard Types
- `int, float`
- `vec2, vec3, vec4`, accessing individual components is done with square brackets `[]`. The vectors are 0-indexed. For example, accessing the 2nd element of a vector type would be: "`myVector[1]`"
- `quaternion`, accessing individual components of a quaternion is done in the same manner as vectors

### Standard Operations/Functions
- `*, /, %, +, -, =, *=, /=, %=, +=, -=, <, >, <=, >=, ==, !=, and, or`
- `rand(min, max)`, returns a random value between `min` and `max`, works for every type except quaternions
- `int()` casts a float to an int by truncating
- `vec2(...)`, `vec3(...)`, `vec4(...)`, creates a vector. If given no parameters, all components of will be 0. If given 1 parameter, all components will be the given parameter. If given n parametrs, the components will be initialized to the parameters given.
- `quaternion(...)`, creates a quaternion. If given no parameters, it will be the identity quaternion. If given a single `vec3` parameter, the quaternion will correspond to a rotation by the euler angles in the parameter. If given a `vec3`, then a scalar parameter, the quaternion will corresond to a rotation about an axis specified by the `vec3` by an angle specified by the scalar
- `sqrt()`, `pow()`, `abs()`, compute their respective math operations, only work with single scalar parameters
- `sin()`, `cos()`, `tan()`, `asin()`, `acos()`, `atan()`, compute their respective trig operations, only work with single scalar parameters.

### Standard Constants
- `M_PI` = 3.141596...
- `M_TAU` = 6.283185...
- `M_E` = 2.718281...

## PropScript for Terra Toy
On top of the standard PropScript functions and constants, Terra Toy supplies some of its own for interfacing with the game.

### Anatomy of a Shape Function
Props in Terra Toy are made up of geometric shapes (in the future, it will have support for custom shapes defined by SDFs as well). These shapes are placed into the world via functions in PropScript. The basic structure of a shape function is as follows:
```
shape(material, color, position, [shape params], transformFunction, [transform function params])
```
- `material` is the material the that the shape will take on, the available materials are listed later
- `color` is the rgb albedo color of the shape, the values are `vec3`s with each component ranging from `0-255`
- `position` is the 3D position the shape will be placed at in local space (unless the world position tag is set)
- `[shape params]` are parameters specific to the shape function, listed later
- `transformFunction` is NOT a function pointer, it is an integer. More information on transform functions as well as the available ones is written later.
- `[transfom function params]` are parameters specific to to the specified transform function

An example of a full shape placement would look like this:
```
sphere(MAT_MATTE, vec3(255 0, 0), vec3(1.0, 0.0, 3.0), 10.0, TF_LEAVES, vec3(0), vec3(255))
```

### Available Shape Functions
- `sphere`, with parameter `radius (float)`
- `box`, with parameters `length (vec3)` and `orient (quaternion)`. NOTE: unintuitively, `length` represents half the volume the box takes up. For example, if `length` is `{2, 3, 4}`, the box will occupy an area of 4x6x8. (This was an error that went unfixed for very long, and now all of the existing props are written with this assumption so it will not be changed).
- `rounded_box`, with parameres `length (vec3)`, `radius (float)`, and `orient (quaternion)`. NOTE: `radius` is added to `length`, so the box may be larger than expected
- `torus`, with parameters `majorRadius (float)`, `minorRadius (float)`, and `orient (quaternion)`
- `ellipsoid`, with parameters `radii (vec3)`, and `orient (quaternion)`
- `cylinder`, with parameters `radius (float)`, `height (float)`, and `orient (quaternion)`. NOTE: the cylinder will be placed at the geometric center of the volume, NOT the base. Similarly to the issue with `box`, `height` actually represents half the height.
- `cone`, with parameters `radius (float)` and `height (float)`
- `pyramid`, with parameters `length (vec3)` and `orient (quaternion)`
- `triangle` with parameters `vertex a (vec3)`, `vertex b (vec3)`, and `vertex c (vec3)`
- `quad` with parameters `vertex a (vec3)`, `vertex b (vec3)`, `vertex c (vec3)`, and `vertex d (vec3)`. NOTE: all the points must be coplanar or you may get strange results

### Available Materials
- `MAT_MATTE`: basic matte, diffuse
- `MAT_REFLECTIVE`: mirror-like reflective
- `MAT_EMISSIVE`: has no shading, gives of light when fancy lighting is enabled
- `MAT_TRANSPARENT`: glass-like, opacity of 0.5
- `MAT_LEAVES`: same as `MAT_MATTE`, but spawns leaf particles on each voxel
- `MAT_EMPTY`: air material, can be used to remove portions of the prop or of terrain
- `MAT_BIOME`: set to the material of the player's selected biome, used for custom terraforming tools

### Player-Chosen Colors
If the prop is tagged as player-colorable (explained later), the selected color can be accessed with the `USER_COLOR` constant.

### Transform Functions
Transform functions are functions run per-voxel when the shapes are voxelized into the world, and are used to modify smaller details that occur on a per-voxel scale. There aren't very many available currently, more will come as release gets closer.
- `TF_NONE`, with no parameters. No transform function, all voxels will be the same
- `TF_LEAVES`, with parameters `color1 (vec3)` and `color2 (vec3)`. Each voxel's color is interpolated randomly between `color1` and `color2`. It has nothing to do with leaves or leaf particles, the name is historic.
- `TF_FRUIT`, with parameters `color1 (vec3)`, `color2 (vec3)`, `fruitColor (vec3)`, and `fruitChance (float)`. Similar to `TF_LEAVES`, with each voxel additionally having a random chance, defined by `fruitChance`, to have its color set to `fruitColor`.
- `TF_UMBRELLA`, with paramters `numSpokes (int)`, `color1 (vec3)`, and `color2 (vec3)`. Creates a radial pinwheel pattern with `numSpokes` sections, alternating between `color1` and `color2`.
- `TF_STONE`, with parameters `color1 (vec3)`, `color2 (vec3)`, and `removeChance (float)`. Similar to `TF_LEAVES`, with each voxel additionally having a random chance, defined by `removeChance`, to be empty
- `TF_CACTUS` and `TF_IGLOO`, with no paramters. Used specifically for the igloo and cactus props.

### Tags
The last component of PropScript files for Terra Toy are tags. Tags are prefixed with an `@` and must come at the top of the file. Some may have parameters. The available tags are:
- `@category [int]`, the category the prop belongs to. 0 = tree, 1 = foliage, 2 = misc, 3 = shape.
- `@placesound [int]`, the sound to play with the prop is placed. 0 = axe/wood, 1 = leaf, 2 = plink, 3 = snow, 4 = rock, 5 = cloth.
- `@rotateable`, specifies that the player sould be able to rotate this prop.
- `@colorable [vec3]`, specifies that the player should be able to edit this prop's color. The parameter represents the default color.
- `@useworldspace`, specifies that the all positions and orientations given are in world space, not local space, and should not be transformed based on where the player clicked.
- `@scale [float]`, specifies an overall scale for the object. Useful if you make the prop larger than desired for ease of development, then want to scale it down when finally in-game.

## Adding a Custom Prop
Once you have created your PropScript file (see the examples if you are still stuck), you must run the proptool executable to package it into a `.prop` file used by the game. Do note that each prop additionally requires an icon image. Simply follow the instructions in the executable, and you will have created your `.prop`. Simply place it into the `assets/props/custom/` directory in the game's local files, run the game, and your prop will appear in the list.

Any parsing errors will be reported by proptool, and any runtime errors will be reported in the `properror.txt` file in the game's local files. (you only need to check this if your prop is not working as expected).

In order to reload the prop without closing and reopening the game, simply select the prop in the menu, and middle click on your mouse. A typical workflow would look like: `edit propscript file -> recreate .prop file (enter "c" in proptool) -> reload prop in game (middle click)`.
