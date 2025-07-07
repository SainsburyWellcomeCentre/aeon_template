
# Workflow 03: Computer Vision Tracking of Experimental Subjects 

Open the third tutorial workflow, [3-tracking.bonsai](../workflows/3-tracking.bonsai) 

In this step, we extend the basic acquisition workflow by adding real-time blob tracking using computer vision. This allows us to monitor and log the experimental subject's position frame by frame. Additionally, we will visualize this movement as a possible stream in the `StreamSelector` as a heatmap.

---
## Group: `Metadata`

### New Subgroup: `TrackingTop`

![3-tracking](../../docs/workflowImages/3.1-tracking.svg)

- **Included Workflow**: `Aeon.Acquisition:PositionTracking.bonsai`

This workflow processes each frame from the "CameraTop" `Subject`. First it applies a mask to the captured image. This mask is an 8-bit binary image of the same dimensions as the captured camera frames, usually generated manually. Pixels to be included in the tracking are set to a value of 255, while those excluded are 0. An example mask image can be found [here](../workflows/Config/ArenaMask-template.png). This is useful to exclude extra parts of the video frame that are not part of the experimental arena, avoiding processing uninformative and noisy pixels. Operators which implement OpenCV functions extracts the centroid of the largest blob within the mask(typically an animal). The coordinates are timestamped and published to the "TrackingTop" `Subject` .

| Property         | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| ThresholdValue   | Threshold applied to the image to segment the foreground object             |
| Mask             | Path to an image mask that defines valid tracking regions (e.g. arena only) |
| TrackingCount    | Max number of objects to track (typically 1 for single-animal tracking)     |
| TrackingEvents   | Name of the published stream of tracking events                             |

## Group: `Visualization`

Within this new group we have added a sequence to take the centroid positions from the `TrackingTop` stream and accumulate them into a 2D histogram that can be visualized as a heatmap.

![3-visualizer-heatmap](../../docs/workflowImages/3.2-visualizer.svg)

It consists of:
- `Histogram2D`: Builds up an occupancy map. Here the parameters for the visualisation image can be set, for example the binning and dimensions of the resulting histogram
- `ConvertScale`: Adjusts brightness for visualization
- `SampleInterval`: Controls sampling rate from the incoming stream.
- `PublishSubject`: Publishes the final heatmap as a shared `Subject` called "HeatMap"

The heatmap is typically used to monitor continually running experiments, and as such is not logged. If desired, the heatmap can be timestamped and logged using a `logVideo` node, or it can be recreated post-hoc from the logged tracking data.
---

## Group: `ControlPanel`

We have added the "Heatmap" `Subject` to the `CameraSelector`, allowing the option to preview the `HeatMap` stream that is generated in the new visualization group.

![3-controlpanel-heatmap](../../docs/workflowImages/3.3-controlpanel.svg.svg)

---

## Group: `Logging`

New logging functionality is added to record the new `TrackingTop` data stream:

![3-tracking-logging](../../docs/workflowImages/3.4-logging.svg)

- The stream is passed through a series of formatting nodes to prepare the data for logging under the same standard as previously described.  
1. The `FormatBinaryRegions` operator converts the tracking data into messages conforming to the harp protocol. `Format` from the `Bonsai.Harp` package further prepares these messages for logging, importantly assigning the register address for this data stream, impacting the file naming for the stored binaries.
- The resulting events are logged using the `LogHarpState` node

---
## Next Steps

➡ Continue to [04: ](../tutorials/workflow_04_XXXX.md)
