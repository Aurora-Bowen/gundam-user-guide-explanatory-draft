---
layout: default
title: Applications
description: User-Facing Applications and Utilities in GUNDAM
---

## 1. Applications Overview

The Applications layer provides the main user-facing entry points to GUNDAM. It includes the primary programs used to run fits or propagate fit results, a set of utility applications for configuration handling and result inspection, internal output components for histograms and event trees, and an optional Python interface.

At the highest level, the Applications section can be understood through four main functional areas. The first is the pair of primary applications, `gundamFitter` and `gundamCalcXsec`, which perform the main analysis tasks. The second is a collection of utility applications used to unfold configurations, compare setups, inspect fit outputs, compare fit results, and extract stored plots. The third is the output and visualization layer, which describes how GUNDAM produces ROOT histograms, canvases, and event trees. The fourth is the Python interface, which gives programmatic access to selected parts of the C++ engine.

Not every item described here is a standalone executable. `PlotGenerator` and `EventTreeWriter` are internal components used by the likelihood workflow. By contrast, tools such as `gundamFitReader` or `gundamPlotExtractor` are command-line applications intended to work with GUNDAM output files. The Python interface is an optional binding rather than a normal executable application.

---

## 2. Primary Applications: `gundamFitter` and `gundamCalcXsec`

### `gundamFitter`

`gundamFitter` is the main executable for likelihood-based fits in GUNDAM. It reads the analysis configuration, constructs the statistical model, loads the configured datasets and samples, applies systematic variations through the propagation machinery, and runs the selected minimization or sampling algorithm.

A typical workflow is:

```text
Configuration files
        ↓
Configuration unfolding and overrides
        ↓
FitterEngine configuration
        ↓
Dataset and model initialization
        ↓
Pre-fit evaluation and diagnostics
        ↓
Minimization or sampling
        ↓
Post-fit evaluation
        ↓
ROOT output
```

The main input is a YAML or JSON configuration file. Additional override files or command-line overrides can be used to change selected settings without directly editing the original configuration. Before the fit begins, GUNDAM resolves the referenced configuration files, applies the requested overrides, and builds the final configuration used at runtime. The fully unfolded configuration is then stored in the output ROOT file together with runtime and build information, which helps preserve the exact state of the fit setup.

`gundamFitter` supports real data, Asimov data, and toy data. Real data is used by default. In Asimov mode, the model prediction is used as the data sample. In toy mode, a generated toy dataset is selected, optionally with a specified toy index. The selected data mode is applied before the datasets and samples are initialized.

The runtime workflow is divided into three main stages. In the `configure()` stage, GUNDAM interprets the fitter configuration and creates the required components, such as the minimizer, likelihood interface, propagators, plotting system, event writer, and parameter scanner. In the `initialize()` stage, the application loads datasets and events, builds model and data samples, initializes parameters and internal caches, and prepares the output structure. In the `fit()` stage, the application evaluates the pre-fit state, performs any enabled scans or diagnostic calculations, runs the chosen minimization or sampling method, and records the resulting post-fit state.

This separation is important in practice because a configuration may be syntactically valid but still fail during initialization if an input file, tree, branch, selection, or sample definition cannot actually be loaded.

`gundamFitter` also provides several levels of dry run. A hyper dry run unfolds the configuration and writes basic application information, but does not create the fitting engine. A super dry run creates and configures the fitting engine, but does not load datasets or initialize the fit. A full dry run performs initialization and enabled pre-fit calculations, but stops before minimization. These dry-run modes are useful for checking different parts of the workflow, from basic configuration handling to actual dataset loading and pre-fit object construction.

The fitting or sampling algorithm is selected in the configuration. Current workflows include ROOT-based minimization and `SimpleMCMC`. For ROOT minimization, Hesse is the default post-fit error calculation, while Minos may be selected instead. These are alternative error procedures rather than two mandatory consecutive steps. Error calculations, scans, model-variation studies, plots, and event-tree outputs are all optional and depend on the configuration, command-line settings, and the runtime outcome.

The output is a ROOT file that may contain the unfolded configuration, runtime metadata, initial parameter states, prior covariance matrices, pre-fit and post-fit likelihood summaries, sample histograms, minimizer status, fitted parameter values, covariance and correlation matrices, parameter scans, event trees, and ROOT canvases. The exact output depends on the enabled features and on whether the fit completed successfully.

### `gundamCalcXsec`

`gundamCalcXsec` is an advanced, analysis-specific application for propagating a configured fit state into cross-section quantities. It is not a universal cross-section calculator with a single built-in formula. The signal definition, binning, normalization conventions, and physical interpretation are supplied by the analysis configuration.

A typical workflow is:

```text
Fitter configuration or fit output
                +
Cross-section configuration
                ↓
Build cross-section samples and normalizations
                ↓
Load a pre-fit or post-fit parameter state
                ↓
Generate parameter toys
                ↓
Propagate systematic responses
                ↓
Normalize the cross-section bins
                ↓
Build covariance and correlation matrices
```

