Intro to terrain creation
============



Creating a terrain for Rigs of Rods is quite technically involved and largely manual process. This page aims to give you an overview of required work, as well as guides and advice for your own projects.

## Anatomy of terrain

Before you start making your own terrain, you need to know and understand what does a terrain consist of and how it all works together.

### Ground

Obviously the most significant part of the terrain. Ground provides a surface for land vehicles to drive on, airplanes to take off from/land on etc. Ground has shape, physical properties like friction and visual attributes.

In RoR, the primary method of defining ground shape is using a heightmap. A heightmap is a grayscale image where black represents lowest areas and white represents highest areas. A visual 3d ground is generated by RoR when map is loaded.

Physics properties are defined using a configuration file which defines surface types, as well as a set of images which define areas of effect for the surface types.

Visual appearance is achieved using tiling textures, texture blending (a.k.a splatting) and built-in shader effects (normal mapping, specular mapping).

An alternative approach to creating ground is to implement it as static object. In this case, none of the above applies. The shape would be defined by the mesh (3d model) geometry. Physics property setup is different. Visual appearance is specified by material script.

### Water

Another typical part of a landscape. Bodies of water can vary in size from small ponds to lakes and seas.

RoR implements water as a single body covering the whole map and penetrating all objects. It's specified as "water height" attribute of the map. All areas below the specified height will be flooded.

### Static objects

Anything that doesn't move. Mostly buildings of all kinds, but also roads, structures like bridges, and finally roads and other tracks.

Each of these objects must have a visual mesh (3d model). RoR uses OGRE's ".mesh" format.

Static objects may also have a collision mesh and associated friction settings, which are defined in a `.odef` file. Those which don't are ethereal - vehicles just drive through them.

Static object may also trigger special events. A "truck shop" building will, when entered, will display a vehicle menu. A "load spawner" deck will bring up a menu with trailers and loads. And so on.

### Stationary dynamic objects

Stationary or semi-stationary objects like cranes, hall doors, bridges or wind turbines are made from dynamic physical model, just like vehicles, except they're completely or partially fixed to the ground.

## The basic structure

Terrains are distributed as ZIP archives containing a set of terrain-definition files and various resources. A terrain _must_ be in a ZIP archive, otherwise it doesn't load correctly.

### The files
* **.terrn2**: Entry point for the terrain. This is the only required file for a terrain to have.

```
[General]
#Name of the terrain
Name = My Terrain
#File name of the heightmap configuration
GeometryConfig = my-terrain.otc
#If your map needs water, set this value to 1
Water=0
#Water level
WaterLine=1.0
#Terrain/water color, usually left at the default values.
AmbientColor = 0.93, 0.86, 0.76
#The position RoRBot spawns at when the terrain is loaded.
#The numbers do not have to be seperated by commas.
StartPosition = 1364.25 121.028  1272.22
#Where the map will be located in the terrain selector menu. Best to leave it as the default.
CategoryID = 129
#Version of the terrain.
Version = 1
#Unique ID of the terrain. You can generate one at https://guidgenerator.com
GUID = 9b202f78-ba1c-4d58-9996-61066fa5d9fc

#Extras

#Landuse config, configures surface types (dirt,mud,etc)
TractionMap = landuse.cfg

#Caelum sky configuration file.
CaelumConfigFile = my-sky.os

#Sandstorm sky cubemap material.
SandStormCubeMap = tracks/skyboxcol

#Authors of the terrain.
[Authors]
terrain = yourname

#Filename for the object (.tobj) file. 
[Objects]
my-terrain.tobj=

#Filename for the Angelscript file.
[Scripts]
my-script.as=
```

