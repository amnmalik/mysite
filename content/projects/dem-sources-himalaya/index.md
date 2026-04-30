---
title: "Mapping the Himalaya: Which DEM to Use?"
date: 2026-04-30
summary: "A look at the different elevation datasets available for the Himalaya - Cartosat, SRTM, ALOS, and the high-resolution HMA DEMs - and what each is actually good for."
tags: ["himalaya", "GIS", "mapping", "DEM", "elevation", "himachal"]
---

DEMs are the basis for a wide range of applications - from hydrological modelling, to landslide risk analysis. For a hiking enthusiast like me, DEMs are used to generate contour lines, which then go on maps (like osmand) to understand the topography over which you are travelling (like identifying ridges, valleys, passes, and peaks), or to understand the steepness of the terrain. In fact, many hiking routes also often go aong contour lines of same elevation. 

Over the last few years for various applications, I've ended up with quite a spread of elevation datasets sitting on my hard drive. This blog post is about the different datasets that I could find and some basic information about them that I gathered from the websites. 

## [The Familiar Baseline: SRTM (~30m and 90m)](https://www.earthdata.nasa.gov/data/catalog/lpcloud-srtmgl1-003)

The Shuttle Radar Topography Mission is probably the most widely used global DEM there is. It was collected in February 2000 over eleven days using radar interferometry from the Space Shuttle Endeavour, and covers most of the land surface between 60N and 56S. The version most people use is SRTM GL1, which has a resolution of one arc-second - roughly 30 metres at these latitudes.

For the Himalaya, SRTM has a known limitation: radar struggles with very steep terrain and with snow-covered surfaces. The radar signal can bounce off the snowpack rather than the actual ground, meaning the elevation values in heavily glaciated areas tend to be a bit off. The voids - areas where the radar couldn't get a clean reading - are more common than in gentler landscapes. Various groups have worked on void-filled versions, and these are generally fine for regional analysis, but at the scale of a single valley or a ridge, you can see the roughness.

## [Cartosat-1 from NRSC (~30m)](https://bhuvan-app3.nrsc.gov.in/data/)

India's own DEM offering comes from the Cartosat-1 satellite, which ISRO launched in 2005. It carries a stereo pair of panchromatic cameras, which NRSC used to generate DEMs through optical stereo photogrammetry rather than radar. The data is distributed through the Bhuvan open data archive, and the version I have is the third release: 16-bit, one arc-second resolution, collected over the 2005-2014 period.

Because it uses optical imagery rather than radar, Cartosat behaves differently over snow. On clear days, the cameras can see the actual snow surface, and stereo matching over uniform white terrain is hard - you end up with noise in glaciated areas. But over rocky terrain, moraine, and forested slopes, the quality is often noticeably better than SRTM. 


## [ALOS World 3D from JAXA (~30m)](https://www.eorc.jaxa.jp/ALOS/en/dataset/aw3d30/aw3d30_e.htm)

Japan's Advanced Land Observing Satellite carried the Prism instrument, a three-lens stereo camera with a 2.5 metre native resolution. JAXA used this to generate the AW3D30 product - a 30 metre DEM that averages multiple acquisitions for better accuracy. Each tile also comes with a stacking number file that tells you how many individual scenes were averaged at each point, which is a useful quality indicator.

The multi-look averaging smooths out some of the noise that you get from single-pass sensors, and the optical stereo approach works reasonably well over rocky Himalayan terrain. Where it struggles, predictably, is over glaciers and permanent snowfields.

## [HMA DEM: The One That Changes Everything (~8m)](https://nsidc.org/data/hma_dem8m_at/versions/1)

The High Mountain Asia DEM dataset is in a different category from everything above. It was produced by the University of Washington's Polar Science Center using commercial stereo imagery - WorldView-1, WorldView-2, and GeoEye-1 - at resolutions of half a metre to two metres, then processed to generate DEMs at eight metre resolution. The data is hosted by NSIDC and covers much of the Hindu Kush-Karakoram-Himalaya-Tibetan Plateau region.

Eight metres changes what you can do with a DEM in mountain terrain. At 30 metres, a narrow river gorge might be represented by just one or two pixels across. At eight metres, you start seeing the actual shape of the valley floor, the terracing on the slopes, the moraine ridges. 

The limitation is coverage.  The strips follow the orbital paths and you can have large gaps between them. For broad regional analysis you still need one of the 30 metre products as a base. But wherever an HMA strip covers your area of interest, it is the one to use.