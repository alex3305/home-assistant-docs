# Silvercrest LED String Lights

![Silvercrest LED String Lights](https://www.lidl-service.com/static/5019460647/pic/main.jpeg)

## Device attributes

### Connectivity

This device is connected through Zigbee.

### Power

This device is powered with a mains power plug.

### Factory reset

Hold the Function (F) key for 5 seconds. When succesful the LED string light will breathe white.

### Pairing (Phoscon)

Pairing is done through resetting the LED string lights. After a reset, pairing should be automatic when the LED string lights are breathing white.

## Control

Control can be done either through deCONZ (Phoscon), deCONZ REST API or Home Assistant. Using the deCONZ REST API gives you the most control.

### Applying custom effects

Through the deCONZ REST API it is possible to apply custom effects with custom colours to the LED string lights. This is currently unsupported within Phoscon or Home Assistant. When you have obtained an API key from the deCONZ REST API (see references below), it is possible to apply an effect.

### Example requests

#### Twinkle multiple colourss

**`PUT http://home-assistant:40850/api/MY_API_KEY/lights/24/state`**

```json
{
    "effect": "twinkle",
    "effectSpeed": 10,
    "effectColours": [
        [172, 0, 0],
        [128, 100, 100],
    ]
}
```

#### Soothing mood light

**`PUT http://home-assistant:40850/api/MY_API_KEY/lights/24/state`**

```json
{
    "effect": "carnival",
    "effectSpeed": 10,
    "effectColours": [
        [172, 0, 0],
        [150, 0, 96],
        [150, 0, 0],
        [128, 100, 100],
        [0, 128, 0],
        [0, 0, 128]
    ]
}
```

!!! note
    It seems that the maximum number of user adjustable colours is 7.

!!! note
    The effect speed should be a value from 0 - 100 in percent. Although for some effects the changed speed seems barely noticable or is inverted.

### Effect list

* `steady` (single colour)
* `snow` (single colour)
* `rainbow` (pre-defined colours)
* `snake`
* `twinkle`
* `fireworks`
* `flag`
* `waves`
* `updown`
* `vintage`
* `fading` 
* `collide`
* `strobe`
* `sparkles`
* `carnival` (7 colours)
* `glow` (7 colours)

## Support

This device is supported since deCONZ 2.07.01.

## Notes

Sometimes the device can appear as `Heiman TS0601`. To fix this, just read the Zigbee Basic attributes from within the deCONZ GUI.

## References

* [deCONZ REST API - Getting Started](https://dresden-elektronik.github.io/deconz-rest-doc/getting_started/)
* [ebaauw in dresden-elektronik/deconz-rest-plugin!3716](https://github.com/dresden-elektronik/deconz-rest-plugin/issues/3716#issuecomment-735467996)
