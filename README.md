# SCOS Transfer Specification
Version 0.2

## Abstract

## 1. Description
This transfer specification defines the controls and data format used within the Spectrum Characterization and Occupancy Sensing (SCOS) system. This system is an NTIA implementation of the [IEEE 802.22.3 draft standard](http://www.ieee802.org/22/P802_22_3_PAR_Detail_Approved.pdf), and draws upon the previous work of NIST and NTIA on the [MSOD System](https://github.com/usnistgov/SpectrumBrowser), while fully implementing the [SigMF Specification](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md) for the Data Plane.  

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

The control plane for SCOS is implemented through the use of a RESTful API residing on the sensor. The following API calls can be made to the sensor.

**AUTO GENERATED API DOCUMENTATATION SHOULD GO HERE.**

https://github.com/NTIA/scos-sensor/blob/master/docs/api/openapi.adoc

## 4. Data Plane
The SCOS specification uses and is fully compliant with the SigMF Specification. Building upon the SigMF [core namespace](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#namespaces), the specification is enhanced through the implementation of a `scos` namespace, the details of which follow.  

### 4.1 Global

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`sensor_definition`|false|object|N/A|Describes the sensor model components. See [Sensor Definition](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#510-sensor-definition) object definition. This object is RECOMMENDED.|
|`sensor_id`|true|string|N/A|Unique name for the sensor.|
|`version`|true|string|N/A|The version of the SCOS SigMF name space.|
|`schedule_entry`|false|object|N/A|See [Schedule Entry](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#59-schedule-entry) object definition.|
|`task_id`|false|integer|N/A|A unique identifier that increments with each task of a `schdeule_entry`.|

### 4.2 Captures
The `scos` specification does not add any enhancements to this section.  

### 4.3 Annotations

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`altitude`|false|float|meters|The height of the antenna above sea level.|
|`environment`||string||A description of the environment where antenna is mounted: `indoor` or `outdoor`.|
|`antenna`||string||Settings of the antenna attached to the sensor: `azimuth_beam_dir`, `elevation_beam_dir` or `polarization`.|
|`system_to_detect`||string||The system that the measurement is designed to detect: `radar–SPN43`, `lte` or `none`.
|`data_sensitivity`||string||The sensitivity of the data captured: `Low`, `Medium` or  `High`
|`measurement_type`||object||The type of measurement acquired: [`fixed_frequency_fft`](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#55-fixed-frequency-fft), [`stepped_frequency_fft`](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#513-stepped-frequency-fft), [`spectrum_analyzer`](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#512-spectrum-analyzer), [`calibration`](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#53-calibration) or [`power_delay_profile`](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#57-power-delay-profile)|
|`window`|||Hz||
|`number_of_ffts`||integer|||
|`detector`||string||The mode of operation for detecting signals: `min`, `max`, `mean`, or `median`|
|`equivalent_noise_bandwidth`||float|Hz|The equivalent noise bandwidth.|
|`attenuation`|||dB||
|`frequency_overlap`||integer|Hz||
|`temperature`||float|celsius||
|`overload_flag`||boolean||Flag indicator of system signal overload.|
|`detected_system_noise_powers`||float|dBm|The detected system noise power referenced to the output of isotropic antenna.|
|`processed`||boolean||If `true` the data is measured powers, as opposed to `false` indicating raw measured powers|

## 5. Object Definitions

The following objects are used within the `scos` SigMF name space. They are listed in alphabetical order.  

### 5.1 Action

The `action` object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`name`||string|N/A||
|`summary`||string|N/A||
|`description`||string|N/A||

### 5.2 Antenna

The `antenna` object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`model`||string|N/A||
|`low_frequency`||float|Hz||
|`high_frequency`||float|Hz||
|`gain`||float|dB||
|`horizontal_beam_width`||float|degrees||
|`vertical_beam_width`||float|degrees||
|`cross_polar_discrimination`||float|||
|`voltage_standing_wave_radio`||float|volts||
|`cable_loss`||float|dB||
|`steerable`||boolean|N/A||
|`mobile`||boolean|N/A||

### 5.3 Calibrations

The `calibrations` array holds calibration objects:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`type`|false|string||`y-factor cal`, etc. |
|`last_time_performed`|false|datetime|[ISO-8601](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#the-datetime-pair)|Date and time of last calibration.|
|`calibration_dictionary`|false|array|dB|A list of attenuations with cooresponding calibration results. Calibration results are gain and noise figure arrays equal in length to the [`sample_count`](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#annotation-segment-objects).|   

An example `calibration_dictionary`, where "1" and "2" are attenuation values:
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

### 5.4 Data Extraction Unit

The `data_extraction_unit` object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`make`||string|||
|`model`||string|||
|`low_frequency`||float|Hz||
|`high_frequency`||float|Hz||
|`noise_figure`||float|dB||
|`max_power`||float|dB||

### 5.5 Fixed Frequency FFT

The `fixed_frequency_fft` object requires the following additional name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|||||

### 5.6 Host Controller

The `host_controller` object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`software_version`||string|N/A||
|`cpu`||string|N/A||
|`memory`||integer|GB||
|`storage_size`||integer|GB||

### 5.7 Power Delay Profile

The `power_delay_profile` object requires the following additional name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|||||

### 5.8 RF Path #

The `rf_path_#` object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`low_frequency_passband`||float|Hz||
|`high_frequency_passband`||float|Hz||
|`low_frequency_stopband`||float|Hz||
|`high_frequency_stopband`||float|Hz||
|`lna_gain`||float|Hz||

### 5.9 Schedule Entry

The `schedule_entry` object requires the following additional name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`id`|true|integer|N/A|The unique identification number assigned to the schedule entry.|
|`start`|false|integer|seconds|Absolute start time of the schedule in [Unix time](https://en.wikipedia.org/wiki/Unix_time).|
|`relative_stop`|false|integer|seconds|Stop time of the schedule relative to `start` time.|
|`stop`|false|integer|seconds|Absolute stop time of the schedule in [Unix time](https://en.wikipedia.org/wiki/Unix_time).|
|`interval`|false?0?|integer|seconds|Interval time between instances of the `action` being performed.|
|`priority`|false|integer|N/A|Priority of the schedule, similar to applying [nice](https://en.wikipedia.org/wiki/Nice_(Unix)). Lower numbers are higher priority.|
|`action`|true|object|N/A|See [Action](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#51-action) object definition.|

### 5.10 Sensor Definition

The `sensor_definition` object requires the following additional name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`antenna`|true|object|N/A|See [Antenna](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#52-antenna) object definition.|
|`signal_conditioning_unit`|true|object|N/A|See [Signal Conditioning Unit](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#511-signal-conditioning-unit) object definition.|
|`data_extraction_unit`|true|object|N/A|See [Data Extraction Unit](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#54-data-extraction-unit) object definition.|
|`host_controller`|false|object|N/A|See [Host Controller](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#56-host-controller) object definition.|

### 5.11 Signal Conditioning Unit

The `signal_conditioning_unit` object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`rf_path_#`|true|integer|N/A||

### 5.12 Spectrum Analyzer

The `spectrum analyzer` object requires the following additional name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`frequency_step`||float|Hz||
|`resolution_bandwidth`||float|Hz||
|`dwell_time`||integer|seconds||
|`video_bandwidth`||float|Hz||

### 5.13 Stepped Frequency FFT

The `stepped_frequency_fft` object requires the following additional name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|||||


# **COMMENTS / WORKING ETC.**

CONTROL PLANE

#### Endpoints
    acquisitions
    actions - List of Action objects
      stepped_center_frequency_dft
      ...
    schedule - List of ScheduleEntry objects
    sensor_definition
      antenna - Antenna object
      signal_conditioning_unit - List of RFPath objects AKA Preselector
      data_extraction_unit - DataExtractionUnit object AKA COTS Sensor/Radio SDR
      host_controller - HostController object
    status
