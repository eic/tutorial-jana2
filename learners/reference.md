---
title: Reference
---

## Glossary

EICrecon
: The ePIC reconstruction software, built on the JANA2 framework.

JANA2
: The multithreaded event-processing framework used by EICrecon.

JEventSource
: A JANA2 component that reads data model objects from an input file.

JEventProcessor
: A JANA2 component that aggregates results from each event into a structured output such as a
  histogram or a file.

JFactory
: A JANA2 component that computes new data model objects from existing ones, on demand and in
  parallel.

plugin
: A compiled, loadable unit that bundles JANA2 components (processors, factories) to be run by
  `eicrecon`.

podio
: The event data model toolkit used to read and write EIC data files.