The main inputs are a cross-section-specific configuration, a fitter ROOT output or fitter configuration, and the requested number of toys. In the usual post-fit workflow, the input is a ROOT file produced by `gundamFitter`. The application reads the unfolded fitter configuration and the stored post-fit parameter state from that file, then combines them with the cross-section configuration. However, a previous fit output is not strictly required. If the supplied fitter input is a configuration file rather than a ROOT file, the application can operate in pre-fit mode using the original parameter priors and covariance definitions.

To perform its work, `gundamCalcXsec` creates and configures a `FitterEngine` so that it can reuse the same dataset, parameter, dial, propagation, and likelihood machinery used by the main fitter. It does not run a new minimization or sampling procedure. Instead, it uses the configured engine components to propagate parameter variations into the cross-section observables.

In the normal post-fit workflow, the application loads the fitted parameter state and uses the fitted values as the central point. Parameter toys are then generated from the post-fit covariance stored in the Hesse output. This means that the current post-fit workflow expects the Hesse covariance in the original parameter space. A Minos-only result or an MCMC posterior is not automatically used as an alternative covariance source. In pre-fit mode, the configured prior covariance is used instead.

The physical normalization is controlled by the cross-section configuration. The application can combine parameter-set-dependent normalization factors, numerical normalization values, statistical variations of those values, and phase-space bin widths. Quantities such as POT, integrated flux, target mass, number of targets, and detector volume are not represented by universal built-in fields with fixed meanings. They must be represented consistently by the analysis configuration or already be included in the sample definition.

For each toy, the enabled parameters are thrown from the selected covariance matrix, propagated through the dial system, and used to update the affected event weights and sample contents. The application may also apply finite-Monte-Carlo and statistical fluctuations. It then applies the configured normalizers, divides by the relevant phase-space bin volume, and stores the resulting cross-section values together with the thrown parameter values. The central value can be taken from the mean of the toy distribution or from a single evaluation at the best-fit point, depending on the selected behavior.

The output ROOT file contains the main cross-section products, including toy results, best-fit cross-section results, covariance and correlation matrices, ROOT histograms and canvases, and in some workflows event-tree output. Because it depends strongly on the analysis configuration and expected fit-result structure, `gundamCalcXsec` is best treated as an advanced application rather than the first tool introduced to new users.

---

## 3. Utility Applications

GUNDAM also provides a set of utility applications that support configuration handling, fit inspection, fit comparison, and plot export. These tools are not responsible for running the main fit itself, but they are important for validating configurations, understanding outputs, and organizing analysis results.

### Configuration Utilities

`gundamConfigUnfolder` resolves a configuration into one complete JSON representation. It reads a YAML or JSON configuration, recursively opens referenced configuration files, expands environment variables in the referenced paths, and applies any requested override files. Without an output path, the unfolded configuration is printed to the terminal, and it can also be written to a JSON file. Even if the original input is YAML, the unfolded output is written as JSON. This tool is especially useful for checking what GUNDAM will actually read after all includes and overrides have been applied.

`gundamConfigCompare` recursively compares two configurations and displays their differences in the terminal. Nested objects are compared field by field. When an array contains entries with a `name` field, the tool can match those entries by name instead of relying only on their order. Arrays without named entries are compared positionally. Separate override files can also be applied to each input before the comparison. This is useful when comparing different versions of an analysis configuration or checking whether two configurations that look different actually produce the same effective setup.

`gundamInputZipper` scans an unfolded configuration for referenced local files, copies the recognized files into a structured output directory, and rewrites the corresponding configuration paths. The collected directory can optionally be compressed into a `.tar.gz` archive. Despite the application name, the resulting archive is not a `.zip` file. This tool is useful for preparing local configuration inputs for transfer or preservation, although the resulting package is not guaranteed to contain every external runtime dependency.

### Fit Inspection and Comparison Utilities

`gundamFitReader` provides a command-line summary of one or more GUNDAM fit outputs. It allows users to inspect important fit information without manually opening the ROOT file. The reported information may include the original command line, host and user information, runtime metadata, GUNDAM and ROOT build information, prior covariance properties, pre-fit and post-fit likelihood summaries, parameter values, parameter uncertainties, minimization status, convergence information, Hesse status, and parameter correlations. It is useful for quickly checking whether a fit converged and what the main fitted results were.

`gundamFitCompare` compares corresponding objects from two fit inputs and writes comparison histograms and canvases. The comparison can include sample histograms, parameter values, parameter uncertainties, normalized uncertainties, and likelihood scans. The two inputs can be different fit files, or the same file can be supplied twice when comparing two states such as pre-fit and post-fit. It is therefore a general two-input comparison tool rather than a tool limited only to pre-fit-versus-post-fit comparisons.

`gundamFitPlot` is a specialized utility for studying how post-fit parameter uncertainties change across a series of fits. Each fit is associated with a user-provided `xValue`, which may represent exposure, sample size, running time, number of events, or another analysis-dependent quantity. The application reads the post-fit Hesse uncertainties from each fit and constructs ROOT `TGraph` objects showing the uncertainty of each parameter as a function of `xValue`. Its name can be misleading, because it is not a general program for drawing fitted sample distributions.

### Plot Export and Additional Utility Tools

