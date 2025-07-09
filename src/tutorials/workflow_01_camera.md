# Workflow 01: Single Camera Acquisition

Open the first tutorial workflow, [1-camera.bonsai](../workflows/1-camera.bonsai) 

## Overview

This workflow demonstrates a minimal configuration for video streaming in [Project Aeon](https://github.com/SainsburyWellcomeCentre/aeon_docs). It sets up a [TimestampGenerator](https://github.com/harp-tech/device.timestampgeneratorgen3) device to provide a common clock for any and all Harp hardware added to the rig. A single [FLIR Spinnaker Blackfly S camera](https://www.flir.com/products/blackfly-s-usb3/?model=BFS-U3-16S2M-CS) will also be set up and controlled using a global trigger from a [Camera Controller](https://github.com/harp-tech/device.cameracontroller) device to acquire synchronized video frames along with experimental metadata. It introduces fundamental Aeon concepts including:

- Modular [Bonsai](https://bonsai-rx.org/) workflow design and structure
- [Harp](https://harp-tech.org/) hardware control and synchronization

This workflow is a single-camera streaming pipeline that forms the foundation for more complex setups.

---

## Workflow Structure

This workflow consists of two main groups:

- **Devices**: Sycnhronizer, video controller and camera interface
- **Control Panel**: Manual start/stop of recording and camera visualizer stream selection

![1-camera](../../docs/workflowImages/1.1-camera.svg)

Many of these workflows are constructed with the use of reuseable `IncludeWorkflows` from the [`Aeon.Acquisition`](https://github.com/SainsburyWellcomeCentre/aeon_acquisition) package. 

---

## Node Groups

### Group: `Devices`

**Purpose**: The Devices group configures and initialises connections with the experimental hardware. To start with here, we instantiate the several [Harp](https://harp-tech.org/) devices: [Timestamp Generator](https://github.com/harp-tech/device.timestampgeneratorgen3), a [Camera Controller](https://github.com/harp-tech/device.cameracontroller), and a camera.

![1-camera-devices](../../docs/workflowImages/1.2-camera-devices.svg)

#### SubGroup:`TimestampGenerator`

Configures a Harp [TimestampGenerator](https://github.com/harp-tech/device.timestampgeneratorgen3) which provides a common clock signal to sycnhronize the data streams and other hardware. 

#> **Note**
> This function can also be achieved with a [Clock Synchronizer](https://github.com/harp-tech/device.clocksynchronizer) device, for which a dedicated `IncludeWorkflow` `Aeon.Acquisition:ClockSynchronizer.bonsai` is also available and can be exchanged for the `TimestampGenerator` group. 

- **Included Workflow**: `Aeon.Acquisition:TimestampGenerator.bonsai`

| Property    | Description |
|-------------|-------------|
| PortName    | COM port    |

#### `VideoController`

Configures and controls a Harp [Camera Controller](https://github.com/harp-tech/device.cameracontroller) device. This device uses two PWM outputs as sources to trigger frame captures at frequencies set in the properties of this group.

- **Included Workflow**: `Aeon.Acquisition:CameraController.bonsai`

| Property               | Description                 |
|------------------------|-----------------------------|
| PortName               | COM port                    |
| GlobalTriggerFrequency | Frequency of global trigger |
| LocalTriggerFrequency  | Frequency of local trigger  |

#### `CameraTop`
- **Included Workflow**: `Aeon.Acquisition:SpinnakerVideoSource.bonsai`

| Property           | Description                                           |
|--------------------|-------------------------------------------------------|
| TriggerSource      | Source signal used to initiate frame capture          |
| TriggerFrequency   | Frequency at which frames are captured                |
| ExposureTime       | Duration of each camera exposure in microseconds      |
| SerialNumber       | Unique identifier of the specific camera device       |
| Gain               | Image gain applied to sensor signal                   |
| Binning            | Sets spatial binning applied to camera image          |
| FrameEvents        | Name of a shared `Subject` to cast captured frames    |
| ExposureTime       | Duration (in microseconds) of each camera exposure    |
| SerialNumber       | Unique ID of the physical camera device               |

---

### Group: `Control Panel`
 This group provides an interactive user interface for the workflow and connected components. Here keyboard inputs are used to trigger recording by initiating or pausing camera triggers, and provides a convenient visualizer to view the camera stream live. In experiments with multiple cameras these will appear in the dropdown of the generated visualizer.

![1-camera-control-panel](../../docs/workflowImages/1.4-control-panel.svg)
#### Key Bindings

- `StartCameras`: Triggered by keys: `Shift + F1`
- `StopCameras`: Triggered by keys: `Shift + F3`

#### Subgroup: `CameraSelector`


- **Stream Selector** using: `p1:StreamSelector`

| Property         | Description                                           |
|------------------|-------------------------------------------------------|
| SelectedStream   | The name of the camera stream displayed by default   |

---

## Next Steps

➡ Continue to [02: Logging Camera Acquisition and Devices](../tutorials/workflow_02_logging.md) to add standardized logging to your workflow.