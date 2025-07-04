# Representing Measures

**Stage**: 1

**Champion**: Ben Allen [@ben-allen](https://github.com/ben-allen)

**Author**: Ben Allen [@ben-allen](https://github.com/ben-allen)

## Goals and needs

Modeling measurements with a precision is useful for any task that involves measurements from the physical world. It can also be useful for other types of measurement; for example, (measurements of) currency amounts.

We propose to create a new object for representing measurements, for producing formatted string representations of measurements.

Common user needs that can be addressed by a robust API for measurements include, but are not limited to:

* The need to keep track of the precision of measured values. A measurement value represented with a large number of significant figures can imply that the measurements themselves are more precise than the apparatus used to take the measurement can support.

* The need to represent currency values. Often users will want to keep track of money values together with the currency in which those values are denominated.

* The need to format measurements into string representations

## Description

We propose creating a new `Amount` API, whose values will be immutable and have the following properties:

Note: ⚠️  All property/method names up for bikeshedding.

* `currency` (String or undefined): The currency code with which number should be understood (with *undefined* indicating "none supplied")
* `unit` (String or undefined): The unit of measurement with which number should be understood (with *undefined* indicating "none supplied")
* `significantDigits` (Number): how many significant digits does this value contain? (Should be a positive integer)
* `fractionalDigits` (Number): how many digits are required to fully represent the part of the fractional part of the underlying mathematical value. (Should be a non-negative integer.)

#### Precision

A big question is how we should handle precision. When constructing an Amount, both the significant digits and fractional digits are recorded.

### Constructor

* `new Amount(value[, options])`. Constructs an Amount with the mathematical value of `value`, and optional `options`.

The object prototype would provide the following methods:

* `toString([ opts ])`: Returns a string representation of the measurement with any unit put in brackets (e.g., `"1.23[kg]`).
* `toLocaleString(locale[, options])`: Return a formatted string representation appropriate to the locale (e.g., `"1,23 kg"` in a locale that uses a comma as a fraction separator)
* `with(opts)`: Represent the same underlying mathematical value, possibly with different precision.

### Examples

Let's construct an Amount, query its properties, and render it.
First, we'll work with a bare number (no unit):

```js
let a = new Amount("123.456");
a.fractionDigits; // 3
a.significantDigits; // 6
a.with({ fractionDigits: 4 }).toString(); // "123.4560"
```

Notice that "upgrading" the precision of a Measurement essentially appends trailing zeroes to the number.

Here's an example with units:

```js
let a = new Amount("42.7", { unit: "kg" });
a.toString(); // "42.7[kg]"
a.toString({ numberOnly: true }); // "42.7"
```

#### Rounding

If one downgrades the precision of a Measurement, rounding will occur. (Upgrading just adds trailing zeroes.)

```js
let a = new Amount("123.456");
a.with({ significantDigits: 5 }).toString(); // "123.46"
```

By default, we use the round-ties-to-even rounding mode, which is used by IEEE 754 standard, and thus by Number and [Decimal](https://github.com/tc39/proposal-decimal). One can specify a rounding mode:

```js
let b = new Amount("123.456");
a.with({ significantDigits: 5, roundingMode: "truncate" }).toString(); // "123.45"
```

## Units

A core piece of functionality for the Measure proposal is to support units (`mile`, `kilogram`, etc.) as well as currency (`EUR`, `USD`, etc.). One can construct these


## Related but out-of-scope features

Amount is intended to be a small, straightforwardly implementable kernel of functionality for JavaScript programmers that could perhaps be expanded upon in a follow-on proposal if data warrants. Some features that one might imagine belonging to Amount are natural and understandable, but are currently out-of scope. Here are the features:

### Mathematical operations

Below is a list of mathematical operations that one could consider supporting. However, to avoid confusion and ambiguity about the meaning of propagating precision in arithmetic operations, *we do not intend to support mathematical operations*. A natural source of data would be the [CLDR data](https://github.com/unicode-org/cldr/blob/main/common/supplemental/units.xml) for both our unit names and the conversion constants are as in CLDR. One could conceive of operations such as:

* raising an Amount to an exponent
* multiply/divide an Amount by a scalar
* Add/subtract two Amounts of the same dimension
* multiply/divide an Amount by another Amount
* Convert between scales (e.g., convert from grams to kilograms)

could be imagined, but are out-of-scope in this proposal.
This proposal focuses on the numeric core that future proposals can build on.

### Unit conversion

One might want to convert an Amount from one unit (e.g., miles) to another (e.g., kilometers).
We envision that functionality to be potentially introduced as part of the [Smart Units](https://github.com/tc39/proposal-smart-unit-preferences) proposal.
This implies that converting from unit to another is not supported,
as well as converting amounts between scales (e.g., converting grams to kilograms).
Our work here in this proposal is designed to provide a foundation for such ideas.

### Compount units

Some units can be combined. In the US, it is common to express the heights of people in terms of feet and inches, rather than a non-integer number of feet or a "large" number of inches. For instance, one would say commonly express a height of 71 inches as "5 feet 11 inches" rather than "71 inches" or "5.92 feet". Thus, one would naturally want to support "foot-and-inch" as a compound unit, derivable from a measurement in terms of feet or inches. Since this is closely related to unit conversion, we prefer to see this functionality in Smart Units.

## Related/See also

* [Smart Units](https://github.com/tc39/proposal-smart-unit-preferences) (mentioned several times as a natural follow-on proposal to this one)
* [Decimal](https://github.com/tc39/proposal-decimal) for exact decimal arithmetic
* [Keep trailing zeroes](https://github.com/tc39/proposal-intl-keep-trailing-zeros) to ensure that when Intl handles digit strings, it doesn't automatically strip trailing zeroes (e.g., silently normalize "1.20" to "1.2").
