# Protocol via RS232 / UART

This document uses C# type declarations to illustrate datatypes.

## Start
| Identifier     | Transmit    | Reply                                                               |
| -------------- | ----------- | ------------------------------------------------------------------- |
| Dev ID / Hello | `0x69` - I  | `0xBAFC01` - the last byte indicates the version between 00 and 255 |

## Sensors
- Temperatures are always reported in Celsius. If you want to use Kelvin or Fahrenheit, you need to use the known equasions to convert the units.

### Get all sensors at once

You send: `0xAA`

You receive:  
* `byte` - sensor_count
* `byte` - channel_count
* temperature sensor values: `short[sensor_count]` (deg C times 100 for two decimal places)
* fan sensor9 values: `ushort[channel_count]` (time passed between revolutions, averaged over 1 second in ms * 10 (tenths of a millisecond))
* matrix return values: `float[channel_count]`
* resulting fan duty cycles: `byte[channel_count]` (0-100 / `0x00` - `0x64`)

in one stream.

## Curves

### Set up curves

You send: `0xB0`, then a `byte` of the curve ID and then in repeatring pattern `short` temperature (deg C times 100 for two decimal places), `byte` duty cycle (0-100 / `0x00` - `0x64`)  
You receive `0xAC`, something else is an error code.

```cs
struct CurvePoint {
    short temperature;
    byte  dutycycle;
}
CurvePoint[] curve;
```

### Get curve set up

You send: `0xB1`, then a `byte` of the curve ID  
You receive: An array of the structure shown in "Set up curves".

## Matrix

### Set up matrix

You send: `0xC0`, then a `byte` of the matrix ID and then an array of `float` for each matrix coefficient for each teperature sensor value in sequence.  
You receive `0xAC`, something else is an error code.

### Get matrix

You send: `0xC1`, then a `byte` of the matrix ID  
You receive: An array of `float` for each matrix coefficient for each teperature sensor value in sequence.

## Test duty cycle

### Test

You send `0xD0` and then a `byte` with the channel ID, then a `byte` with the duty cycle (0-100 / `0x00` - `0x64`).  
You receive `0xAC`, something else is an error code.

### Return to curve operation

You send `0xD1`  
You receive `0xAC`, something else is an error code.