* **.otc**: "Ogre Terrain Config". Configures the OGRE::Terrain subsystem which RoR uses for terrain display. See below for example files.
* **-page-x-x.otc**: Configures the terrain's ground textures. Most terrains only have 1 page, so this file is usually named `mapname-page-0-0.otc`. See below for a example file.
* **.tobj**: Placements for static objects/trees/grass/etc. See the [Static objects](#static-objects-1) section for more info.
* **.os**: Caelum system (sky/weather) config. Visuals only. [Example file here.](https://github.com/RigsOfRods/rigs-of-rods/blob/master/resources/caelum/RoRSkies.os)
* **.hdx**: Hydrax config (0.4.5 and up). Water display. [Example file here.](https://github.com/RigsOfRods/rigs-of-rods/blob/master/resources/hydrax/HydraxDefault.hdx)
* **.as**: Terrain script file, usually used for races. See [this page](http://docs.rigsofrods.org/terrain-creation/scripting/) for more info.

Note that, there is no editor which would create these files for you. You need to copy a template and work manually from there.

Templates can be found [here](https://forum.rigsofrods.org/resources/template-raw-png-terrains.262/).

### The heightmap

As said above, RoR's primary mechanism for shaping a terrain are heightmaps. We use 8-bit or 16-bit unsigned integer RAW heightmaps. As of 0.4.x, using a PNG image for the heightmap is also supported.

If you want to use RAW, They can be converted from images or generated in a specialized tool. We provide tutorials for several of them.

To use the heightmap in a terrain, you must configure it in a `.otc` and `-page-x-x.otc`file (filename, size, bit depth...) and include it into terrain's ZIP archive.

You can use [ImageMagick](http://www.imagemagick.org/script/index.php) to convert to and from the .raw heightmap files:

- To convert from a .raw file to a bitmap image file issue the following command:

```
convert -depth 16 -size 1025x1025 -endian LSB gray:mymap.raw mymap.bmp
```

- To convert back to a .raw file for RoR execute: 

```
convert mymap.bmp -resize 1025x1025 -endian LSB -flip gray:mymap.raw
```

- For more information, see ImageMagick's [command-line processing](http://www.imagemagick.org/script/command-line-processing.php) page. 

The layout of an `.otc` is different depending on which format your heightmap is. 

An example `.otc` file for a RAW heightmap:
```
;Heightmap values
;size (horizontal/vertical)
Heightmap.0.0.raw.size=1025
;bytes per pixel (1 = 8bit, 2=16bit)
Heightmap.0.0.raw.bpp=2
;If the terrain heightmap needs to be flipped (eg. Terragen exports RAW upside down)
Heightmap.0.0.flipX=1

;Terrain size values
;size (both values must be the same for textures to work properly!)
WorldSizeX=3000
WorldSizeZ=3000
;Terrain max height
WorldSizeY=300

;To disable caching (creating a `.mapbin` file in `/cache` folder) when changing terrain textures. 
disableCaching=1

;Filename to define the textures.
PageFileFormat=my-terrain-page-0-0.otc

;Advanced texture values, best to leave them as the defaults.
LightmapEnabled=0
SpecularMappingEnabled=1
NormalMappingEnabled=1
```

Example for a PNG heightmap:
```
## the amount of pages in this terrain
## 0, 0 means that you only have one page
PagesX=0
PagesZ=0

PageFileFormat=my-terrain-page-{X}-{Z}.otc

## the factor with what the heightmap values get multiplied with
WorldSizeY=0

## The world size of the terrain
WorldSizeX=3000
WorldSizeZ=3000

## Sets the default size of blend maps for a new terrain. This is the resolution of each blending layer for a new terrain. default: 1024
LayerBlendMapSize=2048

## disableCaching=1 will always enforce regeneration of the terrain, useful if you want to change the terrain config (.otc) and test it. Does not cache the objects on it.
disableCaching=1

#optimizations

## Minimum batch size (along one edge) in vertices; must be 2^n+1. The terrain will be divided into tiles, and this is the minimum size of one tile in vertices (at any LOD). default: 17
minBatchSize=17

## Maximum batch size (along one edge) in vertices; must be 2^n+1 and <= 65. The terrain will be divided into hierarchical tiles, and this is the maximum size of one tile in vertices (at any LOD). default: 65
maxBatchSize=65

## Whether to support a light map over the terrain in the shader, if it's present (default true).
LightmapEnabled=1

## Whether to support normal mapping per layer in the shader (default true). 
NormalMappingEnabled=1

## Whether to support specular mapping per layer in the shader (default true). 
SpecularMappingEnabled=1

## Whether to support parallax mapping per layer in the shader (default true). 
ParallaxMappingEnabled=0

## Whether to support a global colour map over the terrain in the shader, if it's present (default true). 
GlobalColourMapEnabled=0

## Whether to use depth shadows (default false). 
ReceiveDynamicShadowsDepth=0

## Sets the default size of composite maps for a new terrain, default: 1024
CompositeMapSize=1024

## Set the distance at which to start using a composite map if present, default: 4000
CompositeMapDistance=5000

## the default size of 'skirts' used to hide terrain cracks, default: 30
SkirtSize=30

##  Sets the default size of lightmaps for a new terrain, default: 1024
LightMapSize=1024

## Whether the terrain will be able to cast shadows, default: 0
CastsDynamicShadows=0

## Set the maximum screen pixel error that should  be allowed when rendering, default:
MaxPixelError=0

## dump the blend maps to files named blendmap_layer_X.png
DebugBlendMaps=0
```




### Ground textures

See also: [Ogre Terrain Page Config (.otc)](/terrain-creation/terrn2-subsystem/#ogre-terrain-page-config-otc)

The system is designed for texturing by several tiling textures combined via built-in texture blending (aka texture splatting).
Other built-in effects: [normal mapping](http://wiki.polycount.com/wiki/Normal_map), specular mapping, [parallax mapping](http://wiki.polycount.com/wiki/Parallax_Map).

Terrain supports "Global Light maps" (Mountains will shadow the terrain if the sun is low, etc...), but this feature is not currently used by RoR.

A standard (single layer) `-page-x-x.otc` file:

```
mapname.raw = Heightmap filename 
1 = Amount of layers
worldSize = X/Z value in the .otc [Has to be the same]
mapname.dds = texture name
```
```
mapname.raw
1
; worldSize, diffusespecular, normalheight, blendmap, blendmapmode, alpha
3000     , mapname-diffspec.dds     ,   mapname-normal.dds

```

`-page-x-x.otc` file with multiple layers: (Limited to 6 texture layers)

```
mapname.raw
4
; worldSize, diffusespecular, normalheight, blendmap, blendmapmode, alpha
3500     , mapname-ds.dds      ,   mapname-ds.dds
5, mapname-gravel_diffusespecular.dds,mapname-gravel_normalheight.dds,mapname_RGB.png, R, 0.9
5, mapname-lushGrass2-ds.dds,mapname_lushGrass_NRM-ds.dds,mapname_RGB.png, B, 1.0
5, mapname_Cracked2_DS.dds,mapname_Cracked_NM.dds,mapname_RGB.png, G, 1.0
```

## Static objects

All static objects/grass/etc on a terrain are defined in a `.tobj` file, multiple of these files can be defined in the `.terrn2`.

A normal terrain object is usually formatted like this:

```
// x        y        z    rx  ry rz odefname (without .odef file extension)
875.549, 67.6607, 1155.26, 0, 0, 0, truckshop
```

RoR uses Paged Geometry for trees/grass, see [this page](http://docs.rigsofrods.org/terrain-creation/old-terrn-subsystem/#grass) for more info.

For help editing terrain objects, see: [Adding/moving terrain objects](/terrain-creation/editing-terrain-objects/)

A `.tobj` file featuring trees, grass, objects, and roads:

```
//grass and trees
grass 200, 0.5, 0.05, 10, 0.1, 0.2, 0.2, 1, 1, 1, 0, 9, seaweed none none
grass 200, 0.5, 0.05, 10, 0.3, 0.2, 0.2, 1, 1, 1, 10, 0, grass1 aspen.jpg aspen_grass_density.png
trees 0, 360, 0.1, 0.12, 2, 60, 3000, fir05_30.mesh aspen-test.dds aspen_grass_density2.png 

//trucks/loads/boats
1191.162109, 35.034180, 930.908203, 0.000000, 0.000000, 0.000000, truck wrecker.truck
1131.103516, 35.034180, 910.888672, 0.000000, 0.000000, 0.000000, load acontainer.load
1221.191406, 35.034180, 930.908203, 0.000000, 0.000000, 0.000000, load crate.load
1185.191406, 35.034180, 827.042773, 0.000000, -144.000000, 0.000000, truck an-12.airplane
1091.064453, 37.036133, 1224.194336, 0.000000, 0.000000, 0.000000, truck dodgepolice.truck
332.380859, 33.032227, 1009.191406, 0.000000, 23.000000, 0.000000, boat smit.boat

//objects
1181.152344, 34.183350, 950.927734, 0.000000, 0.000000, 0.000000, truckshop sale RigaDeal
1181.152344, 34.183350, 910.888672, 0.000000, 0.000000, 0.000000, myhangar2 repair Rods
1133.914917, 34.134541, 945.474792, 0.000000, 90.000000, 0.000000, load-spawner sale ColdDepot
1151.123047, 33.833008, 995.971680, 0.000000, 0.000000, 0.000000, a1da0UID-smallhouse
1131.103516, 33.833008, 995.971680, 0.000000, 90.000000, 0.000000, a1da0UID-smallhouse
1231.201172, 33.833008, 940.917969, 0.000000, 0.000000, 0.000000, a1da0UID-myobs
//1141.113281, 33.833008, 1020.996094, 0.000000, 45.000000, 0.000000, a1da0UID-smallhouse2

//some roads
1026.371338, 34.086056, 1007.492920, 0.000000, -144.000000, 0.000000, road
1018.273438, 34.086056, 1013.378662, 0.000000, -144.000000, 0.000000, road
1010.175537, 34.086056, 1019.254395, 0.000000, -144.000000, 0.500000, road
1002.077637, 34.173340, 1025.140137, 0.000000, -144.000000, 0.000000, road
994.111877, 34.173340, 1030.935791, 0.000000, -146.000000, 0.500000, road
985.945923, 34.260723, 1036.441162, 0.000000, -148.000000, 0.000000, road
977.557739, 34.260723, 1041.686279, 0.000000, -149.500000, 0.000000, road
969.035400, 34.260723, 1046.711182, 0.000000, -151.000000, 0.500000, road
960.350952, 34.348011, 1051.525879, 0.000000, -152.000000, 0.000000, road
951.652466, 34.348011, 1056.150391, 0.000000, -154.000000, -0.500000, road
942.834839, 34.260723, 1060.464600, 0.000000, -156.500000, 0.000000, road
933.910156, 34.260723, 1064.348389, 0.000000, -160.000000, 1.000000, road
924.580017, 34.435394, 1067.741699, 0.000000, -161.000000, 1.000000, road
```
