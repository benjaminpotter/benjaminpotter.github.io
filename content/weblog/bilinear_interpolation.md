+++
title = "Bilinear Interpolation for DoFP Polarized Camera"
date = 2026-02-18
draft = true
+++

This is what I think makes sense to do.
First, we split out the `IntensityImage` reading from the base type of the intensity image.
This means that the interpreting of the intensity pixels from the raw image data is its own module.
This `DofpReader` type can construct a `RayImage` from these types of images.

Should the `DofpReader` really construct an `IntensityImage` or just a `RayImage` directly?

