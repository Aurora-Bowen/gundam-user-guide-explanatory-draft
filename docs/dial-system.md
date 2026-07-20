---
layout: default
title: Dial System
description: Systematic Variations in GUNDAM
---

## 1. Dial System: Systematic Variations

A **Dial** represents a response function that modifies an event's weight based on the state of one or more model parameters. The Dial System translates changes in fit parameters into changes in the Monte Carlo prediction. In this way, the Dial System connects the parameter space explored by the fitter or sampler to the event weights used by the Propagator.

A dial may represent a simple normalization change or a response that varies with the current parameter values and the properties of the affected events. For example, a Norm dial can scale all events in a selected category by the same factor, while Spline or Surface dials can provide parameter-dependent responses that differ across configured event categories or event-level inputs.

Depending on the configuration, a dial may apply to all events in a sample, to a selected category of events, or separately to individual events according to their event-level variables. More than one dial response may contribute to the weight of the same event when several systematic effects are enabled. The configuration therefore needs to identify the parameters that control each response, the events to which the response applies, and the dial definition or external response information used to calculate the weight.

When the fitter or sampler proposes a new parameter point, the Dial System receives the updated parameter values and reevaluates the affected responses. The resulting dial weights are combined with the nominal event weight and any other configured weights. The Propagator then uses the updated event weights to rebuild the Monte Carlo histograms that enter the likelihood calculation.

---

## 2. Dial Types and Definitions

GUNDAM provides several dial types for representing different systematic responses. The appropriate type depends on the form and dimensionality of the available response information.

A **Norm dial** applies a common multiplicative response to the affected events. It is suitable for a systematic uncertainty that changes the overall normalization or event rate without introducing a more detailed shape dependence.

A **Graph dial** describes a one-dimensional response using values defined at a series of parameter points. The response between these points is obtained by linear interpolation.

A **Spline dial** also describes a one-dimensional response but uses smooth interpolation between the supplied points. GUNDAM provides compact, uniform, general, monotonic, and ROOT-based spline variants. These variants differ in their treatment of knot spacing, slope information, and interpolation behavior.

A **Surface dial** represents a response that depends jointly on two parameters and is defined on a two-dimensional response grid.

A **Formula dial** calculates the response from an analytical expression. A **Compiled dial** instead obtains the response from external C++ code and can be used when the calculation cannot be conveniently represented by a standard Graph, Spline, Surface, or Formula dial.

A **Tabulated dial** obtains its response from a pre-calculated lookup table. A **Kriged dial** constructs a response from a weighted combination of information associated with several table entries and is intended for more complex multi-dimensional response models.

The nominal parameter point should reproduce the nominal prediction expected by the analysis, which commonly corresponds to a dial response of one.

---

## 3. Dial Factories and Tabulated/Kriged Dials

Dial factories support systematic models that require external code, shared response tables, or specialized event-level initialization. Rather than defining the complete response directly in the main configuration, a factory loads or initializes the required model, prepares the information needed to evaluate the response, and creates the dial objects used during the fit.

A **Tabulated dial factory** provides responses through a lookup table associated with the current parameter values and relevant event variables. Individual dials obtain their responses by reading or interpolating the corresponding table entries. This approach is useful when many events use a common response table but occupy different positions within it according to their event-level variables or kinematics.

A **Kriged dial factory** follows a similar structure but allows an event response to be constructed from a weighted combination of several table entries. It is intended for more complex multi-dimensional response models in which a simple sequential interpolation is not sufficient.

### Event-by-Event and Binned Dials

GUNDAM distinguishes between event-by-event dial collections and collections in which dial objects are shared between events. This distinction determines how the dial definitions are associated with the events loaded by the DataDispenser.

For an **event-by-event collection**, each event has its own `DialBase` object. This is appropriate when the systematic response is specific to the individual event, for example when it is defined using event-level information or kinematics.

For a **binned collection**, events assigned to the same configured bin or category can share a common `DialBase` object. A separate dial object is therefore not required for every event when the same response definition applies to a group of events.

The main difference between the two approaches is how the response definitions are stored and associated with events. In both cases, the current parameter values are used to calculate dial responses, and the resulting weights are applied to the affected events.

Factory-based dials should normally be used only when the analysis already provides the corresponding external implementation. For standard normalization, graph, spline, or surface responses, a regular dial definition is usually simpler to configure and validate.

---

## 4. Spline and Surface Interpolation

Spline and Surface dials are evaluated internally by GUNDAM. Users select the appropriate dial type and provide the corresponding response points or response grid, while GUNDAM interpolates the response at the current parameter values.

For a one-dimensional Spline response, GUNDAM identifies the interval containing the current parameter value and evaluates the response from the surrounding knots. Different spline implementations are available depending on the knot spacing, the provided slope information, and the desired interpolation behavior.

### Uniform Spline

The uniform spline implementation uses cubic interpolation with equally spaced knots. Because the distance between neighboring knots is constant, the interval containing the current parameter value can be determined directly from the lower boundary and the common step size. The response is then calculated using the values and slopes defined at the two ends of that interval.

### General Spline

The general spline implementation supports non-uniform knot spacing. Each knot position and its associated slope are provided explicitly, allowing the response points to be placed at irregular parameter values. GUNDAM locates the relevant neighboring knots and uses their positions, response values, and slopes to evaluate the interpolated response.

### Compact Spline

The compact spline implementation uses a Catmull-Rom spline. Instead of requiring slopes to be supplied separately, it derives the local interpolation behavior from neighboring response values. This reduces the amount of stored information while still providing a smooth cubic response between uniformly spaced knots.

### Monotonic Spline

The monotonic spline implementation applies the Fritsch-Carlson condition when determining the slopes at the knots. If the supplied response values are monotonic, the slopes are adjusted to preserve that behavior between neighboring knots. This prevents cubic interpolation from introducing overshooting or artificial oscillations into the systematic response.

### Surface Interpolation

For a two-dimensional Surface response, GUNDAM uses bilinear or bicubic interpolation on a response grid. Bilinear interpolation evaluates the response from the four grid points surrounding the current parameter position, using linear interpolation along the two grid directions.

Bicubic interpolation provides a smoother variation across the response surface using a two-dimensional Catmull-Rom spline. It first performs interpolation along the X-axis at four neighboring Y-positions to obtain four intermediate values. These values are then interpolated along the Y-axis to produce the final response.

### Extrapolation and Response Range

When extrapolation is disabled, the dial input is restricted to the available response range before the weight is evaluated. This prevents the interpolation routine from evaluating the response outside the region covered by the supplied data.

The selected dial type should match the dimensionality, knot spacing, and format of the supplied response information. For example, a uniform or compact spline requires regularly spaced response points, while a general spline can represent irregular knot positions and explicitly provided slopes.

The configured parameter limits should also be compatible with the available response range. If the dial type, knot positions, parameter range, or response data are inconsistent, the calculated event weights may not represent the intended systematic variation.

---

[Back to Documentation Overview]({{ '/' | relative_url }})
