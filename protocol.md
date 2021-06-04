# Protocol via RS232 / UART

This document uses C# type declarations to illustrate datatypes.

## Start
| Identifier     | Transmit    | Reply                                                               |
| -------------- | ----------- | ------------------------------------------------------------------- |
| Dev ID / Hello | `0x69` - I  | `0xBAFC01` - the last byte indicates the version between 00 and 255 |

## Sensors
There are two sensor types with specific byte ranges to ask for the value.

Temperatures are always reported in Celsius. If you want to use Kelvin or Fahrenheit, you need to use the known equasions to convert the units.

### Get all sensors at once

You send: `0xAA`

You receive:  
* array of `short` of all temp values (deg C times 100 for two decimal places)
* `0x0D0A` - divider for where the fan sensor values start
* array of `ushort` of the time passed between revolutions (1 second average)
* `0x0D0A` - divider for where the matrix return values for each channel starts
* array of `float` of the matrix return values for each channel
* `0x0D0A` - divider for where the resulting fan duty cycle array starts
* array of `byte` of the resulting duty cycle values for each channel (0-100 / `0x00` - `0x64`)

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