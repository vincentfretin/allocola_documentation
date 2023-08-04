### Object info panel

```json
  "objects": [
    {
      "id": "obj4",
      "title": "",
      "description": "",
      "image": "../amplitude-shared/images/LOGO_AMPLIVOLT.png",
      "actionButton": {
        "label": "En savoir plus",
        "url": "https://www.amplivolt.fr"
      }
    }
  ],
```

If there is a description, it will include a close button to be able to close it for mobile.

### Panel theme

By default the panel will be `light` or `dark` depending of the user preferred color-scheme defined in their OS settings. But you can force it for a panel, for dark:

```json
  "objects": [
    {
      "id": "obj1",
      "theme": "dark",
```

for light:

```json
  "objects": [
    {
      "id": "obj1",
      "theme": "light",
```

### Color based configurator

Each choice can have optional actions, it's the same actions as for Buttons described above.

```json
  "objects": [
    {
      "id": "obj1",
      "title": "Supercar",
      "qrcode": {
        "model": "cars/supercar.glb"
      },
      "choices": [
        {
          "label": "NOIR",
          "color": "#000000",
          "actions": [
            {
              "type": "sendMessage",
              "onlyOnce": true,
              "role": "assistant",
              "content": "Le noir c'est la meilleure couleur !"
            }
          ],
          "onselect": {
            "material-values__body": "materialName:body_color;color:#000000;metalness:0.85;roughness:0.25;opacity:1"
          }
        },
        {
          "label": "BLANC",
          "color": "#bcc0c1",
          "onselect": {
            "material-values__body": "materialName:body_color;color:#bcc0c1;metalness:0.85;roughness:0.25;opacity:1"
          }
        }
      ]
    }
  ],
```

A svg qrcode is generated for the given model.

If you need a png qrcode, you can create one with
[https://www.nayuki.io/page/qr-code-generator-library](https://www.nayuki.io/page/qr-code-generator-library)

The svg qrcode is created from the same library, with ECC Medium, border 4.

### Texture based configurator

```json
  "objects": [
    {
      "id": "obj1",
      "theme": "dark",
      "title": "Change wood",
      "qrcode": {
        "model": "models/cabinet.glb"
      },
      "choices": [
        {
          "label": "Dark wood",
          "preview": "textures/003_RV_WOOD_DARK_512.webp",
          "onselect": {
            "material-values__wood": "materialName:texture.wood.002;map:textures/003_RV_WOOD_DARK_1024.webp"
          }
        },
        {
          "label": "Light wood",
          "preview": "textures/light_wood_8_77_diffuse_512.webp",
          "onselect": {
            "material-values__wood": "materialName:texture.wood.002;map:textures/light_wood_8_77_diffuse_1024.webp"
          }
        }
      ]
    }
  ],
  "children": [
    {
      "enabled": true,
      "components": {
        "position": "0 0.01 0",
        "gltf-model": "models/cabinet.glb",
        "material-values__wood": "materialName:texture.wood.002;map:textures/003_RV_WOOD_DARK_1024.webp"
      },
      "children": [
        {
          "components": {
            "position": "-2 1.6 0",
            "rotation": "0 0 0",
            "object-info-panel": "ref:obj1"
          },
          "children": [
            {
              "class": "collidable",
              "components": {
                "geometry": "primitive:sphere;radius:6",
                "visible": false
              }
            }
          ]
        }
      ]
    }
  ]
```

Note: For a choice, you can set `"enabled": false` to temporarily hide this choice.
### Show banner in AR

Be aware this disables the take picture button on iPhone, but not on iPad. On Chrome Android, there is no take picture button (see [https://github.com/google/model-viewer/issues/1481](https://github.com/google/model-viewer/issues/1481)).

This will show title and description with a Close link on both Android and iOS:

```json
    {
      "id": "obj1",
      "theme": "dark",
      "title": "Feu à pellets FOREST LC",
      "description": "",
      "qrcode": {
      "model": "props/FOREST_LC.glb",
      "banner": {
        "description": "Hauteur (mm): 1066, Largeur (mm): 480, Profondeur (mm): 557"
      }
    },
```

The iOS specific banner is also implemented.
If you specify `ios` options, it takes precedence over the `banner` config.

Example:

```json
    {
      "id": "obj1",
      "theme": "dark",
      "title": "Feu à pellets FOREST LC",
      "description": "",
      "qrcode": {
        "model": "props/FOREST_LC.glb",
        "ios": {
          "callToAction": "Fermer",
          "checkoutTitle": "FOREST LC",
          "checkoutSubtitle": "Hauteur (mm): 1066, Largeur (mm): 480, Profondeur (mm): 557"
        }
      },
```

Or you can use a custom banner like this:

```json
    {
      "id": "obj1",
      "theme": "dark",
      "title": "Feu à pellets FOREST LC",
      "description": "",
      "qrcode": {
        "model": "props/FOREST_LC.glb",
        "ios": {
          "custom": "https://example.com/forest_lc_banner.html"
        }
      },
```

If you want to host your own html file for the AR banner, it needs to be served on https with a valid certificate. Here is an example of an html file:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8"/>
    <style>
body {
  font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
}
    </style>
  </head>
  <body>
    <div style="position:absolute;inset:0;padding:0px 20px;margin:0;font-size:12px;display:flex;flex-direction:column;justify-content:center">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <div>
          <div style="color:#000;font-weight:bold;margin-bottom:0.1em">FOREST LC</div>
          <div style="color:#747275">Hauteur (mm): 1066, Largeur (mm): 480, Profondeur (mm): 557</div>
        </div>
        <div style="padding-left:15px;font-size:16px"><a style="text-decoration:none;color:#007aff;font-weight:bold;" href="#">Fermer</a></div>
      </div>
    </div>
  </body>
</html>
```

This is the same html that is generated when you use the generic "banner" settings that work on both Android and iOS.

To know more about iOS specific options:
- https://github.com/google/model-viewer/discussions/2755
- https://developer.apple.com/documentation/arkit/adding_an_apple_pay_button_or_a_custom_action_in_ar_quick_look
- https://modelviewer-ios16-arql-banner-fix.glitch.me/

The `ios-src` attribute with be set on `model-viewer` tag accordingly if the usdz exists (specified `"model"` with glb extension replaced by usdz), for example: `model.usdz#applePayButtonType=plain&checkoutTitle=Spaceman%20Toy&checkoutSubtitle=1969 Limited Edition&price=$99&canonicalWebPageURL=https%3A%2F%2Fdeveloper.apple.com`
If the usdz file doesn't exist, it will be auto-generated, in this case the hash is set on `src` attribute ([see relevant code in model-viewer](https://github.com/google/model-viewer/blob/7fdf88c370e79ffdd2aa0549080faca9ae98152a/packages/model-viewer/src/features/ar.ts#L381-L384)).