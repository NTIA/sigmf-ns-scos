# SCOS Transfer Specification
Version 0.1

## Abstract
The SCOS Transfer Specification defines a standard for the controls and data format used within the [IEEE 802.22.3 Draft Standard: Spectrum Characterization and Occupancy Sensing (SCOS) system](http://www.ieee802.org/22/P802_22_3_PAR_Detail_Approved.pdf).

## Table of Contents

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->

- [SCOS Transfer Specification](#scos-transfer-specification)
    - [Abstract](#abstract)
    - [Table of Contents](#table-of-contents)
    - [1. Description](#1-description)
    - [2. Conventions Used in this Document](#2-conventions-used-in-this-document)
    - [3. Control Plane](#3-control-plane)
        - [3.1 ScheduleEntry Object](#31-scheduleentry-object)
        - [3.2 Action Object](#32-action-object)
    - [4. Data Plane](#4-data-plane)
        - [4.1 Global](#41-global)
            - [4.1.1 SensorDefinition Object](#411-sensordefinition-object)
                - [Antenna Object](#antenna-object)
                - [DataExtractionUnit Object](#dataextractionunit-object)
                - [SignalConditioningUnit Object](#signalconditioningunit-object)
                - [RFPath Object](#rfpath-object)
        - [4.2 Captures](#42-captures)
        - [4.3 Annotations](#43-annotations)
            - [4.3.1 Measurement Types](#431-measurement-types)
                - [SingleFrequencyFFTDetection Object](#singlefrequencyfftdetection-object)
                - [SteppedFrequencyFFTDetection Object](#steppedfrequencyfftdetection-object)
                - [SweptTunedDetection Object](#swepttuneddetection-object)
                - [YFactorCalibration Object](#yfactorcalibration-object)
            - [4.3.2 Dynamic Sensor Settings](#432-dynamic-sensor-settings)
                - [DynamicAntennaSettings Object](#dynamicantennasettings-object)
                - [DynamicDEUSettings Object](#dynamicdeusettings-object)
                - [DynamicSCUSettings Object](#dynamicscusettings-object)
            - [4.3.3 SystemToDetect Object](#433-systemtodetect-object)
    - [5. Index](#5-index)

<!-- markdown-toc end -->

## 1. Description
This transfer specification defines the controls and data format used within the Spectrum Characterization and Occupancy Sensing (SCOS) system. This system is an NTIA implementation of the [IEEE 802.22.3 draft standard](http://www.ieee802.org/22/P802_22_3_PAR_Detail_Approved.pdf), and draws upon the previous work of NIST and NTIA on the [MSOD System](https://github.com/usnistgov/SpectrumBrowser), while fully implementing the [SigMF Specification](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md) for the Data Plane. The goal is to standardize control method and data transfer format for “time share” access to a networked fleet of sensors. Use cases require a solid solution for basic sensor control and RF data collection with flexibility to incorporate new sensing solutions and metrics in the future.

## 2. Conventions Used in this Document

We have adopted SigMF's conventions:

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”,
“SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be
interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

JSON keywords are used as defined in [ECMA-404](http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf).

Augmented Backus-Naur form (ABNF) is used as defined by [RFC
5234](https://tools.ietf.org/html/rfc5234) and updated by [RFC
7405](https://tools.ietf.org/html/rfc7405).

Fields defined as "human-readable", a "string", or simply as "text" shall be
treated as plaintext where whitespace is significant, unless otherwise
specified.

## 3. Control Plane

The control plane for SCOS is implemented through the use of a RESTful API residing on the sensor, see [scos-sensor](https://github.com/NTIA/scos-sensor/blob/master/docs/api/openapi.adoc) (_* note this is currently a private repository and will be released at a future date_). A sensor advertises its **capabilities**, among which are **actions** that you can **schedule** the sensor to do. Some actions acquire data, and those **acquisitions** are retrievable in an easy to use archive format. Acquisitions are "owned" by the schedule entry. Schedule entries are "owned" by a specific user.

Actions are functions that the sensor owner implements and exposes. Actions can do anything, e.g., rotate an antenna, start streaming data over a websocket and never return. Sensor actions are scheduled via a **schedule entry** post, and the sensor executes a schedule entry to the best of its ability given all actions it is already responsible to perform.

The following objects are used within the `scos` SigMF name space in the Control and Data planes.

### 3.1 ScheduleEntry Object
The ScheduleEntry object requires the following name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`name`|true|string|N/A|The unique identification string assigned to the schedule entry.|
|`start`|false|integer|seconds|Absolute start time of the schedule in [Unix time](https://en.wikipedia.org/wiki/Unix_time).|
|`relative_stop`|false|integer|seconds|Stop time of the schedule relative to `start` time.|
|`stop`|false|integer|seconds|Absolute stop time of the schedule in [Unix time](https://en.wikipedia.org/wiki/Unix_time).|
|`interval`|false|integer|seconds|Interval time between instances of the `action` being performed.|
|`priority`|false|integer|N/A|Priority of the schedule, similar to applying [nice](https://en.wikipedia.org/wiki/Nice_(Unix)). Lower numbers are higher priority.|
|`action`|true|string|N/A|Name of action to be performed. See [Action Object](#32-action-object) definition.|

### 3.2 Action Object
The Action object requires the following name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`name`|true|string|N/A|The unique identification string assigned to the action.|
|`summary`|false|string|N/A|A succinct description of the action. Not required, but is RECOMMENDED.|
|`description`|false|string|N/A|A full description of the action. Not required, but is RECOMMENDED.|

## 4. Data Plane
The SCOS specification uses and is fully compliant with the SigMF Specification. Building upon the SigMF [core namespace](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#namespaces), the specification is enhanced through the implementation of a `scos` namespace, the details of which follow.  

### 4.1 Global
Per SigMF, the global object consists of name/value pairs that provide information applicable to the entire dataset. It contains the information that is minimally necessary to open and parse the dataset file, as well as general information about the recording itself.

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`SensorDefinition`|false|object|N/A|Describes the sensor model components. See [SensorDefinition Object](#411-sensordefinition-object) definition. This object is RECOMMENDED.|
|`sensor_id`|true|string|N/A|Unique name for the sensor.|
|`version`|true|string|N/A|The version of the SigMF SCOS namespace extension.|
|`ScheduleEntry`|false|object|N/A|See [ScheduleEntry Object](#31-scheduleentry-object) definition.|
|`task_id`|false|integer|N/A|A unique identifier that increments with each task of a `schdeule_entry`.|

#### 4.1.1 SensorDefinition Object
Sensor definition follows a simplified hardware model comprised of the following elements: Antenna, Signal Conditioning Unit (SCU), Data Extraction Unit (DEU), and Host Controller. Sensor implementations are not required to have each component, but metadata must specify the presence, model numbers, and operational parameters associated with each.

The following global objects are used within the `scos` SigMF name space to define the sensor.

The SensorDefinition object requires the following additional name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`Antenna`|true|object|N/A|See [Antenna Object](#antenna-object) definition.|
|`SignalConditioningUnit`|false|object|N/A|See [SignalConditioningUnit Object](#signalconditioningunit-object) definition.|
|`DataExtractionUnit`|true|object|N/A|See [DataExtractionUnit Object](#dataextractionunit-object) definition.|
|`host_controller`|false|string|N/A|Description of host computer. E.g. Make, model, and configuration.|

##### Antenna Object
The Antenna object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`model`|true|string|N/A|Antenna make and model number. E.g. `"ARA CSB-16"`, `"L-com HG3512UP-NF"`.|
|`type`|false|string|N/A|Antenna type. E.g. `"dipole"`, `"biconical"`, `"monopole"`, `"conical monopole"`.|
|`low_frequency`|false|float|Hz|Low frequency of operational range.|
|`high_frequency`|false|float|Hz|High frequency of operational range.|
|`gain`|false|float|dBi|Antenna gain in direction of maximum radiation or reception.|
|`horizontal_gain_pattern`|false|array|dBi|Antenna gain pattern in horizontal plane.|
|`vertical_gain_pattern`|false|array|dBi|Antenna gain pattern in vertical plane.|
|`horizontal_beam_width`|false|float|degrees|Horizontal 3-dB beamwidth.|
|`vertical_beam_width`|false|float|degrees|Vertical 3-dB beamwidth.|
|`cross_polar_discrimination`|false|float|N/A|Cross-polarization discrimination.|
|`voltage_standing_wave_ratio`|false|float|volts|Voltage standing wave ratio.|
|`cable_loss`|false|float|dB|Cable loss for cable connecting antenna and preselector.|
|`steerable`|false|boolean|N/A|Defines if the antenna is steerable or not.|
|`mobile`|false|boolean|N/A|Defines is the antenn is mobile or not.|

##### DataExtractionUnit Object
The DataExtractionUnit object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`model`|true|string|N/A|Make and model of DEU. E.g. `"Ettus N210"`, `"Ettus B200"`, `"Keysight N6841A"`, `"Tektronix B206B"`.|
|`low_frequency`|false|float|Hz|Low frequency of operational range of DEU.|
|`high_frequency`|false|float|Hz|High frequency of operational range of DEU.|
|`noise_figure`|false|float|dB|Noise figure of DEU.|
|`max_power`|false|float|dB|Maximum input power of DEU.|

##### SignalConditioningUnit Object
The SignalConditioningUnit object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`rf_path_specs`|false|array|N/A|Specication of SCU RF paths via [RFPath Object](#rfpath-object).|

##### RFPath Object
Each RFPath object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`rf_path_number`|false|integer|N/A|RF path number.|
|`low_frequency_passband`|false|float|Hz|Low frequency of filter 1-dB passband.|
|`high_frequency_passband`|false|float|Hz|High frequency of filter 1-dB passband.|
|`low_frequency_stopband`|false|float|Hz|Low frequency of filter 60-dB stopband.|
|`high_frequency_stopband`|false|float|Hz|High frequency of filter 60-dB stopband.|
|`lna_gain`|false|float|dB|Gain of low noise amplifier.|
|`lna_noise_figure`|false|float|dB|Noise figure of low noise amplifier.|
|`cal_source_type`|false|string|N/A|E.g., `"calibrated noise source"`.|
|`cal_source_ENR`|false|float|dB|Excess noise ratio of calibrated noise source at frequency of RF path.|

### 4.2 Captures
Per SigMF, the captures value is an array of capture segment objects that describe the parameters of the signal capture. The `scos` specification does not add any enhancements to this section.  

### 4.3 Annotations
Per SigMF, the annotations value is an array of annotation segment objects that describe anything regarding the signal data not part of the captures and global objects.

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`altitude`|false|float|meters|The height of the antenna above mean sea level.|
|`environment`|false|string|N/A|A description of the environment where antenna is mounted. E.g. `"indoor"` or `"outdoor"`.|
|`measurement_type`|true|object|N/A|The type of measurement acquired: [SingleFrequencyFFTDetection](#singlefrequencyfftdetection-object), [SteppedFrequencyFFTDetection](#steppedfrequencyfftdetection-object), [SweptTunedDetection](#swepttuneddetection-object) or [YFactorCalibration](#yfactorcalibration-object).|
|`SystemToDetect`|false|object|N/A|The system that the measurement is designed to detect. See [SystemToDetect Object](#433-systemtodetect-object) definition.|
|`data_sensitivity`|false|string|N/A|The sensitivity of the data captured. E.g. `"low"`, `"moderate"` or  `"high"`.|
|`DynamicAntennaSettings`|false|object|N/A|Dynamic parameters associated with the antenna. See [DynamicAntennaSettings Object](#dynamicantennasettings-object) definition.|
|`DynamicSCUSettings`|false|object|N/A|Dynamic parameters associated with the SCU. See [DynamicSCUSettings Object](#dynamicscusettings-object) definition.|
|`DynamicDEUSettings`|false|object|N/A|Dynamic parameters associated with the DEU. See [DynamicDEUSettings Object](#dynamicdeusettings-object) definition.|
|`detected_system_noise_powers`|false|float|dBm|The detected system noise power referenced to the output of isotropic antenna.|
|`temperature`|false|float|celsius|Environmental temperature.|
|`overload_flag`|false|boolean|N/A|Flag indicator of system signal overload.|

#### 4.3.1 Measurement Types
The following annotation objects are used within the `scos` SigMF name space associated with `measurement_type`. 

##### SingleFrequencyFFTDetection Object
Single-frequency FFT detection is a standard software-defined radio measurement. The SingleFrequencyFFTDetection object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`number_of_samples_in_fft`|true|integer|N/A|Number of samples in FFT to calcluate delta_f = [`samplerate`](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#global-object)/`number_of_samples_in_fft`.|
|`window`|true|string|N/A|E.g. `"blackman-harris"`, `"flattop"`, `"gaussian_a3.5"`, `"gauss top"`, `"hamming"`, `"hanning"`, `"rectangular"`.|
|`equivalent_noise_bandwidth`|false|float|Hz|Bandwidth of brickwall filter that has same integrated noise power as that of the actual filter.|
|`detector`|true|string|N/A|E.g. `"sample_iq"`, `"sample_power"`, `"mean_power"`, `"max_power"`, `"min_power"`, `"median_power"`.|
|`number_of_ffts`|true|integer|N/A|Number of FFTs to be integrated over by detector.|
|`units`|true|string|N/A|Data units, e.g., `"dBm"`, `"watts"`, `"volts"`.|
|`reference`|false|string|N/A|Data reference point, e.g., `"DEU input"`, `"antenna output"`, `"output of isotropic antenna"`.|

##### SteppedFrequencyFFTDetection Object
The SteppedFrequencyFFTDetection object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`center_frequency_start`|true|float|Hz|First center frequency of scan.|
|`center_frequency_stop`|true|float|Hz|Last center frequency of scan.|
|`center_frequency_step`|true|float|Hz|Center frequency step of scan.|
|`SingleFrequencyFFTDetection`|true|object|N/A|See [SingleFrequencyFFTDetection Object](#singlefrequencyfftdetection-object) definition.|

##### SweptTunedDetection Object
Swept-tuned detection is a standard spectrum analyzer measurement. The SweptTunedDetection object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`frequency_start`|true|float|Hz|First frequency of scan.|
|`frequency_stop`|true|float|Hz|Last frequency of scan.|
|`frequency_step`|true|float|Hz|Frequency step of scan.|
|`dwell_time`|true|float|seconds|Integration time of detector at each frequency step.|
|`resolution_bandwidth`|true|float|Hz|Resolution bandwidth.|
|`video_bandwidth`|true|float|Hz|Video bandwidth.|
|`units`|true|string|N/A|Data units, e.g., `"dBm"`, `"watts"`, `"volts"`.|
|`reference`|false|string|N/A|Data reference point, e.g., `"DEU input"`, `"antenna output"`, `"output of isotropic antenna"`.|

##### YFactorCalibration Object
The YFactorCalibration object requires the following:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`last_time_performed`|true|datetime|[ISO-8601](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#the-datetime-pair)|Date and time of last calibration.|
|`calibration_dictionary`|false|array|dB|A list of DEU attenuations with cooresponding calibration results. Calibration results are gain and noise figure arrays equal in length to the [`sample_count`](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#annotation-segment-objects).|   
|`reference`|false|string|N/A|Data reference point, e.g., `"DEU input"`, `"antenna output"`, `"output of isotropic antenna"`.|

An example `calibration_dictionary`, where "1" and "2" are DEU attenuation values:
```
{ 1: { "gain" : [],
       "noise_figure" : []
     },
  2: { "gain" : [],
       "noise_figure" : []
     },
  ...
}
```

#### 4.3.2 Dynamic Sensor Settings
The following annotation objects are used within the `scos` SigMF name space associated with dynamic settings in the antenna, SCU, and DEU.

##### DynamicAntennaSettings Object
The DynamicAntennaSettings object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`azimuth_angle`|false|float|degrees|Angle of main beam in azimuthal plane from North.|
|`elevation_angle`|false|float|degrees|Angle of main beam in elevation plane from horizontal.|
|`polarization`|false|float|string|E.g. `"vertical"`, `"horizontal"`, `"slant-45"`, `"left-hand circular"`, `"right-hand circular"`.|

##### DynamicDEUSettings Object
The DynamicDEUSettings object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`attenuation`|false|float|dB|Attenuation of DEU.|

##### DynamicSCUSettings Object
The DynamicSCUSettings object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`rf_path_number`|false|integer|N/A|SCU RF path number.|

#### 4.3.3 SystemToDetect Object
The SystemToDetect object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`system_name`|false|string|N/A|Name of system to be detected.|
|`transmit_power`|false|float|dBm|Transmitter power going into antenna.|
|`antenna_gain`|false|float|dBi|Antenna gain.|
|`signal_type`|false|string|N/A|Type of signal, e.g., `"pulsed"`
|`latitude`|false|float|degrees|Latitude.|
|`longitude`|false|float|degrees|Longitude.|
|`altitude`|false|float|meters|Altitude above mean sea level.|

## 5. Index
[Action Object](#32-action-object)  
[Annotations](#43-annotations)  
[Antenna Object](#antenna-object)  
[Captures](#42-captures)  
[Control Plane](#3-control-plane)  
[DataExtractionUnit Object](#dataextractionunit-object)  
[Data Plane](#4-data-plane)  
[DEU](#dataextractionunit-object)  
[DynamicAntennaSettings Object](#dynamicantennasettings-object)  
[DynamicDEUSettings Object](#dynamicdeusettings-object)  
[DynamicSCUSettings Object](#dynamicscusettings-object)  
[Dynamic Sensor Settings](#432-dynamic-sensor-settings)  
[Global](#41-global)  
[Measurement Types](#431-measurement-types)  
[RFPath Object](#rfpath-object)  
[ScheduleEntry Object](#31-scheduleentry-object)  
[SCU](#signalconditioningunit-object)  
[SensorDefinition Object](#411-sensordefinition-object)  
[SignalConditioningUnit Object](#signalconditioningunit-object)  
[SingleFrequencyFFTDetection Object](#singlefrequencyfftdetection-object)  
[SteppedFrequencyFFTDetection Object](#steppedfrequencyfftdetection-object)  
[SweptTunedDetection Object](#swepttuneddetection-object)  
[SystemToDetect Object](#433-systemtodetect-object)  
[YFactorCalibration Object](#yfactorcalibration-object)  
