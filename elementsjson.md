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

When you use the `environment` component, the `reflection` component is automatically added to create an envmap of the scene, you can disable it like this, you can also configure the created hemisphere light intensity that is created with the `environment` component:

```json
  "components": {
    "environment": "playArea:400;preset:default;dressing:none;shadow:false;fog:0"
  },
  "overrides": {
    "reflection": false,
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

## Available components

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

### Navmesh

You can use the nav-mesh and visible (with value false) hubs component when you create your scene in blender with the hubs addon.

Or you can create a plane in the json:

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

### Navmesh hole

The `"class": "navmesh-hole"` line will dynamically cut the navmesh so the avatar can't go inside the bounding box of the plant, but if your shadow plane is too large or the leaves of the plant are too large, the area cut will be wrong, so depending of the plant, you may want to allow just to traverse it, so removing the class. But first reduce your shadow plane.
If the glb file is too complex, it may hurt performance. In this case define a simple plane as a child and use the class on it, [see example](https://github.com/AdaRoseCannon/aframe-xr-boilerplate#simple-navmesh-constraintjs) in html:

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

Note: If you have a shadow plane in the glb and you have z-fighting, move up your plane by 1 mm in the model.

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

### Seat clickable

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
    "reflection": false,
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

### change-scene-on

Change scene on click (default), the change the scene for all participants.

```json
"change-scene-on": "sceneId:real-estate-1"
```

Defaults:

```json
event: "click"
sceneId: ""
```

### change-room-on

Change room on click (default), it change the room only for you.

```json
"change-room-on": "roomId:12345678;sceneId:real-estate-1"
```

Defaults:

```json
event: "click"
roomId: ""
sceneId: ""
```

### teleporter

This component uses the `change-room-on` component.

```json
{
  "enabled": true,
  "components": {
    "position": "8 0 5.5",
    "rotation": "0 -45 0",
    "teleporter": {
      "roomId": "12345678",
      "sceneId": "my-scene",
      "image": "/scenes/my-scene/preview.webp"
    }
  }
}
```


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

### event-set

See [documentation](https://github.com/supermedium/superframe/tree/master/components/event-set/#readme)

### Add a NPC

```json
  {
    "enabled": true,
    "class": "npc",
    "components": {
      "position": "-7.73 0.03 -4.22",
      "rotation": "0 140 0",
      "player-info": "nametag:Odette;avatarSrc:/avatars/models/female/044.glb;avatarOutfit:outfit_3_lowpoly;state:IDLE;avatarPose:sit"
    }
  }
```

`"class": "npc"` will add an `agent-movement` component so that staff can control npc by clicking on nametag.

`state:IDLE` or any available animation.

`avatarPose:sit` is currently needed if you want to sit on a seat. It won't be needed if we do a `attach-to-waypoint` component.

### NPC with an AI

```json
  {
    "enabled": true,
    "class": "npc",
    "components": {
      "position": "0 0.03 13.9",
      "player-info": "nametag:Tiphany;avatarSrc:/avatars/models/Assistante01.glb;avatarOutfit:outfit_0_lowpoly|21",
      "ai-agent": {
        "enabled": true,
        "lang": "fr",
        "personality": "Tu te nommes #agent, une vendeuse automobile dans un showroom virtuel. Tu es prête à aider avec toutes les questions que les visiteurs pourraient avoir, et tu es capable de le faire de manière optimiste et positive. Tu es également empathique et compréhensive vis-à-vis des émotions et des besoins des autres. Tu travailles pour "Allocola Cars", dont le site internet est https://allocola.com réalisé par l'entreprise "Illusion 3D" spécialisée dans les metaverses. Tu proposes aux visiteurs de ne parler que des automobiles qui sont dans le showroom : la Ford Mustang MACH-E dans la zone A, l'OPEL MOKKA E dans la zone B et la KIA EV6 GT dans la zone C. Tu peux te déplacer dans les 3 zones, et demander aux visiteurs de te suivre pour leur permettre de voir les voitures. Il est possible également de voir les voitures en réalité augmentée en scannant avec le téléphone le QR code s'affichant au-dessus de la voiture lorsqu'on est proche de celle-ci. Tu n'es pas autorisé à discuter sur d'autres sujets que les automobiles du showroom que tu dois présenter. Pour les sujets autres tu proposes de laisser sur le tchat les demandes du client afin qu'une personne de "Allocola Cars" les recontacte. Tu peux soit proposer de faire le tour des modèles ou alors d'écouter le besoin du client pour lui proposer la voiture la plus adéquate. Il y a une remise sur la KIA de février à aout de 2% sur l'année 2023. Tu ne peux pas proposer d'essais routiers."
      }
    }
  },
```

You can use `#agent` placeholder in the personality field, it will be replaced by the `player-info` nametag value.

Default values for the `ai-agent` component are:

```json
"enabled": true, // You can disable the AI agent by setting it to false.
"enabledForAll": false, // By default, only staff can chat with the AI agent, set it to true to allow everybody.
"lang": "fr", // used for tts voice, you can use "en" for English
"personality": "",
"greetingMessage": "Bonjour, est-ce que je peux vous renseigner ?",
"model": "gpt-3.5-turbo", // The other choice is "text-davinci-003"
"temperature": 0.7, // between 0 and 2
"maxTokens": 200, // between 0 and 4096-prompt tokens
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

```json
stop: ["###"],
```

Only the ten latest messages in the chat are used when making the API call.

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

To show the button only if user is moderator :

```json
    "availableExpr": "isModerator",
```

To show the button but disabled if the user is not moderator

```json
    "disabledExpr": "not isModerator",
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

### Options and overrides

The following configurable options are available with their default value:

```json
  "options": {
    "voice": true,
    "chat": true,
    "configurator": false,
    "enterMultiplayer": true,
    "enterSolo": false,
    "enterSoloRightAway": false,
    "help": true,
    "hubsAvatars": false,
    "movementControls": true,
    "orbitControls": false,
    "shareCamera": true,
    "shareScreen": true,
    "sceneSwitcher": false,
    "syncOpened360BetweenParticipants": false,
    "useInstancedMesh": true,
  },
```

For a simple configurator experience without multi users:

```json
  "overrides": {
    "reflection": false,
    "hemisphereLightIntensity": 0,
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
    "reflection": false,
    "hemisphereLightIntensity": 2.0
  },
  "options": {
    "syncOpened360BetweenParticipants": true,
    "movementControls": false
  },
```

For other experience like a round table with several views, you want to keep `syncOpened360BetweenParticipants` to false (the default).

## Object info panel

See [Object info panel](object_info_panel.md)
