# SCOS Transfer Specification
Version 1.0

## Abstract

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

The control plane for SCOS is implemented through the use of a RESTful API residing on the sensor - see [scos-sensor](https://github.com/NTIA/scos-sensor/blob/master/docs/api/openapi.adoc). A sensor advertises its **capabilities**, among which are **actions** that you can **schedule** the sensor to do. Some actions acquire data, and those **acquisitions** are retrievable in an easy to use archive format. Acquisitions are "owned" by the schedule entry. Schedule entries are owner by a specific user.

Actions are functions that the sensor owner implements and exposes. Actions can do anything, e.g., rotate an antenna, start streaming data over a websocket and never return. Sensor actions are scheduled via a **schedule entry** post, and the sensor executes a schedule entry to the best of its ability given all actions it is already responsible to perform.

The following objects are used within the `scos` SigMF name space in the Control and Data planes.

### 3.1 Schedule Entry Object
The `schedule_entry` object requires the following name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`name`|true|string|N/A|The unique identification string assigned to the schedule entry.|
|`start`|false|integer|seconds|Absolute start time of the schedule in [Unix time](https://en.wikipedia.org/wiki/Unix_time).|
|`relative_stop`|false|integer|seconds|Stop time of the schedule relative to `start` time.|
|`stop`|false|integer|seconds|Absolute stop time of the schedule in [Unix time](https://en.wikipedia.org/wiki/Unix_time).|
|`interval`|false|integer|seconds|Interval time between instances of the `action` being performed.|
|`priority`|false|integer|N/A|Priority of the schedule, similar to applying [nice](https://en.wikipedia.org/wiki/Nice_(Unix)). Lower numbers are higher priority.|
|`action`|true|string|N/A|Name of action to be performed. See [Action Object](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#51-action) definition.|

### 3.2 Action Object
The `action` object requires the following name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`name`|true|string|N/A|The unique identification string assigned to the action.|
|`summary`|false|string|N/A|A succinct description of the action. Not required, but strongly recommended.|
|`description`|false|string|N/A|A full description of the action. Not required, but strongly recommended.|

## 4. Data Plane
The SCOS specification uses and is fully compliant with the SigMF Specification. Building upon the SigMF [core namespace](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#namespaces), the specification is enhanced through the implementation of a `scos` namespace, the details of which follow.  

### 4.1 Global
Per SigMF, the global object consists of name/value pairs that provide information applicable to the entire dataset. It contains the information that is minimally necessary to open and parse the dataset file, as well as general information about the recording itself.

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`sensor_definition`|false|object|N/A|Describes the sensor model components. See [Sensor Definition](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#510-sensor-definition) object definition. This object is RECOMMENDED.|
|`sensor_id`|true|string|N/A|Unique name for the sensor.|
|`version`|true|string|N/A|The version of the SCOS SigMF name space.|
|`schedule_entry`|false|object|N/A|See [Schedule Entry](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#59-schedule-entry) object definition.|
|`task_id`|false|integer|N/A|A unique identifier that increments with each task of a `schdeule_entry`.|

#### 4.1.1 Sensor Definition Object
Sensor definition follows a simplified hardware model comprised of the following elements: Antenna, Signal Conditioning Unit (SCU), Data Extraction Unit (DEU), and Host Controller (HC). Sensor implementations are not required to have each component, but metadata must specify the presence, model numbers, and operational parameters associated with each.

The following global objects are used within the `scos` SigMF name space to define the sensor.

The `sensor_definition` object requires the following additional name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`antenna`|true|object|N/A|See [Antenna](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#52-antenna) object definition.|
|`signal_conditioning_unit`|true|object|N/A|See [Signal Conditioning Unit](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#511-signal-conditioning-unit) object definition.|
|`data_extraction_unit`|true|object|N/A|See [Data Extraction Unit](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#54-data-extraction-unit) object definition.|
|`host_controller`|false|object|N/A|See [Host Controller](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#56-host-controller) object definition.|

##### Antenna Object
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

##### Data Extraction Unit Object
The `data_extraction_unit` object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`make`|false|string||`Ettus`, `Keysight`, `Tektronix`|
|`model`|false|string||`N210`, `B200`, `N6841A`, `B206B`|
|`low_frequency`|false|float|Hz|Low frequency of operational range of DEU|
|`high_frequency`||float|Hz|High frequency of operational range of DEU|
|`noise_figure`||float|dB|Noise figure of DEU|
|`max_power`||float|dB|Maximum power of DEU|

##### Host Controller Object
The `host_controller` object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`software_version`||string|N/A||
|`cpu`||string|N/A||
|`memory`||integer|GB||
|`storage_size`||integer|GB||

##### Signal Conditioning Unit Object
The `signal_conditioning_unit` object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`number rf_paths`|false|integer||Number of RF paths in SEU|
|`rf_path_spec`|false||object||Specification of RF pathsin SEU|

##### RF Path # Object
The `number_rf_path` object requires the following additional name/value pairs

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`rf_path_number`|false|integer||RF path number|
|`low_frequency_passband`|false|float|Hz|Low frequency of filter 1-dB passband|
|`high_frequency_passband`|false|float|Hz|High frequency of filter 1-dB passband|
|`low_frequency_stopband`|false|float|Hz|Low frequency of filter 60-dB stopband|
|`high_frequency_stopband`|false|float|Hz|High frequency of filter 60-dB stopband|
|`lna_gain`|false|float|dB|Gain of low noise amplifier|
|`lna_noise_figure`|false|float|dB|Noise figure of low noise amplifier|
|`cal_source_type`|false|string||`calibrated noise source`|
|`cal_source_ENR`|false|float|dB|Excess noise ratio of calibrated noise source at frequency of RF path|

### 4.2 Captures
Per SigMF, the captures value is an array of capture segment objects that describe the parameters of the signal capture. The `scos` specification does not add any enhancements to this section.  

### 4.3 Annotations
Per SigMF, the annotations value is an array of annotation segment objects that describe anything regarding the signal data not part of the captures and global objects.

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`altitude`|false|float|meters|The height of the antenna above sea level.|
|`environment`||string||A description of the environment where antenna is mounted: `indoor` or `outdoor`.|
|`dynamic_antenna_settings`|false|object||Dynamic parameters associated with the antenna attached to the sensor|
|`dynamic_SCU_settings`|false|object||attenuation of Signal Conditioning Unit|
|`dynamic_DEU_settings`|false|object||attenuation of Data Extraction Unit|
|`system_to_detect`||string||The system that the measurement is designed to detect: `radar–SPN43`, `lte` or `none`.
|`data_sensitivity`||string||The sensitivity of the data captured: `Low`, `Medium` or  `High`
|`measurement_type`||object||The type of measurement acquired: [`single_frequency_fft`](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#55-single-frequency-fft), [`stepped_frequency_fft`](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#513-stepped-frequency-fft), [`spectrum_analyzer`](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#512-spectrum-analyzer), [`calibration`](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#53-calibration) or [`power_delay_profile`](https://github.com/NTIA/SCOS/blob/master/documents/transfer-spec/SCOSTransferSpec.md#57-power-delay-profile)|
|`temperature`||float|celsius||
|`overload_flag`||boolean||Flag indicator of system signal overload.|
|`detected_system_noise_powers`||float|dBm|The detected system noise power referenced to the output of isotropic antenna.|
|`processed`||boolean||If `true` the data is measured powers, as opposed to `false` indicating raw measured powers|

#### 4.3.1 Dynamic Sensor Settings
The following annotation objects are used within the `scos` SigMF name space associated with dynamic settings in the antenna, SCU, and DEU. They are listed in alphabetical order.  

##### Dynamic Antenna Settings Object
The `dynamic_antenna_settings` object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`azimuth_beam_dir`|false|float|degrees|Angle of maximum antenna gain from North|
|`elevation_beam_dir`|false|float|degrees|Angle of maximum antenna gain from horizontal|
|`polarization`|false|float|string|`Vertical`, `Horizontal`, `Slant-45`, `Left-Hand Circular`, `Right-Hand Circular`|

##### Dynamic DEU Settings Object
The `dynamic_DEU_settings` object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`attenuation`|false|float|dB|Attenuation of DEU|

##### Dynamic SCU Settings Object
The `dynamic_SCU_settings` object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`rf_path_#`|false|float|degrees|Angle of maximum antenna gain from North|

#### 4.3.2 Measurement Types
The following annotation objects are used within the `scos` SigMF name space associated with `measurement_type`. They are listed in alphabetical order.  

##### Calibrations Object
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

##### Single Frequency FFT Object
The `single_frequency_fft` object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`number_of_samples_in_fft`|true|integer||Number of samples in FFT (n) to calcluate delta_f = samplerate/n|
|`window`|true|string||`Blackman-Harris`, `Flattop`, `Gaussian_a3.5`, `Gaussian_a4`, `Gauss Top`, `Hamming`, `Hanning`, `Rectangular`|
|`equivalent_noise_bandwidth`|false|float|Hz||
|`detector`|true|string||`sample_iq`, `sample_power`, `mean_power`, `max_power`, `min_power`, `median_power`|
|`number_of_ffts`|true|integer||Number of FFTs to be integrated over by detector|

##### Power Delay Profile Object
The `power_delay_profile` object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|||||

##### Spectrum Analyzer Swept Frequency Object
The `spectrum_analyzer_swept_frequency` object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`frequency_start`|true|float|Hz|First frequency of scan|
|`frequency_stop`|true|float|Hz|Last frequency of scan|
|`frequency_step`|true|float|Hz|Frequency step of scan|
|`dwell_time`|true|float|seconds|Integration time of detector at each frequency step|
|`resolution_bandwidth`|true|float|Hz|Resolution bandwidth|
|`video_bandwidth`|true|float|Hz|Video bandwidth|

##### Stepped Frequency FFT Object
The `stepped_frequency_fft` object requires the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`center_frequency_start`|true|float|Hz|First center frequency of scan|
|`center_frequency_stop`|true|float|Hz|Last center frequency of scan|
|`center_frequency_step`|true|float|Hz|Center frequency step of scan|
|`single_frequency_fft`|true|object|||

## 5. Index
[Action Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#32-action-object)  
[Annotations](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#43-annotations)  
[Antenna Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#antenna-object)  
[Calibrations Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#calibrations-object)  
[Captures](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#42-captures)  
[Control Plane](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#3-control-plane)  
[Data Extraction Unit Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#data-extraction-unit-object)  
[Data Plane](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#4-data-plane)  
[Dynamic Antenna Settings Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#dynamic-antenna-settings-object)  
[Dynamic DEU Settings Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#dynamic-deu-settings-object)  
[Dynamic SCU Settings Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#dynamic-scu-settings-object)  
[Dynamic Sensor Settings](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#431-dynamic-sensor-settings)  
[Single Frequency FFT Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#single-frequency-fft-object)  
[Global](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#41-global)  
[Host Controller Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#host-controller-object)  
[Measurement Types](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#432-measurement-types)  
[Power Delay Profile Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#power-delay-profile-object)  
[RF Path # Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#rf-path--object)  
[Schedule Entry Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#31-schedule-entry-object)  
[Sensor Definition Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#411-sensor-definition-object)  
[Signal Conditioning Unit Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#signal-conditioning-unit-object)  
[Spectrum Analyzer Swept Frequency Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#spectrum-analyzer-swept-frequency-object)  
[Stepped Frequency FFT Object](https://github.com/NTIA/SCOS-Transfer-Spec/blob/master/README.md#stepped-frequency-fft-object)  
