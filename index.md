---
layout: default
title: GUNDAM User Guide
description: Draft Explanatory Documentation
---

# Documentation Overview

This website contains draft explanatory documentation for the GUNDAM User Guide.

The documentation introduces the roles, relationships, and main behavior of important GUNDAM components without focusing on detailed implementation.

## Documentation Pages

### 1. [Core Architecture](docs/core-architecture.md)

### 2. [Dial System](docs/dial-system.md)

### 3. [Statistical Inference](docs/statistical-inference.md)

### 4. [Applications](docs/applications.md)

## Workflow Diagrams

The central vertical path in each diagram represents the main workflow.  
The side branches show related inputs, configuration choices, methods, tools, or outputs.

### Core Architecture

<p align="center">
  <a href="{{ '/assets/images/gundam-core-architecture.png' | relative_url }}">
    <img
      src="{{ '/assets/images/gundam-core-architecture.png' | relative_url }}"
      alt="GUNDAM Core Architecture workflow diagram"
      width="760"
    >
  </a>
</p>

The diagram summarizes how configuration, datasets, samples, parameters, propagation, likelihood evaluation, and inference are connected.

[Read the Core Architecture page →]({{ '/docs/core-architecture.html' | relative_url }})

---

### Dial System

<p align="center">
  <a href="{{ '/assets/images/gundam-dial-system.png' | relative_url }}">
    <img
      src="{{ '/assets/images/gundam-dial-system.png' | relative_url }}"
      alt="GUNDAM Dial System workflow diagram"
      width="760"
    >
  </a>
</p>

The diagram shows how parameter changes are translated into dial responses, event-weight updates, and an updated Monte Carlo prediction.

[Read the Dial System page →]({{ '/docs/dial-system.html' | relative_url }})

---

### Statistical Inference

<p align="center">
  <a href="{{ '/assets/images/gundam-statistical-inference.png' | relative_url }}">
    <img
      src="{{ '/assets/images/gundam-statistical-inference.png' | relative_url }}"
      alt="GUNDAM Statistical Inference workflow diagram"
      width="760"
    >
  </a>
</p>

The diagram summarizes the repeated objective-evaluation workflow and the different roles of numerical minimization and MCMC sampling.

[Read the Statistical Inference page →]({{ '/docs/statistical-inference.html' | relative_url }})

---

### Applications

<p align="center">
  <a href="{{ '/assets/images/gundam-applications.png' | relative_url }}">
    <img
      src="{{ '/assets/images/gundam-applications.png' | relative_url }}"
      alt="GUNDAM Applications workflow diagram"
      width="760"
    >
  </a>
</p>

The diagram organizes the main applications, configuration tools, output components, inspection utilities, and optional interfaces.

[Read the Applications page →]({{ '/docs/applications.html' | relative_url }}) 

