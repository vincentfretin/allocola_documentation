# Documentation of the elements.json file

## Configure auto formatting of the json file

Use [vscode](https://code.visualstudio.com/) with the prettier extension for code formatting and configure auto format on save:

- `ctrl+shift+p`, search "install" and select "install extensions", search "prettier", click on "Prettier" and click "Install" button
- Configure format on save go to `File->Preferences->Settings->Text editor->Formatting` and check "Format On Save"
- Open a json file, `ctrl+shift+p`, search "format", select "Format File", select the prettier formatter

From now on, each time you save, it will auto format with prettier.

## General structure of the elements.json file

### Minimal json

You need at least

```json
{
  "sceneTitle": "Duc's sandbox 1",
  "sceneModel": "Health_room_01.glb",
}
```

`"sceneModel"` can be empty string `""`. If you don't use a scene model, you can define some components at the scene level (not in children), like the `environment` component, this way you can add some lights in the scene:

```json
  "components": {
    "environment": "playArea: 400; preset: arches; dressing: none; ground: none; shadow: false; fog: 0"
  },
```

You don't need the `environment` component if you create a scene in blender and you defined an envmap, lightmaps and reflection probes with the hubs components in blender.

You can configure the created hemisphere light intensity that is created with the `environment` component:

```json
  "components": {
    "environment": "playArea:400;preset:default;dressing:none;shadow:false;fog:0"
  },
  "overrides": {
    "hemisphereLightIntensity": 2.0
  },
```

To add entities, you add them in `"children"`:

```json
{
  "sceneTitle": "Duc's sandbox 1",
  "sceneModel": "Health_room_01.glb",
  "children": [
    {
      your entity here
    }
  ]
}
```

An entity is defined by `"element": "a-entity"` (the default), `"components"` object structure where the key is a component name and `"children"` that is an array of entities.

Other keys may also be used, currently: `"id"`, `"class"`, `"comment"`, `"enabled"`.
Here is a full example of what an entity looks like:

### Mirror with black border

```json
{
  "enabled": true,
  "element": "a-mirror",
  "comment": "Vertical mirror showing my avatar",
  "components": {
    "mirror": "layers: 0,3",
    "position": "-6.52 1.8 12.02",
    "scale": "3 2 1",
    "rotation": "0 90 0"
  },
  "children": [
    {
      "components": {
        "geometry": "primitive: box",
        "material": "color: black",
        "position": "0 0 -0.02",
        "scale": "1.02 1.02 0.01"
      }
    }
  ]
},
```

Note: layer 0 is all entities, layer 3 is your own avatar.

### Comment

You can't add comment with `//` in a json file, but you can use a comment key, this key is just ignored when creating the entities in the room, example:

```json
  {
    "comment": "Use an horizontal mirror for water",
    "class": "navmesh-hole",
    "element": "a-mirror",
    "components": {
      "mirror": "layers: 0",
      "position": "-0.041 0.23 6.64",
      "scale": "4.84 11.2 1",
      "rotation": "-90 0 0"
    }
  },
```

### Temporarily disable an entity

If you want to comment, meaning deactivate a whole set of line you can use `"enabled": false`

`"enabled"` key is optional, default to true, you can put it to false if you don't want it in the room temporarily.

### Group comment

You can combine the two keys `"comment"` and `"enabled"` to make a group comment if you want...

```json
{
  "enabled": false,
  "comment": "PLANTS",
},
```

## Environment components

### environment

You can use the `environment` component, see the [documentation](https://github.com/supermedium/aframe-environment-component).
We added some overrides to specify light intensity of the hemisphere light and directional light (sun) created by the `environment` component. The `sunLightPosition` for `environment` component needs to be a normalized `Vector3` because it uses the sun position to calculate the fog distance, but sometimes it means the light is inside a model, so you may want to move it up, especially for shadows. Specifying it in "overrides" will make sure the sun light position is set after the environment component finished calculating fog distance based on the default or specified `sunLightPosition` of the environment component.
You can also tune [shadowBias](https://aframe.io/docs/master/components/light.html#configuring_shadows_shadowbias) option if you see shadows are not crisp.

```json
  "components": {
    "environment": "playArea:400;preset:arches;dressing:none;shadow:true;shadowSize:120;fog:0.4"
  },
  "overrides": {
    "hemisphereLightIntensity": 1.5,
    "sunLightPosition": "-22 32 66",
    "sunLightIntensity": 3.0,
    "shadowBias": 0.004
  },
```

To dynamically calculate an envmap of the scene glb with the directional light created from the environment component, you can set reflection to true, this will use the aframe `reflection` component at the right time.

```json
  "overrides": {
    "reflection": true
  }
```

### environment-settings

If you defined it in blender via the hubs addon, you don't need to use it in the json.
Here are the default values:

```json
"components": {
  "environment-settings": {
    "toneMapping": "LUTToneMapping",
    "toneMappingExposure": 1,
    "backgroundColor": "skyblue",
    "backgroundTexture": "",
    "envMapTexture": ""
  }
}
```
The `backgroundTexture` and the `envMapTexture` support both hdr and ldr (jpeg, webp).
The `toneMapping` is one of `"NoToneMapping", "LinearToneMapping", "ReinhardToneMapping", "CineonToneMapping", "ACESFilmicToneMapping", "CustomToneMapping", "LUTToneMapping"`

Example of sky with clouds:

```json
    "backgroundTexture": "../kloofendal_48d_partly_cloudy_puresky_1k.hdr",
```

Note that specifying `backgroundTexture` this is not compatible with using the `environment` component. You will need to disable the sky from the `environment` component like this: `"environment": "skyType:none"`.

## Adding models

### Adding a model inside a mediaframe

A mediaframe here in the a mediaframe created in the scene glb with the hubs blender addon, here we have a mediaframe named "Mediaframeplant003".

```json
{
  "class": "navmesh-hole",
  "components": {
    "attach-to-mediaframe": "name: Mediaframeplant003",
    "gltf-model": "plants/Plant_vase_3.glb"
  }
},
```

Don't use `position` and `rotation` if you're using `attach-to-mediaframe`, those components are mutually exclusive.

### Adding a model without mediaframe

```json
{
  "class": "navmesh-hole",
  "components": {
    "position": "2 0 14",
    "rotation": "0 0 0",
    "gltf-model": "plants/palm_pot.glb"
  }
},
```

You can find the correct values for `position` and `rotation` by pressing the `p` key while in the room in scene mode. It will show on the bottom your current avatar position and rotation.

Press `Ctrl+Shift+i` or `AltGr+i` to open the editor mode, press again the shortcut to be back in scene mode.

Move near the model you want to modify and open the editor, you can select the model and tweak the position and rotation, then copy the values in `elements.json`.

Note: There is currently an issue with mouse dragging in editor mode, it wrongly pivots from the center of the scene instead of where you are. Some tips:
- Press `f` key to focus on an object, the pivot will be the focused object.
- If you go back to scene mode and back to edit mode you will be again at your avatar position.

Note: If your model appears black, if generally means there is no lights defined, so be sure you have an envmap in your scene glb or that you use the `environment` component.

### Image on 2d mediaframe

```json
{
  "components": {
    "attach-to-mediaframe": "name: mediaframe006; resize: cover",
    "geometry": "primitive: plane; width: 2; height: 1",
    "material": "src: images/kia_red_2x1.webp; shader:flat; toneMapped: true; side: front"
  }
},
```

Note: this will change soon to be able to use a `"media-image": "images/kia_red_2x1.webp"` component instead of defining `geometry` and `material`.

### billboard

```js
schema: {
  size: { type: "string", default: "large", oneOf: ["large", "small"] },
  color: { type: "color", default: "#eeeeee" },
  screenColor: { type: "color", default: "#ffffff" },
  src: { type: "string", default: "" },
  text: { type: "string", default: "" },
  ratio: { type: "number", default: 1.777 }, // 16/9
},
```

large 16/9 billboard with image:

```json
{
  "enabled": true,
  "components": {
    "position": "-8 0.013 7",
    "rotation": "0 90 0",
    "billboard": "src:images/Formation-pratique_1024x576.jpg"
  }
},
```

if the image is square, specify the ratio 1:

```json
{
  "enabled": true,
  "components": {
    "position": "33 0.10 -22",
    "rotation": "0 140 0",
    "billboard": "src:images/a_photo_1024x1024.webp;ratio:1"
  }
},
```

The `billboard` component is used with the `teleporter` component when you set the `src` or `text` property. It's configured like this:

```json
"billboard": { "size": "small", "color": "#292824", "screenColor": "#292824", "src": "an_image.webp", "text": "some text" },
```

`screenColor` is used only if there is no image specified.

## Animating things

### Animated cube

```json
{
  "class": "navmesh-hole",
  "components": {
    "attach-to-mediaframe": "name: mediaframemid001",
    "animation": "property: object3D.rotation.y; from: 0; to: -360; dur: 13333; loop: true; easing: linear",
    "gltf-model": "cube.glb"
  }
},
```

### Animated platform with a car on it

```json
{
  "class": "navmesh-hole",
  "components": {
    "position": "-6.6 0.11 10.2",
    "geometry": "primitive: cone; height: 0.1; radiusBottom: 2.5; radiusTop: 2.4; segmentsRadial: 72",
    "material": "color: #000000; metalness: 0.7; roughness: 0.87",
    "animation": "property: object3D.rotation.y; from: 0; to: -360; dur: 40000; loop: true; easing: linear"
  },
  "children": [
    {
      "components": {
        "position": "0 0.1 0",
        "gltf-model": "cars/Ford_Mustang_MACH-E_LP2.glb",
      }
    }
  ]
},
```

### Play animation included in glb

You can use the `animation-mixer` component for that, see [options](https://github.com/c-frame/aframe-extras/tree/master/src/loaders#animation)

Default it to play all animations on repeat:

```json
    "gltf-model": "props/robot.glb",
    "animation-mixer": ""
```

This is equivalent to

```json
    "animation-mixer": "clip:*;loop:repeat"
```

If you have several animations, you can specify one by name, for example an animation called "move":

```json
    "animation-mixer": "clip:move"
```

If you want the animation to play in normal direction then in reverse on repeat, you can use:

```json
    "animation-mixer": "clip:move;loop:pingpong"
```

### animation-buttons

This is to run an animation on the parent entity. An animation clip name is attached to each button. The animation is run once.

The default state is the last frame of the last animation in the list, so the last button is not shown initially. You can change this with the `initialClip` property. The buttons are hidden during animation, and the clicked button disappears.

Here is an example of opening and closing a door of a car, the door is closed by default in the model. Each animation-buttons component is auto-creating a unique `animation-mixer` component on the parent. The minimum required property is buttons that take a comma separated list like this: `labelAnimation1,animationName1,labelAnimation2,animationName2`.

```json
  {
    "components": {
      "enabled": true,
      "class": "navmesh-hole",
      "position": "0 0.1 0",
      "gltf-model": "models/car.glb"
    },
    "children": [
      {
        "enabled": true,
        "components": {
          "position": "-0.85 1.45 -1",
          "rotation": "0 -90 0",
          "animation-buttons": "buttons:Open door,LF_door_open,Close door,LF_door_close"
        }
      },
      {
        "enabled": true,
        "components": {
          "position": "-0.85 1.45 1.4",
          "rotation": "0 -90 0",
          "animation-buttons": "buttons:Open door,LB_door_open,Close door,LB_door_close"
        }
      },
      {
        "enabled": true,
        "components": {
          "position": "0.85 1.45 1.4",
          "rotation": "180 -90 180",
          "animation-buttons": "buttons:Open door,RB_door_open,Close door,RB_door_close"
        }
      },
      {
        "enabled": true,
        "components": {
          "position": "0.85 1.45 -1",
          "rotation": "180 -90 180",
          "animation-buttons": "buttons:Open door,RF_door_open,Close door,RF_door_close"
        }
      }
    ]
  },
```

Complex example, full animation by default on loop, then you can trigger each step of the animation. Here you need to specify an `animationMixerId`, otherwise another `animation-mixer` is auto created giving weird results.

```json
  {
    "enabled": true,
    "class": "navmesh-hole",
    "components": {
      "position": "14 0 6",
      "rotation": "0 -90 0",
      "animation-mixer__1": "clip:Full_Anim;loop:pingpong",
      "gltf-model": "props/Surgery_Robot.glb"
    },
    "children": [
      {
        "components": {
          "position": "-1 1.222 0.98",
          "rotation": "-30 0 0",
          "animation-buttons": "animationMixerId:1;initialClip:Full_Anim;buttons:Step 1,Etape_1,Step 2,Etape_2,Step 3,Etape_3"
        }
      }
    ]
  },
```

### Load a 360 image on click

Example of a clickable sphere that loads a 360 image:

```json
  {
    "class": "clickable",
    "components": {
      "position": "-12 0.28 6",
      "geometry": "primitive:sphere;radius:0.25",
      "material": "src:360/room1_empty.jpg",
      "load360onclick": "360/room1_empty.jpg"
    }
  },
```

The `clickable` class is needed to register the click event for the `load360onclick` component. This is not needed if you have the `waypoint` component on the entity, the `waypoint` component is auto adding the `clickable` class.

To use a stereoscopic top-bottom image, suffix your image by `_TB` like this `myimage_TB.webp`.
This uses a modified version of the [aframe stereo component](https://github.com/oscarmarinmiro/aframe-stereo-component) to support top-bottom images ([see relevant PR](https://github.com/oscarmarinmiro/aframe-stereo-component/pull/45)).

### uv-scroll

```json
  {
    "enabled": true,
    "components": {
      "position": "-8 0.013 7",
      "rotation": "0 90 0",
      "billboard": ""
    },
    "children": [
      {
        "components": {
          "position": "0 2.175 0.12",
          "geometry": "primitive:plane;width:4;height:2.25",
          "material": "src:Lambo_Scroll.jpg;shader:flat;repeat: 0.33 1",
          "uv-scroll": {
            "increment": { "x": 0.33, "y": 0 },
            "speed": { "x": 0.1, "y": 0 }
          }
        }
      }
    ]
  },
```

## Navmesh and movement

### Navmesh

You can use the nav-mesh and visible (with value false) hubs component when you create your scene in blender with the hubs addon.

Or you can create a plane with class "navmesh" in the json:

```json
{
  "class": "navmesh",
  "components": {
    "position": "0 0 0",
    "rotation": "-90 0 0",
    "geometry": "primitive: plane; width: 500; height: 500",
    "material": "color: green",
    "visible": "false"
  }
},
```

Having the class "navmesh" will automatically add the `nav-mesh` component from aframe-extras (used for ai agents) and `simple-navmesh-constraint` (used for your avatar) is set automatically on your camera rig if navmesh entities are available.
For information, it's configured with those parameters:
`navmesh:.navmesh;fall:0.5;height:0;exclude:.navmesh-hole;`

### Navmesh hole

The `"class": "navmesh-hole"` line will dynamically cut the navmesh so the avatar can't go inside the bounding box of the mesh/model. Depending of your mesh, you may want to allow traversing it, in this case don't use the class.

For now, define a simple plane as a child and use the class on it, [see example](https://github.com/AdaRoseCannon/aframe-xr-boilerplate#simple-navmesh-constraintjs) in html:

```html
<a-gltf-model src="piano.glb" position="-6 0 1">
  <a-plane
    class="navmesh-hole"
    rotation="-90 0 0"
    width="1.5"
    height="0.6"
    position="0 0.001 0"
    color="red"
    visible="true"
  ></a-plane>
</a-gltf-model>
```

converted to json that would be:

```json
{
  "components": {
    "gltf-model": "piano.glb",
    "position": "-6 0 1"
  },
  "children": [
    {
      "class": "navmesh-hole",
      "components": {
        "position": "0 0.001 0",
        "rotation": "-90 0 0",
        "geometry": "primitive: plane; width: 1.5; height: 0.6",
        "material": "color: red",
        "visible": "true"
      }
    }
  ]
},
```

### Clickable seat

```json
  {
    "id": "wp3",
    "class": "seat",
    "components": {
      "position": "1.573 0.5 -1.744",
      "rotation": "0 90 0",
      "waypoint": { "canBeClicked": true, "canBeOccupied": true, "willDisableMotion": true }
    }
  },
```

Note that for clickable waypoint, it's really rotated by 180 degrees. This is not the case for non clickable waypoint like spawn point or teleportable zones, if you use them via hubs blender addon those will be wrong by 180 degrees currently.
This can be a bit weird when adding clickable waypoints via the editor and specifying the y rotation with the current rotation of the camera rig (you can see it with pressing `p` shortcut).
This is currently this way because of the blender hubs convention that uses blender Y-negative for the forward direction for the avatar. FrameVR on the other end uses the Y-positive ([see video](https://youtu.be/E136tQdsWLw?t=323) that uses `fcta` named node for chairs) that matches better the aframe rotation you see.
I'll probably change the behavior later to apply the 180 degrees rotation only for waypoint components defined in glb.
I'll add support for `fcta` named node in the future to add the waypoint component to it.

`waypoint` schema with defaults:

```js
schema: {
  canBeClicked: { type: "bool", default: false },
  canBeOccupied: { type: "bool", default: false },
  canBeSpawnPoint: { type: "bool", default: false },
  snapToNavMesh: { type: "bool", default: false },
  willDisableMotion: { type: "bool", default: false },
  willDisableTeleporting: { type: "bool", default: false },
  willMaintainInitialOrientation: { type: "bool", default: false },
  isOccupied: { type: "bool", default: false },
  occupiedBy: { type: "string", default: "scene" },
}
```

`snapToNavMesh`, `willDisableTeleporting`, `willMaintainInitialOrientation` are currently not used.

### Clickable stand up position

This is the same as the clickable seat but using `willDisableMotion` to false.

```json
  {
    "id": "wp4",
    "components": {
      "position": "1.573 0.5 -1.744",
      "rotation": "0 90 0",
      "waypoint": { "canBeClicked": true, "canBeOccupied": true, "willDisableMotion": false }
    }
  },
```

### Spawn point

```json
  {
    "enabled": true,
    "id": "spawnPoint",
    "components": {
      "position": "28 0.13 2",
      "rotation": "0 -90 0",
      "waypoint": { "canBeSpawnPoint": true }
    }
  },
```

Current behavior is using the exact position defined, so several avatars can be at the same position. Avatars that are close to you (lower than 0.5m) are invisible to you but other participants can see that there are avatars on the same position.
I'll probably implement a repulsive behavior in the future if there is already an avatar at that position.

### Spawn point seated

```json
  {
    "id": "wp1",
    "class": "seat",
    "components": {
      "load360onclick": "360/Round_Table_cam1_360.jpg",
      "position": "0 0 0",
      "rotation": "0 0 0",
      "waypoint": { "canBeSpawnPoint": true, "willDisableMotion": true }
    }
  },
```

If there is a `load360onclick` component associated with a spawn point, the 360 image will be loaded when you enter the room.

### Teleportable zone

Left and right arrows are shown to switch between zones if you set the `"teleportableZones": true` option.

A teleportable zone is a waypoint with all the options to false (the default):

```json
  {
    "id": "zoneA",
    "components": {
      "position": "-7.58 0.03 16.48",
      "rotation": "-2 90 0",
      "waypoint": {}
    }
  },
```

### change-scene

Change scene on click (default), the change the scene for all participants.

```json
"change-scene": "sceneId:real-estate-1"
```

Schema with defaults:

```js
schema: {
  on: { type: "string", default: "click" },
  sceneId: { type: "string", default: "" },
}
```

> Note: As a convention, component name starting with a verb does an action when triggered by an event specified by the `on` property.

### change-room

Change room on click (default), it change the room only for you.

```json
"change-room": "roomId:12345678;sceneId:real-estate-1"
```

Schema with defaults:

```js
schema: {
  on: { type: "string", default: "click" },
  roomId: { type: "string", default: "" },
  sceneId: { type: "string", default: "" },
}
```

### teleporter

With image:

```json
{
  "enabled": true,
  "components": {
    "position": "8 0 5.5",
    "rotation": "0 -45 0",
    "change-room": "on:hitstart;roomId:12345678;sceneId:my-scene",
    "teleporter": {
      "src": "/scenes/my-scene/preview.webp"
    }
  }
}
```

With text:

```json
{
  "enabled": true,
  "components": {
    "position": "44.48 0.004 -25.54",
    "rotation": "0 -140 0",
    "move-to-waypoint": "on:hitstart;id:zoneCuveTop",
    "teleporter": { "text": "Monter en haut\nde la cuve" }
  }
}
```

You can have both image and text.

### move-to-waypoint

To link to another zone inside the same room, for this you create a named `waypoint` (the id) with all the properties to false (the default) in the area you want to go to.

```json
  {
    "enabled": true,
    "id": "lobby",
    "components": {
      "position": "28 0.13 2",
      "rotation": "0 -90 0",
      "waypoint": ""
    }
  },
```

then use the `move-to-waypoint` component on a plane for example:

```json
  {
    "class": "clickable",
    "components": {
      "position": "45 1.6 -1.7",
      "rotation": "0 -150 0",
      "geometry": "primitive:plane;width:1;height:0.5",
      "material": "color:black;opacity:1.0",
      "text": "value:Click to go up;align:center;font:/assets/custom-msdf.json;negate:false",
      "move-to-waypoint": "id:lobby"
    }
  },
```

or on a `teleporter` for example:

```json
{
  "components": {
    "position": "8 0 5.5",
    "rotation": "0 -45 0",
    "move-to-waypoint": "on:hitstart;id:lobby",
    "teleporter": {
      "src": "destination.webp"
    }
  }
}
```

## Other components

### open-url-on-click

```json
  {
    "enabled": true,
    "class": "clickable",
    "components": {
      "position": "-7.5 1.8 12.76",
      "rotation": "0 180 0",
      "geometry": "primitive:plane;width:2;height:1.04;depth:0.01",
      "material": "src:360/room1_empty_preview.webp;shader:flat",
      "open-url-on-click": "https://allocola.com/scenes/real-estate-360/#config=room1-empty/config.json"
    }
  }
```

### event-set

See [documentation](https://github.com/supermedium/superframe/tree/master/components/event-set/#readme)

## Adding a NPC

### Add a NPC

```json
  {
    "enabled": true,
    "class": "npc",
    "components": {
      "position": "-7.73 0.03 -4.22",
      "rotation": "0 140 0",
      "player-info": "name:Odette;avatarSrc:/avatars/models/female/044.glb;avatarOutfit:outfit_3_lowpoly;state:IDLE;avatarPose:sit"
    }
  }
```

`"class": "npc"` will add an `agent-movement` component so that staff can control npc by clicking on nametag.

`state:IDLE` or any available animation.

`avatarPose:sit` is currently needed if you want to sit on a seat. It won't be needed if we do a `attach-to-waypoint` component.

### NPC with an AI

Corporate woman agent:

```json
  {
    "enabled": true,
    "class": "npc",
    "components": {
      "position": "0 0.03 13.9",
      "player-info": "name:Tiphany;avatarSrc:/avatars/models/Assistante01.glb;avatarOutfit:outfit_0_lowpoly|21",
      "ai-agent": {
        "enabled": true,
        "gender": "female",
        "lang": "fr",
        "personality": "Tu te nommes #agent, une vendeuse automobile dans un showroom virtuel. Tu es prête à aider avec toutes les questions que les visiteurs pourraient avoir, et tu es capable de le faire de manière optimiste et positive. Tu es également empathique et compréhensive vis-à-vis des émotions et des besoins des autres. Tu travailles pour "Allocola Cars", dont le site internet est https://allocola.com réalisé par l'entreprise "Illusion 3D" spécialisée dans les metaverses. Tu proposes aux visiteurs de ne parler que des automobiles qui sont dans le showroom : la Ford Mustang MACH-E dans la zone A, l'OPEL MOKKA E dans la zone B et la KIA EV6 GT dans la zone C. Tu peux te déplacer dans les 3 zones, et demander aux visiteurs de te suivre pour leur permettre de voir les voitures. Il est possible également de voir les voitures en réalité augmentée en scannant avec le téléphone le QR code s'affichant au-dessus de la voiture lorsqu'on est proche de celle-ci. Tu n'es pas autorisé à discuter sur d'autres sujets que les automobiles du showroom que tu dois présenter. Pour les sujets autres tu proposes de laisser sur le tchat les demandes du client afin qu'une personne de "Allocola Cars" les recontacte. Tu peux soit proposer de faire le tour des modèles ou alors d'écouter le besoin du client pour lui proposer la voiture la plus adéquate. Il y a une remise sur la KIA de février à aout de 2% sur l'année 2023. Tu ne peux pas proposer d'essais routiers."
      }
    }
  },
```

Young girl agent:

```json
{
  "enabled": true,
  "class": "npc",
  "id": "lina",
  "components": {
    "position": "40 0 -28",
    "rotation": "0 180 0",
    "player-info": "name:Lina;avatarSrc:/avatars/models/YoungGirl02.glb;avatarOutfit:outfit_3_lowpoly|2",
    "ai-agent": {
      "pitch": 1.1,
      "enabled": true,
      "gender": "female",
      "lang": "fr",
      "greetingMessage": "...",
      "personality": "..."
    }
  }
}
```

You can use `#agent` placeholder in the personality field, it will be replaced by the `player-info` name value.

Default values for the `ai-agent` component are:

```json
"enabled": true, // You can disable the AI agent by setting it to false.
"enabledForAll": false, // By default, only staff can chat with the AI agent, set it to true to allow everybody.
"gender": "female", // used for tts voice, "female" or "male"
"lang": "fr", // used for tts voice, you can use "en" for English
"personality": "",
"greetingMessage": "Bonjour, est-ce que je peux vous renseigner ?",
"model": "gpt-3.5-turbo", // Other choices are "text-davinci-003", "gpt-4"
"temperature": 0.7, // between 0 and 2
"maxTokens": 200, // Between 0 and 4096 tokens or more depending on model. Note this is used for both prompt and response on gpt-3.5-turbo, you can set it to null to not give any limit for the response.
"pitch": 0.8, // pitch of the voice, between 0.8 and 1.2 is good. Not used with Coqui TTS.
"rate": 1.2, // speed of the voice, between 1.0 and 1.2 is good. Not used with Coqui TTS.
"useCoqui": true, // Use the Coqui TTS server when speechSynthesis API is not available (on Meta browser with Quest headset)
"alwaysUseCoqui": false, // Always use the Coqui TTS server even if speechSynthesis API is available
```

You can change them if needed.

For ChatGPT api, aka "gpt-3.5-turbo", see
[https://platform.openai.com/docs/guides/chat](https://platform.openai.com/docs/guides/chat)
[https://platform.openai.com/docs/api-reference/chat/create](https://platform.openai.com/docs/api-reference/chat/create)

The personality field is used as a message with role system.

If you use the "text-davinci-003" model (GPT-3), the following string in appended to the personality field automatically to be used as the prompt:

```json
"\n###\n#conversation\n#speaker: #input\n#agent:"
```

For both models, those values are hard coded in the server code that make the API call:

```js
stop: ["###"],
```

Only the ten latest messages in the chat are used when making the API call.

## UI customization

### Top banner

This will show a top box that you can close with the specified text, with optional action button.

```json
"banner": {
  "actionButton": {
    "label": "Télécharger la brochure",
    "url": "https://allocola.com/scenes/oil-refinery/ILLUSION3D_Plaquette_VR_Immersion.pdf"
  },
  "text": "Bienvenue sur la solution allocola.com de l'entreprise ILLUSION3D, dans ce module de démonstration, vous pouvez en même temps vous connecter à la fois en VR, en desktop et en mobile."
},
```

### Buttons

You can define custom buttons to be added to the user interface. A button has a label and one action or more.

`moveTo` and `sendMessage` actions:

```json
"buttons": [
  {
    "label": "grue plateforme 1",
    "actions": [
      {
        "type": "moveTo",
        "position": "17.36 36.5 -9.14",
        "rotation": "-55 -80 0"
      },
      {
        "type": "sendMessage",
        "onlyOnce": true,
        "role": "user",
        "content": "On est en haut, à côté de la cabine de la grue. Que peut-on en dire ?"
      }
    ]
  }
],
```

`moveTo` can also have an optional `"target": "#aiAgent1"` to move the AI agent instead of the avatar.

For the `sendMessage` action, when you click the button it will send the message to the closest `ai-agent` you encountered. If `onlyOnce` is true (false by default), it will send the content only once. If you click the button again, it won't trigger the action.

The field `"role"` follows the ChatGPT semantic, so can be `"user"` or `"assistant"`, for example:

```json
  {
    "label": "sol",
    "actions": [
      {
        "type": "moveTo",
        "position": "6 0.013 0.6",
        "rotation": "0 -90 0"
      },
      {
        "type": "sendMessage",
        "onlyOnce": true,
        "role": "assistant",
        "content": "Voulez-vous que je vous fasse un résumé de la grue ?"
      }
    ]
  },
```

`open360` action:

```json
  {
    "label": "pièce avec homestaging 1",
    "actions": [
      {
        "type": "open360",
        "src": "360/room1_homestaging2_4k.webp"
      }
    ]
  },
```

`close360` action:

```json
  {
    "label": "Retour au showroom",
    "availableExpr": "isOpened360",
    "actions": [
      {
        "type": "close360"
      }
    ]
  },
```

`availableExpr` / `disabledExpr` options:

```json
    "availableExpr": "isOpened360",
```

makes the button visible only if a 360 image is opened.

To show the button only if user is moderator:

```json
    "availableExpr": "isModerator",
```

To show the button but disabled if the user is not moderator:

```json
    "disabledExpr": "not isModerator",
```

To temporarily remove the button but keep it in the json file:

```json
   "availableExpr": "isCommented",
```

To remove a button if in VR:

```json
   "availableExpr": "not isInVR",
```

`changeScene` action:

```json
  {
    "label": "Retour au showroom",
    "actions": [
      {
        "type": "changeScene",
        "sceneId": "real-estate"
      }
    ]
  },
```

`changeRoom` action:

```json
  {
    "label": "Retour au showroom",
    "actions": [
      {
        "type": "changeRoom",
        "roomId": "12345678",
        "sceneId": "real-estate"
      }
    ]
  },
```

### Training

A training consists of a label (title) and is composed of several sections containing lessons.
A lesson is really a UI button like described in the previous section that run actions.

```json
"training": {
  "label": "Formation basique BTP",
  "sections": [
    {
      "label": "Les zones du chantier",
      "lessons": [
        {
          "label": "Entrée sur le chantier",
          "actions": [
            {
              "type": "moveTo",
              "position": "1.18 0.02 2.11",
              "rotation": "0 153 0"
            },
            {
              "type": "moveTo",
              "target": "#tiphany",
              "position": "-4.47 0.02 6.43",
              "rotation": "0 -40 0"
            },
            {
              "type": "sendMessage",
              "role": "assistant",
              "content": "Entrée sur le chantier : il est important de comprendre les procédures de sécurité à suivre pour entrer sur un chantier car cela affecte directement la sécurité de toutes les personnes sur le chantier. Suivez-moi."
            },
            {
              "delay": 12000,
              "type": "moveTo",
              "target": "#tiphany",
              "walk": true,
              "position": "2.8 0.02 5.8",
              "rotation": "0 -140 0"
            },
            {
              "delay": 14000,
              "trigger": "navigation-end",
              "type": "moveTo",
              "target": "#tiphany",
              "walk": true,
              "position": "2.8 0.02 16.3",
              "rotation": "0 90 0"
            },
            {
              "delay": 18000,
              "trigger": "navigation-end",
              "type": "moveTo",
              "target": "#tiphany",
              "walk": true,
              "position": "-2.12 0.02 13.28",
              "rotation": "0 -112 0"
            },
            {
              "delay": 23000,
              "trigger": "hitstart",
              "type": "sendMessage",
              "role": "assistant",
              "content": "À gauche le badgeage et à droite le passage piéton là où se trouvent les plots."
            }
          ]
        }
      ]
    }
  ]
```

### Options and overrides

The following configurable options are available with their default value:

```json
  "options": {
    "voice": true,
    "chat": true,
    "enterMultiplayer": true,
    "enterSolo": false,
    "enterSoloRightAway": false,
    "help": true,
    "movementControls": true,
    "cursorTeleportControls": true,
    "orbitControls": false,
    "shareCamera": true,
    "shareScreen": true,
    "sceneSwitcher": false, // deprecated, it works only with public scenes on home page
    "roomSwitcher": true,
    "syncOpened360BetweenParticipants": false,
    "useInstancedMesh": true,
    "teleportableZones": false,
    "followModeratorToZone": false,
    "aiChatEdit": false,
    "targetOrigin": ""
  },
```

- `roomSwitcher` enables a menu with the rooms defined in the `/tenants/${tenanId}/rooms.json` file when you set `"tenantId": "some-customer-name"` in `elements.json` (for now, it will be deprecated once we'll use a database to store the scenes and rooms). Having the `rooms.json` file also enables the `https://allocola.com/tenantId` url that list the rooms.
- `teleportableZones` shows the left and right arrows to switch between zone
- `followModeratorToZone` will follow a moderator when they switch zone via the UI buttons that use the `moveTo` action or use a `teleporter` that switch zone with the `move-to-waypoint` component.
- `useInstancedMesh` auto use instanced mesh for `gltf-model` having the same url.
- `aiChatEdit` shows a "Start a new conversation" and "Edit personality" buttons in AI Chat.
- `targetOrigin` is used to define the target origin for the [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) calls to the parent window for an experience embed as an iframe like a configurator. The calls are done only if `targetOrigin` is not the empty string, for example `"targetOrigin": "https://allocola.com"`. For now only this event is implemented:
```js
const targetOrigin = options.targetOrigin;
const data = { category: categoryLabel, choice: choice.label };
window.parent.postMessage(
  { action: "choice-selected", detail: data }, targetOrigin);
```
Example of iframe integration in https://allocola.com/scenes/shower-configurator/
## Scene templates

For a simple configurator experience without multi users:

```json
  "overrides": {
    "orbitControls": {
      "autoRotate": true,
      "autoRotateSpeed": 0.75,
      "enableRotate": true,
      "rotateSpeed": 0.5,
      "target": "0 0.5 0"
    }
  },
  "options": {
    "movementControls": false,
    "orbitControls": true,
    "help": false,
    "enterMultiplayer": false,
    "enterSolo": true,
    "enterSoloRightAway": true
  },
```

For a 360 home staging experience:

```json
  "components": {
    "environment": "playArea:400;preset:arches;dressing:none;ground:none;shadow:false;fog:0"
  },
  "overrides": {
    "hemisphereLightIntensity": 2.0
  },
  "options": {
    "syncOpened360BetweenParticipants": true,
    "movementControls": false
  },
```

For other experience like a round table with several views, you want to keep `syncOpened360BetweenParticipants` to false (the default).

### Scene with only 360 images

If you create a scene with only 360 images, you will have something like this:

```json
{
  "sceneTitle": "Roundtable",
  "sceneModel": "",
  "components": {
    "environment": "playArea:400;preset:arches;dressing:none;ground:none;shadow:false;fog:0"
  },
  "overrides": {
    "hemisphereLightIntensity": 2.0
  },
  "options": {
    "syncOpened360BetweenParticipants": false
  },
  "children": [
    {
      "id": "wp1",
      "class": "seat",
      "components": {
        "load360onclick": "360/Round_Table_cam1_360.jpg",
        "position": "0 0 0",
        "rotation": "0 0 0",
        "waypoint": { "canBeSpawnPoint": true, "willDisableMotion": true }
      }
    },
    {
      "id": "wp2",
      "class": "seat",
      "components": {
        "load360onclick": "360/Round_Table_cam2_360.jpg",
        "position": "-0.088 0.853 -5.364",
        "rotation": "0 0 0",
        "waypoint": { "canBeClicked": true, "canBeOccupied": true, "willDisableMotion": true }
      }
    },
    {
      "components": {
        "position": "-0.088 0.403 -5.364",
        "rotation": "0 0 0",
        "gltf-model": "models/Chaise_2.glb"
      }
    },
    {
      "components": {
        "position": "0 0 0",
        "rotation": "0 -90 0",
        "gltf-model": "models/Estrade.glb"
      }
    }
  ]
}
```

Your camera in blender/3dsmax where you make the 360 image render needs to be at the position of the eyes of the avatar, that's it 1.6m when standing and 1.6-0.45=1.15m when seated. If your chair origin on the platform is at 0.403 and waypoint on chair is 0.853, you should have your camera on a seat at 0.853 + (1.15 - (0.853 - 0.403)) = 1.553

More generally waypointPosition + (1.15 - (waypointPosition - chairPosition))
So simply chairPosition + 1.15
Really?

`syncOpened360BetweenParticipants` is true by default, depending on your scene you may want to set it to false.

## Object info panel

See [Object info panel](object_info_panel.md)

