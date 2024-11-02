# Representing Measures

**Stage**: 1

**Champion**: Ben Allen [@ben-allen](https://github.com/ben-allen)

**Author**: Ben Allen [@ben-allen](https://github.com/ben-allen)


# Goals and needs

Modeling units of measure is useful for any task that involves measurements from the physical world. It can also be useful for other types of measurement; for example, measurements of currency amounts. 

Common user needs that can be addressed by a robust API for measurements include, but are not limited to:

* The need to convert measurements from one scale to another

* The need to format measurements into string representations

* Related to both of the above, the need to localize measurements. This can be relatively straightforward, as in the case of converting temperature measurements between Celsius and Fahrenheit, or Celsius and Kelvin. It can be more complex, though: notably, many locales represent measurements of certain types of object using mixed units
    - For example, in the United States it is customary to represent the heights of people in feet and inches.

* The need to represent and manipulate derived units, such as square meters for area and meters cubed for volume

* The need to represent and manipulate compound units, such as velocity expressed in kilometers per hour or density expressed in unit of mass per unit of volume

We propose to create a new object for representing measurements, for producing formatted string representations of measurements, and for converting measurements between scales.

* The need to keep track of the precision of measured values. A measurement value represented with a large number of significant figures can imply that the measurements themselves are more precise than the apparatus used to take the measurement can support.

* The need to represent currency values. Often users will want to keep track of money values together with the currency in which those values are denominated.
    - As in the "feet and inches" example, sometimes it is useful to present currency values with the major and minor units separated, as in "19 dollars and 17 cents." See: [Design for currency and unit inputs that carry their values ](https://github.com/tc39/ecma402/issues/911#issuecomment-2238619851)

* The need to define custom units.
    - For example, CLDR includes units for velocity and acceleration, but none of the higher-order derivatives of position with respect to time.

# Description

We propose creating a new `Measure` API, with the following properties.

Note: ⚠️  Serious questions remain about how the API should handle mixed units. The current version allows for mixed unit output, but *not* mixed unit input. This simplifies the API, at the cost of certain infelicities. The decision to disallow mixed unit input is subject to change if clear use cases can be identified.

Note: ⚠️  All property/method names up for bikeshedding.

* `unit`, a String representing the measurement unit. This could be expressed using the names used by [CLDR's units.xml](https://github.com/unicode-org/cldr/blob/main/common/supplemental/units.xml). The API design below presumes that these names are used. For example, if a user wanted to use the `convertTo` method to convert a length measurement to feet and inches, they would provide the string `"foot-and-inch"` as the value of the `unit` argument.

* `value`, the numerical value of the measurement.

* `precision`, a number indicating the precision of the measurement. This precision is (provisionally) represented in terms of the number of fractional digits displayed.

Note: ⚠️  It may be appropriate to instead represent precision in terms of number of significant digits. Feedback on this matter is desired.

Constructor:

* `Measure(value, unit, precision)`. Constructs a Measure with `value` as the numerical value of the Measure and `unit` as the unit of measurement, with the optional `precision` parameter used to specify the precision of the measurement. In the case of `unit` values indicating mixed units, the `value` is given in terms of the quantity of the *largest* unit.

The object prototype would provide the following methods:

* `convertTo(unit, precision)`. This method returns a Measure in the scale indicated by the `unit` parameter, with the value of the new Measure being the value of the Measure it is called on converted to the new scale. The `precision` parameter is optional.

* `split()`. This method returns the measurement represented as an array of objects. The returned array for a Measure given in non-mixed units would contain one object with the following properties:

    - `value`, representing the numerical value
    - `unit`, representing the unit of measurement.
    - `precision`, representing the precision of the measurement.

The returned array for a Measure given in mixed units would contain one element for each component of the Measure. The `precision` property would only be present in the object representing the smallest unit.

* `convertToLocale(locale, usage)`. This method returns a Measure in the customary scale and at the customary precision for `locale`. The optional `usage` parameter can be used to indicate that the Measure should use the (potentially idiosyncratic) locale-specific customary unit of measurement for measurements of specific types of things. If, for example, the value was a measurement of a person's height, the value of `usage` would be (following the CLDR names) `"person-height"`. If the measurement is of a distance traveled by road, the value of `usage` would be `"road"`.

* `toLocaleString(locale, usage)`. This method returns an appropriately formatted string for the locale given by `locale` and the usage given by `usage`, with `usage` being an optional parameter.

* `toString()`. This method returns a string representation of the unit.

# Examples

```js

let m = new Measure(1.8, "meter");
m.convertTo('foot', 2);
// Returns a new Measure with the following properties:
// `{value: 5.905511811023621, unit: "foot", precision: 2}`
m.toString();
// "5.91 feet"
m.localeConvert("en-CA", "person-height")
// `{value: 5.905511811023621, unit: "foot-and-inch", precision: 2}`
m.toLocaleString("en-CA", "person-height")
// "5 feet 11 inches"
```
