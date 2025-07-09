# Workflow 02: Logging Camera Acquisition and Devices

Open the second tutorial workflow, [2-logging.bonsai](../workflows/2-logging.bonsai) 

## Overview
This workflow adds a "Logging" group to the workflow, and places the `Devices` group inside a `Metadata` group. This group will be searched by name by a custom python script called from Bonsai, which logs the properties of all encapsulated operators with the data, as a snapshot of the experimental parameters. 

![2-logging](../../docs/workflowImages/2.1-logging.svg)

This group handles raw data and relevant metadata and writes them to disk in a structured format. The Aeon platform is particularly well-suited and originally designed for expansive, open field behavioural studies that may not immediately be hypothesis driven. The goal of logging, therefore, is to be efficient and holistic enough that nothing is lost and every session can be traced and re-analyzed, allowing for mining of the data for evidence to different questions, while maintaining reproducibility.

An Aeon experiment may go on continuously for several days, neccessitating the writing of data into manageable sizes, or "chunks" of 1 hr. Additionally, The repository branch name and git commit hash or tag are integrated into the data path for full reproducibility, ensuring the executed code is always recoverable. 

### Logging Criteria

1. Maintain a standardised directory structure, ensuring subject, timestamps and experiment type are faithfully logged
1. Log all hardware events and states during an experiment on a common harp clock
2. Chunk data into manageable sizes for downstream data handling
3. Log metadata and the executed code with the raw data to maintain reproducability while utilising a developing code-base.

#### Standardized directory structure

Each session folder is timestamped and contains subfolders for each device with their logs clearly separated. The git repository tag name and/or commit hash forms part of the root folder name, ensuring the original code run can always be recovered. This structure is handled automatically by the Aeon `RepositoryLogController` and `IncludeWorkflows`. The events from all devices are logged in dedicated folders as labelled binary files, one for each register of the device. 

For example:

```text
ProjectAeon/
└── social-ephys_ae64a4047b7ec76b875f431e406677b585128074/
    └── 2024-04-08T12-15-57
        ├── metadata.yml
        ├── CameraTop
        │   └── CameraTop_2024-04-08T12-00-00.avi
        │   └── CameraTop_2024-04-08T12-00-00.csv
        ├── ClockSynchronizer/
        │   └── ClockSynchronizer_8_2024-04-08T12-00-00.bin
        ├── VideoController/
        │   └── VideoController_8_2024-04-08T12-00-00.bin
        │   └── VideoController_66_2024-04-08T12-00-00.bin
        │   └── VideoController_68_2024-04-08T12-00-00.bin
``` 

## Workflow Structure
Inside the "Logging" group, the criteria are achieved using reuseable `IncludeWorkflows` in `Aeon.Acquisition`: in this workflow, `LogHarpState` and `LogVideo` nodes, controlled by a `RepositoryLogController`.

![2-logging-group](../../docs/workflowImages/2.2-logging-group.svg)

### Group: RepositoryLogController
- **Included Workflow**: `Aeon.Acquisition:RepositoryLogController.bonsai`

This workflow generates a formatted path from the repository and passes it to a shared `Subject` named "PathPrefix", that is utilized by all Aeon logging modules to provide a root path to log data to. 

| Property        | Description                                                |
|-----------------|------------------------------------------------------------|
| RepositoryPath  | Root path for session data                                 |
| DataPath        | Directory where logged data will be stored                 |
| FallbackPath    | Backup path if main path fails                             |
| ChunkSize       | Length of time, in hours, to chunk data                    |

#### Subgroup: `ExportMetadata`
- **Included Workflow**: `Aeon.Acquisition:ExportMetadata.bonsai`

**Purpose**: This node calls a python script within your virtual environment, [`metadata.py`](../../calibration/metadata.py), which exports metadata about the workflow being run to a `metadata.yml` file, stored with the raw data. We have moved the `Devices` group into the `Metadata` group in the workflow because this script searches through the properties of operators inside the `Metadata` group in the workflow. 

This export is only triggered once, at the beginning of each session and includes internal checks to ensure it completes successfully, raising an exception message if it fails. An exception may also be thrown if the local repository is not 'clean', meaning there are changes that may make the commit hash tracking of executed code inaccurate. This functionality can be switched on or off inside the `metadata.py` script itself.

| Property       | Description                                            |
|----------------|--------------------------------------------------------|
| WorkflowName   | The name of the current Bonsai workflow being executed |

Output file: `Metadata.yml`, saved to the current session’s data folder, controlled by the [`RepositoryLogController`](#repositorylogcontroller)

### LogHarpState
- **Used to log**: Harp devices such as `ClockSynchronizerEvents` and `VideoControllerEvents`. `LogName` is appended to the `PathPrefix` emitted by the `RepositoryLogController` to generate the full save path.  

| Property         | Description                                               |
|------------------|-----------------------------------------------------------|
| LogName          | Filename for this Harp log                                |
| Heartbeats       | Heartbeat Source `Subject` name                           |
| ClosingDuration  | Allowance for file closure                                |

### LogVideo
- **Used to log**: A video of type `Aeon.Acquistion.VideoDataFrame` and timestamped on the same harp time as all other devices. For each camera, here `CameraTop`, a raw .avi movie file is logged, along with a .csv file that contains the chunk data, and hardware frameID and timestamps, especially useful when developing or debugging cameras. 

| Property         | Description                                               |
|------------------|-----------------------------------------------------------|
| FrameRate        | Rate at which video frames are expected to be recorded    |
| LogName          | Filename for this Harp log                      |
| Heartbeats       | Heartbeat Source `Subject` name                           |
| ClosingDuration  | Allowance for file closure                                |

---

## Next Steps

➡ Continue to [#03: Computer Vision Tracking of Experimental Subjects](../tutorials/workflow_03_tracking.md) to add tracking of an experimental subject to this workflow.