`gundamPlotExtractor` converts stored ROOT canvases into external image files. It recursively searches the ROOT directory structure, finds each `TCanvas`, and saves it into a corresponding output directory. PDF is the default format, while other ROOT-supported formats such as PNG can also be requested. This makes it possible to export large collections of plots without manually opening every ROOT directory.

`gundamContinue` is an installed script rather than a compiled C++ executable. It supports continuation or restart workflows based on previous fit setups or outputs, and should be understood as a workflow-specific helper rather than a standard standalone application.

`gundamRoot` is a customized ROOT interactive shell intended mainly for development and debugging. It prepares an interactive ROOT environment with selected headers and settings preloaded, but it is not a complete front end to all fitter and propagator libraries. For normal analysis work, the dedicated GUNDAM applications are the preferred interface.

---

## 4. Output and Visualization

The output and visualization part of GUNDAM is mainly implemented through the internal components `PlotGenerator` and `EventTreeWriter`, together with the external utility `gundamPlotExtractor`.

`PlotGenerator` is an internal component of the likelihood workflow rather than a standalone command-line application. It reads the plotting configuration, creates the requested histograms, fills them from the model and data samples, and writes ROOT histograms and canvases. A histogram can use explicitly configured binning, the binning of a sample, or the binning associated with a selected observable. Model histograms may be split according to configured event variables, and variable dictionaries can map the values of such variables to readable labels, colors, and fill styles. `PlotGenerator` can build stacked model histograms, create legends, and generate comparison plots between a reference prediction and a varied prediction. These objects are used by several diagnostic and visualization workflows.

`EventTreeWriter` is another internal component. It exports the current event state to ROOT `TTree` objects. The event tree may contain the current parameter-dependent event weight, the original input-tree weight, the sample-bin index, the dataset index, input entry identifiers, and loaded scalar variables that can be represented as ROOT leaves. The state stored in the event tree depends on when the writer is called, so it may correspond to a pre-fit state, a post-fit state, or a state used during another application such as `gundamCalcXsec`. Event-tree output is useful for validation and downstream studies, but it can significantly increase file size.

Although the configuration contains a `writeDials` option, the standard application workflow does not currently guarantee that per-event dial-response graphs will be produced. That behavior should therefore not be presented as a standard output feature.

`PlotGenerator` creates ROOT histograms and canvases but does not directly produce PNG or PDF files. External image export is handled by `gundamPlotExtractor`. The visualization chain is therefore:

```text
PlotGenerator
        ↓
ROOT histograms and canvases
        ↓
gundamPlotExtractor
        ↓
PDF, PNG, or other image files
```

This distinction is important. `PlotGenerator` is responsible for creating and storing the plotting objects inside the ROOT output, while `gundamPlotExtractor` is responsible for converting those stored canvases into external files for viewing or presentation.

---

## 5. Python Interface

The Python interface is an optional `pybind11` module that exposes selected parts of the GUNDAM C++ API. It is disabled by default and must be enabled during the build with:

```bash
-D WITH_PYTHON_INTERFACE=ON
```

The module is imported using:

```python
import GUNDAM
```

The bindings provide selected functionality from `ConfigBuilder`, `ConfigReader`, `GundamApp`, `FitterEngine`, `LikelihoodInterface`, `Propagator`, `ParametersManager`, and `MinimizerBase`. The exposed methods represent only part of the full C++ interface.

A Python-driven fit follows the same basic lifecycle as the C++ application. The configuration is loaded, a `FitterEngine` is created and configured, the data behavior is selected, the engine is initialized, and the fit is started. The data behavior must be set before initialization because the initialization stage creates and loads the data samples.

A minimal workflow is conceptually:

```text
Read or build the configuration
        ↓
Create and configure FitterEngine
        ↓
Set the data behavior
        ↓
Initialize the engine
        ↓
Run the fit
```

A complete Python workflow must also create a `GundamApp`, open an output ROOT file, and assign a save directory to the `FitterEngine`. A minimal example is:

```python
import GUNDAM

GUNDAM.setLightOutputMode(True)
GUNDAM.setNumberOfThreads(2)

builder = GUNDAM.ConfigUtils.ConfigBuilder("config.yaml")
reader = GUNDAM.ConfigUtils.ConfigReader(builder.getConfig())

reader.defineField(
    GUNDAM.ConfigUtils.ConfigReader.FieldDefinition(
        "fitterEngineConfig"
    )
)

engine_config = reader.fetchValueConfigReader(
    "fitterEngineConfig"
)

app = GUNDAM.GundamApp("Python fitter")
app.openOutputFile("python-fit.root")
app.writeAppInfo()

engine = GUNDAM.FitterEngine()
engine.setSaveDir(app, "FitterEngine")
engine.setConfig(engine_config)
engine.configure()

engine.getLikelihoodInterface().setForceAsimovData(True)

engine.initialize()
engine.fit()
```

The Python interface is still preliminary. It does not provide full command-line parity with `gundamFitter`, and it does not expose every data-mode control or every C++ method. It should therefore be presented as an optional programmatic interface for advanced users rather than as the default way to run GUNDAM.

---

[Back to Documentation Overview]({{ '/' | relative_url }})
