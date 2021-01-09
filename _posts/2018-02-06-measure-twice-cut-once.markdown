---
layout: post
title: "Measure Twice, Cut Once"
date: 2018-02-06 19:04:26 GMT
tags: Foundation API swift
---

We can all agree, measurements are not sexy, but they can be important, and troublesome. Don’t believe? Ask [NASA](https://edition.cnn.com/TECH/space/9909/30/mars.metric.02/). 

The thing is that it might seem simple enough, but as with many other problems, handling measurements is not always easy. And, I haven’t started yet with localizations problems. Weights, distances or velocity are expressed in many different ways around the planet. 

Foundation came to the rescue with `NSMeasurement` (bridged to `Measurement` for Swift). An `NSMeasurement` object represents a quantity and unit of measure. The `NSMeasurement` class provides a programmatic interface agnostic to units that allow clients to perform simple calculations. 

As expected, the interface of `Measurement` is a generic 

```
struct Measurement<unittype>
```

## Units
The available list of units is awesome, from the obvious `UnitArea` or `UnitLength` to the more technicals like `UnitElectricCharge`. 

It’s really easy to add new units: 

```
extension UnitLength {
    static var smoot: UnitLength {
        // 1 league = 5556 meters
        return UnitLength(symbol: "smoot", converter: UnitConverterLinear(coefficient: 1.70))
    }
}

let hardvardBridgeLength = Measurement(value: 364.4, unit: UnitLength.smoot)
hardvardBridgeLength.converted(to: UnitLength.meters) // 619.48 m
```

The [list of available units](https://developer.apple.com/documentation/foundation/units_and_measurement) is fascinating on its own.

## Measurements
Working with measurements is trivial due to the fact that they conform to equatable out of the box. 

```
// Really interesting https://paullaherty.com/2012/05/25/boeing-737-vs-toyota-prius-this-might-surprise-you/
// Units normalized fuel efficiency based on passenger seat miles.
let priusFuelEfficiency = Measurement(value: 200, unit: UnitFuelEfficiency.milesPerGallon)

let boeing737800FuelEfficiency = Measurement(value: 90, unit: UnitFuelEfficiency.milesPerGallon)

let mostEfficient = min(priusFuelEfficiency, boeing737800FuelEfficiency)

assert(mostEfficient == boeing737800FuelEfficiency)
```

The heavy lifting is performed by the API. And, there is no need for the compared measures to be expressed on the same unit. 

```
let riceGrainWeight = Measurement(value: 44, unit: UnitMass.centigrams)
let humanOvum = Measurement(value: 3.6, unit: UnitMass.micrograms)
assert(riceGrainWeight > humanOvum)
```

At the very least, every unit has an associated `symbol`. And, in most cases, they will inherit from `Dimension`, in which case they also have a *converter*. 


## User Facing Formatting

By default, magnitudes are printed in the defined unit

```
print(priusFuelEfficiency) // 200.0 mpg
print(boeing737800FuelEfficiency) // 2.6135 L/100km
```

But, that might not be ideal for user-facing strings. For those cases, the API introduces a new formatter, much like the date and the number ones. 

```
let distanceToTheMoon = Measurement(value: 384_400, unit: UnitLength.kilometers)

let formatter = MeasurementFormatter()
formatter.locale = Locale(identifier: "de_DE")
formatter.string(from: distanceToTheMoon) // 384.400 km
formatter.locale = Locale(identifier: "en_GB")
formatter.string(from: distanceToTheMoon) // 238,855.68 mi
formatter.locale = Locale(identifier: "zh_Hans") // Chinese (Simplified Han)
formatter.string(from: distanceToTheMoon) // 384,400公里
``` 

## Custom Units

Obviously, out of the box, we cannot mingle with measures of different types. 

```
let clorineMass = Measurement(value: 3.214, unit: UnitMass.grams)
let volume = Measurement(value: 1.0, unit: UnitVolume.liters)

let clorineDensity = clorineMass / volume

error: MyPlayground.playground:26:34: error: binary operator '/' cannot be applied to operands of type 'Measurement<unitmass>' and 'Measurement<unitvolume>'
let clorineDensity = clorineMass / volume
```

But, we can always overload `/`

```
func /(numerator: Measurement<UnitMass>, denominator: Measurement<UnitVolume>) -> Measurement<UnitConcentrationMass>? {
    let denominatorAsLiters = denominator.converted(to: UnitVolume.liters)
    guard denominator.value != 0 else {
        return nil
    }
    let numeratorAsGrams = numerator.converted(to: UnitMass.grams)
    let densityValue = numeratorAsGrams.value / denominatorAsLiters.value
    let densityUnit = UnitConcentrationMass.gramsPerLiter
    let density = Measurement(value: densityValue, unit: densityUnit)

    return density
}
``` 