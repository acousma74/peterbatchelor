---
layout: default
title: "A New Project"
---
# Designing a Distributed Playback Engine in Pure Data

In previous posts, I described the overall architecture and deployment of the Chamberlain Clock Tower installation: a distributed network of Raspberry Pis, each responsible for local playback within a campus-wide multichannel sound work.

This post focuses on the software layer that made this possible: a custom-built Pure Data patch, deployed identically across all devices, and designed to handle timing, control, and multichannel playback in a flexible and resilient way.

----------

## A Single Patch, Many Roles

For efficiency, I aimed to maintain a  single, unified patch  across all Raspberry Pis, regardless of their role in the system, or whether they handled one, two, four or eight channels. Any differences in behaviour were determined at runtime—primarily through device identification and configuration—rather than through maintaining separate versions.

This significantly simplified deployment and maintenance. Updates could be made once and propagated across all pis.

----------

## Patch Overview

At a high level, the patch can be understood as a signal and control pipeline:

**Time → TimeTranslation → File Selection → Playback → Gain Staging**

Each of these stages was implemented as a dedicated subpatch, many of which were exposed using  _graph-on-parent_  to allow monitoring from the main interface.

----------

## Time Handling (`[pd TIME]`)

The system begins with a dedicated time subpatch, responsible for generating and manipulating time data.

Using a [time] external, the patch retrieves current time in (hh mm ss).
    

These are then processed through a custom  **time-shifting mechanism**, which:

1.  Converts the current time into total seconds since midnight
    
2.  Applies an offset (to account for clock drift)
    
3.  Converts the result back into hours, minutes, and seconds
    

This offset value was sourced from the remote control system (via JSON), allowing the entire network to be calibrated dynamically against the clock tower.

A secondary component formats the time into a string (`HH:MM:SS`), which is used both for display and as a key for lookup operations later in the patch.

----------

## Time Translation (Handling Clock Drift)

One of the more specific components of the system is the  **time translation layer**.

As discussed in the previous post, the clock tower chimes did not occur at consistent absolute times. To address this, a mapping system was implemented using  `coll`  objects.

This mapping:

-   Takes an  _observed_  (adjusted) time
    
-   Outputs the  _intended_  canonical time (e.g. “10:15:00”)
    

In effect, this acts as a lookup table that compensates for the irregular behaviour of the clock. It allows the system to remain aligned with the chimes, even as their timing drifts.

----------

## File Selection (`pd file caller`)

Once a valid trigger time is identified, the patch determines which audio file should be played.

This is handled by a dedicated subpatch that:

-   Selects the appropriate  `coll`  file based on the device identity
    
-   Uses the formatted time as an index
    
-   Outputs:
    
    -   The audio file name
        
    -   Optional parameters (e.g. gain)
        

Each Raspberry Pi was assigned an ID at initialisation. This ID determined which set of audio files it should access—allowing the same patch to serve different roles across the installation.

Audio files followed a structured naming convention, enabling the patch to dynamically construct the correct filename for each channel.

----------

## Playback Engine (`pd players`)

At the core of the system is the playback engine.

This was implemented using  `readsf~`, configured as an  **8-channel playback system**, regardless of the actual number of outputs used by a given device.

While this is technically inefficient for devices only outputting one or two channels, it allowed for a  **uniform patch design**  across all nodes.

From a practical perspective, this consistency was more valuable than optimising for minimal resource usage.

The playback subpatch also exposed key monitoring information:

-   Active file name
    
-   Channel assignment
    
-   Output levels
    

----------

## Gain Structure

A three-stage gain system was implemented to provide flexible control over levels:

### 1. Composition-Level Gain

Defined within the  `coll`  entries themselves:

-   Allows normalisation between different compositions
    
-   Ensures consistent perceived loudness across works
    

----------

### 2. Per-Device Gain

A dedicated subpatch allows adjustment of individual Raspberry Pis:

-   Useful for compensating for speaker placement or local conditions
    
-   Could be adjusted remotely
    

----------

### 3. Global Gain

A final master control applied to all outputs:

-   Provides overall level control for the system
    

----------

### Metering

Basic metering (via number boxes) provided visibility of signal levels across channels.

----------

## Clock and Playback Control

A gating mechanism allows the entire system to be enabled or disabled. This was useful during setup and testing, and also allowed selective control of different parts of the installation.

Additionally, a fail-safe subpatch ensured that playback only occurred within a defined time window (e.g. 07:00–21:00). Outside of these hours, audio output was automatically disabled to prevent unintended activity.

----------

## Artificial Chime System

A separate signal path handled  **synthetic clock chimes**, which could be enabled if the physical clock failed.

This system:

-   Played a mono recording of the chimes
    
-   Routed it across relevant outputs
    
-   Could be toggled remotely
    

In practice, this proved valuable when the clock tower temporarily stopped during the installation period. It also introduced an interesting inversion: when using artificial chimes, the system effectively became self-synchronising.

----------

## Live Triggering

In addition to scheduled playback, the patch supported  **live triggering**.

Commands sent via the remote control system specified:

-   A file to play
    
-   A future trigger time (typically ~15 seconds ahead)
    

Each Raspberry Pi would:

-   Receive the instruction
    
-   Schedule playback locally
    

This enabled:

-   System testing (e.g. sending a “bong” to a specific node)
    
-   Ad hoc playback during setup
    
-   Flexible intervention during the installation
    

----------

## Remote Data Integration (`pd JSON puller`)

The patch included a subpatch responsible for retrieving control data from a remote JSON source.

This was implemented using a command-line call (via a  `command`  object), which proved more reliable than earlier approaches using shell-based objects.

Each Raspberry Pi polled the remote data at regular intervals (approximately every seven seconds), with slight random variation introduced to avoid simultaneous requests across all devices.

The retrieved data included:

-   Timing offsets
    
-   Playback instructions
    
-   Control flags
    

----------

## Initialisation

A dedicated initialisation subpatch handled:

-   Library loading (e.g. Cyclone, Zexy, Mapping)
    
-   Audio configuration (sample rate, etc.)
    
-   Device identification
    

The Raspberry Pi’s identity (used throughout the patch) was derived from its network configuration, allowing each node to determine its role automatically at startup.

----------

## Reflections

The Pure Data system was designed with consistency, flexibility, and resilience in mind. Rather than building highly specialised patches for each device, the emphasis was on creating a single adaptable framework that could accommodate a range of roles.

In some cases, this meant accepting inefficiencies—such as running an 8-channel playback engine on devices using fewer outputs. In practice, however, the benefits of uniformity outweighed these costs.

Perhaps the most important aspect of the design is that it bridges multiple layers of the system:

-   Temporal control
    
-   Networked coordination
    
-   Audio playback
    

All within a single, coherent patch.

----------

## Closing Thoughts

At a glance, the patch is a collection of subpatches and control structures. In operation, however, it functions as a distributed instrument—one that translates time, space, and networked data into coordinated sonic events.

If the earlier stages of the project were about designing the system, this patch is where that system becomes concrete.

----------

_(In a future post, I may break down individual subpatches in more detail, particularly the time translation and remote control mechanisms.)_