# Representing Measures

**Stage**: 1

**Champion**: Ben Allen [@ben-allen](https://github.com/ben-allen)

**Author**: Ben Allen [@ben-allen](https://github.com/ben-allen)

## Goals and needs

Modeling measurements with a precision is useful for any task that involves measurements from the physical world. It can also be useful for other types of measurement; for example, (measurements of) currency amounts.

We propose to create a new object for representing measurements, for producing formatted string representations of measurements.

Common user needs that can be addressed by a robust API for measurements include, but are not limited to:

* The need to convert measurements from one scale to another

* The need to format measurements into string representations

* The need to keep track of the precision of measured values. A measurement value represented with a large number of significant figures can imply that the measurements themselves are more precise than the apparatus used to take the measurement can support.

## Description

We propose creating a new `Amount` API, with the following properties.

Note: ⚠️  All property/method names up for bikeshedding.

* `value` (String), the numerical value of the measurement, representing an exact mathematical value;
* `significantDigits` (Number): how many significant digits does this value contain? (Should be a positive integer)
* `fractionalDigits` (Number): how many digits are required to fully represent the part of the fractional part of the underlying mathematical value. (Should be a non-negative integer.)

#### Precision

A big question is how we should handle precision. When constructing an Amount, both the significant digits and fractional digits are recorded.

### Constructor

* `new Measure(value)`. Constructs a Measure with the digit String `value` as the numerical value of the Measure.

The object prototype would provide the following methods:

* `toString()`. This method returns a string representation of the unit.
* `with(opts)`: Represent the same underlying mathematical value, possibly with different precision.

### Examples

```js
let a = new Amount("123.456");
a.fractionDigits; // 3
a.significantDigits; // 6
a.with({ fractionDigits: 4 }).toString(); "123.4560"
```

## Related but out-of-scope features

Measure is intended to be a small, straightforwardly implementable kernel of functionality for JavaScript programmers that could perhaps be expanded upon in a follow-on proposal if data warrants. Some features that one might imagine belonging to Measure are natural and understandable, but are currently out-of scope. Here are the features:

### Mathematical operations

Below is a list of mathematical operations that one could consider supporting. However, to avoid confusion and ambiguity about the meaning of propagating precision in arithmetic operations, *we do not intend to support mathematical operations*. A natural source of data would be the [CLDR data](https://github.com/unicode-org/cldr/blob/main/common/supplemental/units.xml) for both our unit names and the conversion constants are as in CLDR. One could conceive of operations such as:

* raising a Measurement to an exponent
* multiply/divide a Measurement by a scalar
* Add/subtract two Measurements of the same dimension
* multiply/divide a Measurement by another Measurement
* Convert between scales (e.g., convert from grams to kilograms)

could be imagined, but are out-of-scope in this proposal. This proposal focuses on on the numeric core that future proposals can build on.

### Units

One naturally might want to include units (`mile`, `kilogram`, etc.) but we currently envision that functionality being part of the [Smart Units](https://github.com/tc39/proposal-smart-unit-preferences) proposal. This implies that converting from unit to another is not supported, as well as converting measurements between scales (e.g., converting grams to kilograms). Our work is designed to be a foundation for such ideas.

## Related/See also

* [Smart Units](https://github.com/tc39/proposal-smart-unit-preferences) (mentioned several times as a natural follow-on proposal to this one)
* [Decimal](https://github.com/tc39/proposal-decimal) for exact decimal arithmetic
* [Preserve trailing zeroes](https://github.com/tc39/proposal-intl-keep-trailing-zeros) to ensure that when Intl handles digit strings, it doesn't automatically strip trailing zeroes (e.g., silently normalize "1.20" to "1.2").
