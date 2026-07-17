---
layout: default
title: Core Architecture
description: GUNDAM User Guide Explanatory Documentation
---
# Core Architecture

## 1. Overall Structure of GUNDAM

GUNDAM is a configuration-driven fitting framework. A fit is mainly defined through YAML or JSON configuration files, while the input events are usually read from ROOT files. The configuration specifies which datasets are loaded, how events are selected and grouped into analysis samples, which parameters are allowed to vary, how systematic effects are applied to the Monte Carlo prediction, and how the prediction is compared with data.

A typical fit begins with the datasets defined in the configuration. GUNDAM reads the input ROOT files and makes the event-level information available to the analysis. The sample definitions then determine how those events are selected, weighted, and binned into histograms. These histograms form the data distributions and the Monte Carlo prediction used in the fit.

Fit parameters are managed separately from the event definitions. They may represent normalization factors, systematic uncertainties, spline parameters, oscillation parameters, or other quantities defined by the analysis. During the fit, the fitter or sampler changes the parameter values. These changes are applied to the Monte Carlo prediction through the Propagator, which updates the relevant event weights or systematic responses and rebuilds the prediction histograms.

After the prediction is updated, the Likelihood Interface compares it with the data. The resulting likelihood value is used by the fitter or sampler to choose the next parameter point. This process is repeated until the fit reaches a best-fit result or the requested sampling procedure is complete.

The main components discussed below are the Propagator, the Parameters Manager, Sample and Event Management, and Dataset Loading. Other components, such as the Likelihood Interface and the Fitter Engine or sampler, connect these parts to the likelihood calculation and fitting procedure.

---

## 2. Propagator

The Propagator applies the current fit parameter values to the Monte Carlo prediction. It is the main component that connects the parameter configuration to the samples and histograms used in the likelihood calculation.

When the fitter or sampler changes parameter values, the Propagator updates the relevant Monte Carlo events according to the configured dials and other parameter responses. A normalization parameter typically multiplies the weights of all affected events by the same factor. By contrast, a shape systematic changes event weights according to event properties, which can modify the relative contents of the histogram bins.

The Propagator relies on consistent connections between samples, parameters, systematic responses, and event weights. Defining a parameter is not enough by itself; the parameter must be connected to the intended effect on the Monte Carlo prediction. If this connection is missing or incorrect, the parameter will not influence the likelihood in the expected way.

After the Propagator updates the prediction histograms, those histograms are passed to the Likelihood Interface and compared with data. The resulting likelihood value is then used by the fitter or sampler to continue the fit.

---

## 3. Parameters Manager

The Parameters Manager defines and organizes the quantities that can vary during the fit. GUNDAM does not assign a fixed physical meaning to these parameters. Depending on the analysis, they may represent normalization factors, oscillation parameters, or systematic effects associated with the neutrino flux, interaction cross sections, and detector response. Other model parameters can also be included when they are defined and connected to the prediction in the analysis configuration.

In GUNDAM, parameters are organized into parameter sets. A parameter set contains a list of related parameters and may also specify a shared prior covariance matrix. The configuration can define parameter names, prior values, allowed ranges, step sizes, fixed or free states, and covariance information. Dial sets associated with the parameters define how parameter changes are translated into changes in event weights.

The Parameters Manager provides the current parameter values used by the fit. When the fitter or sampler proposes a new parameter point, these values are made available to the Propagator. A parameter affects the likelihood only when it is connected through a dial or another configured response that modifies the Monte Carlo prediction.

A clear parameter configuration is important because it defines the parameter space explored by the fit. The choice of prior values, parameter limits, fixed states, and covariance constraints can affect fit stability, uncertainty estimation, and the physical interpretation of the result.

---

## 4. Sample and Event Management

Sample and Event Management defines how loaded events are organized into analysis samples. A sample usually corresponds to an analysis channel, such as a beam mode, flavor category, event topology, signal region, background region, or control sample.

The sample configuration usually specifies which datasets can contribute to the sample, which selection cut is applied, which event or sample weight is used, and how the selected events are binned. A sample definition therefore determines which loaded events are included in the sample, which weights are applied to those events, and how the selected events are turned into histograms.

This stage connects the input datasets to the physics categories used in the analysis. The same loaded dataset can contribute to multiple samples if the configuration defines different selections or binning rules. For example, events from one Monte Carlo dataset can be separated into different flavor-like samples or control regions.

During the fit, Monte Carlo events remain associated with their configured samples. When parameter values change, the affected event weights or systematic responses are updated, and the corresponding sample histograms are rebuilt. Therefore, sample definitions directly control which distributions enter the likelihood calculation.

---

## 5. Dataset Loading

Dataset Loading defines where the input events come from and how GUNDAM should read them. In most analyses, the events are stored in ROOT TTrees or TChains. The dataset configuration specifies the input ROOT files, the tree name, the nominal event weight, and the variables or branch mappings needed later for selections, weights, binning, and systematic variations.

A dataset definition also tells GUNDAM how the events should be treated in the fit. For example, a dataset may represent Monte Carlo, real data, Asimov data, or toy data. For Monte Carlo datasets, the configuration may also enable systematic reweighting or other variations that affect the prediction during the fit.

Dataset Loading makes the configured event source available to the rest of the analysis. After the events are loaded, the sample configuration determines which events enter each sample, how they are weighted, and how they are binned. For example, one loaded Monte Carlo dataset may later be divided into numu-like and nue-like samples using different selection cuts and binning definitions.

Because later stages depend on the loaded events, the dataset configuration must be consistent with the ROOT files. Incorrect file paths, tree names, branch mappings, or nominal weight formulas can affect sample construction, prediction histograms, and the final likelihood comparison.

---

[Back to Documentation Overview](../README.md)
