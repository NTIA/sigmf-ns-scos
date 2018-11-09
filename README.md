# SCOS SigMF Namespace

## Abstract
The SCOS SigMF namespace defines a standard for the controls and data format used within the [IEEE 802.22.3 Draft Standard: Spectrum Characterization and Occupancy Sensing (SCOS) system](http://www.ieee802.org/22/P802_22_3_PAR_Detail_Approved.pdf).

## Table of Contents

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->

- [SCOS Transfer Specification](#scos-transfer-specification)
    - [Abstract](#abstract)
    - [Table of Contents](#table-of-contents)
    - [1. Description](#1-description)
    - [2. Conventions Used in this Document](#2-conventions-used-in-this-document)
    - [3. Global](#3-global)
        - [3.1 Sensor Object](#31-sensor-object)
            - [Antenna Object](#antenna-object)
            - [Receiver Object](#receiver-object)
            - [Preselector Object](#preselector-object)
            - [RFPath Object](#rfpath-object)
        - [3.2 Transmitter Object](#32-transmitter-object)
        - [3.3 ScheduleEntry Object](#33-scheduleentry-object)
        - [3.4 Digital Filter Object](#34-digitalfilter-object)
    - [4. Captures](#4-captures)
    - [5. Annotations](#5-annotations)
        - [5.1 Dynamic Sensor Settings](#51-dynamic-sensor-settings)
            - [DynamicAntennaSettings Object](#dynamicantennasettings-object)
            - [DynamicReceiverSettings Object](#dynamicreceiversettings-object)
            - [DynamicPreselectorSettings Object](#dynamicpreselectorsettings-object)
        - [5.2 Measurement Types](#52-measurement-types)
            - [TimeDomainDetection Object](#timedomaindetection-object)
            - [FrequencyDomainDetection Object](#frequencydomaindetection-object)
            - [SteppedFrequencyMeasurement Object](#steppedfrequencymeasurement-object)
            - [SweptTunedMeasurement Object](#swepttunedmeasurement-object)
            - [YFactorCalibration Object](#yfactorcalibration-object)
    - [6. Index](#6-index)

<!-- markdown-toc end -->

## 1. Description
This SCOS SigMF namespace defines the data format used within the NTIA implementation of the [IEEE 802.22.3 draft standard](http://www.ieee802.org/22/P802_22_3_PAR_Detail_Approved.pdf) - also called the NTIA Spectrum Characterization and Occupancy Sensing (SCOS) system. This system fully implements the [SigMF Specification](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md) for the data plane. The goal of SCOS is to standardize the control method and data transfer format for “time share” access to a networked fleet of sensors. Use cases require a solid solution for basic sensor control and RF data collection with flexibility to incorporate new sensing solutions and metrics in the future.

## 2. Conventions Used in this Document

The SCOS specification uses and is fully compliant with the SigMF specification and conventions. Building upon the SigMF [core namespace](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#namespaces), the specification is enhanced through the implementation of a `scos` namespace, the details of which follow.  

We have adopted SigMF's conventions:
- The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).
- JSON keywords are used as defined in [ECMA-404](http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf).
- Augmented Backus-Naur form (ABNF) is used as defined by [RFC5234](https://tools.ietf.org/html/rfc5234) and updated by [RFC7405](https://tools.ietf.org/html/rfc7405).
- Fields defined as "human-readable", a "string", or simply as "text" shall be treated as plaintext where whitespace is significant, unless otherwise specified.
- Information is given as global, capture, or annotation

## 3. Global
Global informataion is applicable to the entire dataset.

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`sensor_id`|true|string|N/A|Unique name for the sensor.|
|`sensor_definition`|false|object|N/A|Describes the sensor model components. See [Sensor Object](#31-sensor-object) definition. This object is RECOMMENDED.|
|`transmitter_definition`|false|object|N/A|The transmitter that the measurement is designed to detect, e.g., propagation measurements. See [Transmitter Object](#32-transmitter-object) definition.|
|`version`|true|string|N/A|The version of the SigMF SCOS namespace extension.|
|`schedule_entry`|false|object|N/A|See [ScheduleEntry Object](#33-scheduleentry-object) definition.|
|`task_id`|false|integer|N/A|A unique identifier that increments with each task of a `schedule_entry`.|
|`anti_aliasing_filter`|false|object|N/A|Describes anti-aliasing low-pass filter applied to IQ captures. See [DigitalFilter Object](#34-digital-filter-object) definition.|

### 3.1 Sensor Object
Sensor definition follows a simplified hardware model comprised of the following elements: Antenna, Preselector, Receiver, and Host Controller. The antenna converts electromagnetic energy to a voltage. The preselector can provide local calibration signals, RF filtering to protect from strong out-of-band signals, and low-noise amplification to improve sensitivity. The receiver (e.g., software defined radio) provides tuning, downcoversion, sampling, and digital signal processing. Sensor implementations are not required to have each component, but metadata SHOULD specify the presence, model numbers, and operational parameters associated with each.

The `Sensor` object contains the following name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`antenna`|true|object|N/A|See [Antenna Object](#antenna-object) definition.|
|`preselector`|false|object|N/A|See [Preselector Object](#preselector-object) definition.|
|`receiver`|true|object|N/A|See [Receiver Object](#receiver-object) definition.|
|`host_controller`|false|string|N/A|Description of host computer. E.g. Make, model, and configuration.|

#### Antenna Object
The `Antenna` object contains the following name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`model`|true|string|N/A|Antenna make and model number. E.g. `"ARA CSB-16"`, `"L-com HG3512UP-NF"`.|
|`type`|false|string|N/A|Antenna type. E.g. `"dipole"`, `"biconical"`, `"monopole"`, `"conical monopole"`.|
|`low_frequency`|false|float|Hz|Low frequency of operational range.|
|`high_frequency`|false|float|Hz|High frequency of operational range.|
|`gain`|false|float|dBi|Antenna gain in direction of maximum radiation or reception.|
|`horizontal_gain_pattern`|false|array of floats|dBi|Antenna gain pattern in horizontal plane from 0 to 359 degrees in 1 degree steps.|
|`vertical_gain_pattern`|false|array of floats|dBi|Antenna gain pattern in vertical plane from -90 to +90 degrees in 1 degree steps.|
|`horizontal_beam_width`|false|float|degrees|Horizontal 3-dB beamwidth.|
|`vertical_beam_width`|false|float|degrees|Vertical 3-dB beamwidth.|
|`cross_polar_discrimination`|false|float|N/A|Cross-polarization discrimination.|
|`voltage_standing_wave_ratio`|false|float|volts|Voltage standing wave ratio.|
|`cable_loss`|false|float|dB|Cable loss for cable connecting antenna and preselector.|
|`steerable`|false|boolean|N/A|Defines if the antenna is steerable or not.|
|`mobile`|false|boolean|N/A|Defines if the antenna is mobile or not.|

#### Receiver Object
The `Receiver` object contains the following name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`model`|true|string|N/A|Make and model of the receiver. E.g., `"Ettus N210"`, `"Ettus B200"`, `"Keysight N6841A"`, `"Tektronix B206B"`.|
|`low_frequency`|false|float|Hz|Low frequency of operational range of the receiver.|
|`high_frequency`|false|float|Hz|High frequency of operational range of the receiver.|
|`noise_figure`|false|float|dB|Noise figure of the receiver.|
|`max_power`|false|float|dBm|Maximum input power of the receiver.|

#### Preselector Object
The `Preselector` object contains the following name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`rf_paths`|false|array of objects|N/A|Specification of the preselector RF paths via [RFPath Object](#rfpath-object).|

#### RFPath Object
Each `RFPath` object contains the following name/value pairs:

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

### 3.2 Transmitter Object
The `Transmitter` object contains the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`system_name`|false|string|N/A|Name of system to be detected.|
|`transmit_power`|false|float|dBm|Transmitter power going into antenna.|
|`antenna`|true|object|N/A|See [Antenna Object](#antenna-object) definition.|
|`waveform`|false|object|N/A|See SigMF [waveform namespace](https://github.com/NTIA/sigmf-ns-waveform)
|`latitude`|false|float|degrees|Latitude.|
|`longitude`|false|float|degrees|Longitude.|
|`altitude`|false|float|meters|Altitude above mean sea level.|

### 3.3 ScheduleEntry Object
The `ScheduleEntry` object is associated with the SCOS control plane, which is implemented through the use of a RESTful API residing on the sensor, see [SCOS Sensor](https://github.com/NTIA/scos-sensor) and [SCOS Control Plane API Reference](https://ntia.github.io/scos-sensor/). A sensor advertises its **capabilities**, among which are **actions** that users can schedule the sensor to do. Sensor actions are scheduled by posting a **schedule entry** to the sensor's **schedule**. The scheduler periodically reads the schedule and populates a task queue in priority order. A **task** represents an action at a _specific_ time. Therefore, a schedule entry represents a range of tasks. The scheduler continues populating its task queue until the schedule is exhausted. When executing the task queue, the scheduler makes a best effort to run each task at its designated time, but the scheduler SHOULD NOT in most cases cancel a running task to start another task, even of higher priority. **priority** is used to disambiguate two or more tasks that are scheduled to start at the same time.

The `ScheduleEntry` object contains the following name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`name`|true|string|N/A|The identification string assigned to the schedule entry. MUST be unique on the sensor.|
|`start`|false|datetime|[ISO-8601](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#the-datetime-pair)|Requested time to schedule the first task. If unspecified, task will start as soon as possible.|
|`relative_stop`|false*|integer|seconds|Relative seconds after `start` when the task will end.|
|`absolute_stop`|false*|datetime|[ISO-8601](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#the-datetime-pair)|Absolute time when the task will end.| 
|`interval`|false|integer|seconds|Interval between tasks. If unspecified, task will run exactly once and then mark the entry inactive.|
|`priority`|false|integer|N/A|Priority of the entry, similar to applying [nice](https://en.wikipedia.org/wiki/Nice_(Unix)). Lower numbers are higher priority. Default is 10.|
|`action`|true|string|N/A|Name of action to be performed.|

\* If both `relative_stop` and `absolute_stop` are unspecified, the task will carry on forever. Specifying both `relative_stop` and `absolute_stop` will result in an error.

### 3.4 DigitalFilter Object
Each `DigitalFilter` object contains the following name/value pairs:

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`type`|false|string|N/A|Description of digital filter, e.g., `"FIR"`: Finite impulse response|
|`length`|false|integer|N/A|Number of taps.|
|`frequency_cutoff`|false|float|Hz|Frequency at which the magnitude response decreases (from its maximum) by `attenuation_cutoff`.|
|`attenuation_cutoff`|false|float|dB|Magnitude response threshold (below maximum) that specifies `frequency_cutoff`.|
|`ripple_passband`|false|float|dB|Ripple in passband.|
|`attenuation_stopband`|false|float|dB|Attenuation of stopband.|
|`frequency_stopband`|false|float|Hz|Point in filter frequency response where stopband starts.|

## 4. Captures
Per SigMF, the `Captures` value is an array of capture segment objects that describe the parameters of the signal capture. The `scos` specification does not add any enhancements to this section.

## 5. Annotations
Per SigMF, the `Annotations` value is an array of annotation segment objects that describe anything regarding the signal data not part of the `global` and `captures` objects. Each SigMF annotation segment object must contain a `core:sample_start` name/value pair, which indicates the first index at which the rest of the segment's name/value pairs apply.

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`altitude`|false|float|meters|The height of the antenna above mean sea level.|
|`environment`|false|string|N/A|A description of the environment where antenna is mounted. E.g. `"indoor"` or `"outdoor"`.|
|`dynamic_antenna_settings`|false|object|N/A|Dynamic parameters associated with the antenna. See [DynamicAntennaSettings Object](#dynamicantennasettings-object) definition.|
|`dynamic_preselector_settings`|false|object|N/A|Dynamic parameters associated with the preselector. See [DynamicPreselectorSettings Object](#dynamicpreselectorsettings-object) definition.|
|`dynamic_receiver_settings`|false|object|N/A|Dynamic parameters associated with the receiver. See [DynamicReceiverSettings Object](#dynamicreceiversettings-object) definition.|
|`transmitter_identification`|false|object|N/A|Transmitter identification parameters. See [Transmitter Object](#32-transmitter-object) definition.|
|`measurement_type`|true|object|N/A|The type of measurement acquired, e.g., [SteppedFrequencyMeasurement](#steppedfrequencymeasurement-object), [SweptTunedMeasurement](#swepttunedmeasurement-object) or [YFactorCalibration](#yfactorcalibration-object).|
|`data_sensitivity`|false|string|N/A|The sensitivity of the data captured. E.g. `"low"`, `"moderate"` or  `"high"`.|
|`detected_system_noise_powers`|false|float|dBm|The detected system noise power referenced to the output of isotropic antenna.|
|`temperature`|false|float|celsius|Environmental temperature.|
|`overload_flag`|false|boolean|N/A|Flag indicator of system signal overload.|

### 5.1 Dynamic Sensor Settings
The following annotation objects are used within the `scos` SigMF name space associated with dynamic settings in the antenna, the preselector, and the receiver.

#### DynamicAntennaSettings Object
The `DynamicAntennaSettings` object contains the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`azimuth_angle`|false|float|degrees|Angle of main beam in azimuthal plane from North.|
|`elevation_angle`|false|float|degrees|Angle of main beam in elevation plane from horizontal.|
|`polarization`|false|float|string|E.g. `"vertical"`, `"horizontal"`, `"slant-45"`, `"left-hand circular"`, `"right-hand circular"`.|

#### DynamicReceiverSettings Object
The `DynamicReceiverSettings` object contains the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`attenuation`|false|float|dB|Attenuation of the receiver.|

#### DynamicPreselectorSettings Object
The `DynamicPreselectorSettings` object contains the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`rf_path_number`|false|integer|N/A|The preselector RF path number.|

### 5.2 Measurement Types
The following annotation objects are used within the `scos` SigMF name space associated with `measurement_type`. 

#### TimeDomainDetection Object
Time-domain detection algorithms are applied to IQ time series captured at a single frequency. The `TimeDomainDetection` object contains the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`detector`|true|string|N/A|E.g. `"sample_power"`, `"mean_power"`, `"max_power"`, `"min_power"`, `"median_power"`, `"m4s_power"`.|
|`detection_domain`|true|string|N/A|Domain in which detector is applied, i.e., `"time"`.|
|`number_of_samples`|true|integer|N/A|Number of samples to be integrated over by detector.|
|`units`|true|string|N/A|Data units, e.g., `"dBm"`, `"watts"`, `"volts"`.|
|`reference`|false|string|N/A|Data reference point, e.g., `"receiver input"`, `"antenna output"`, `"output of isotropic antenna"`.|

#### FrequencyDomainDetection Object
Frequency-domain detection algorithms are applied to discrete Fourier transforms of IQ time series captured at a single frequency. The `FrequencyDomainDetection` object contains the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`detector`|true|string|N/A|E.g. `"fft_sample_iq"`, `"fft_sample_power"`, `"fft_mean_power"`, `"fft_max_power"`, `"fft_min_power"`, `"fft_median_power"`, `"fft_m4s_power"`.|
|`detection_domain`|true|string|N/A|Domain in which detector is applied, i.e., `"frequency"`.|
|`number_of_ffts`|true|integer|N/A|Number of FFTs to be integrated over by detector.|
|`units`|true|string|N/A|Data units, e.g., `"dBm"`, `"watts"`, `"volts"`.|
|`reference`|false|string|N/A|Data reference point, e.g., `"receiver input"`, `"antenna output"`, `"output of isotropic antenna"`.|
|`number_of_samples_in_fft`|true|integer|N/A|Number of samples in FFT to calcluate delta_f = [`samplerate`](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#global-object)/`number_of_samples_in_fft`.|
|`window`|true|string|N/A|E.g. `"blackman-harris"`, `"flattop"`, `"gaussian_a3.5"`, `"gauss top"`, `"hamming"`, `"hanning"`, `"rectangular"`.|
|`equivalent_noise_bandwidth`|false|float|Hz|Bandwidth of brickwall filter that has same integrated noise power as that of the actual filter.|

#### SteppedFrequencyMeasurement Object
The `SteppedFrequencyMeasurement` object contains the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`frequencies`|false|array of floats|Hz|Center frequencies where stepped-frequency detections are performed.|
|`algorithm`|true|object|N/A|Algorithm applied to IQ samples. See [TimeDomainDetection Object](#timedomaindetection-object), [FrequencyDomainDetection Object](#frequencydomaindetection-object) definition.|

Example `SteppedFrequencyMeasurement` object for case where mean-power measurements are performed at five frequencies:

```
{
  "frequencies": [100000000, 200000000, 300000000, 400000000, 500000000],
  "algorithm": {
    "detector": "mean_power",
    "number_of_samples": 100000,
    "units": "dBm",
    "reference": "output of isotropic antenna"
  }
}
```

#### SweptTunedMeasurement Object
Swept-tuned measurement is a standard spectrum analyzer measurement. The `SweptTunedMeasurement` object contains the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`frequency_start`|true|float|Hz|First frequency of scan.|
|`frequency_stop`|true|float|Hz|Last frequency of scan.|
|`frequency_step`|true|float|Hz|Frequency step of scan.|
|`dwell_time`|true|float|seconds|Integration time of detector at each frequency step.|
|`resolution_bandwidth`|true|float|Hz|Resolution bandwidth.|
|`video_bandwidth`|true|float|Hz|Video bandwidth.|
|`units`|true|string|N/A|Data units, e.g., `"dBm"`, `"watts"`, `"volts"`.|
|`reference`|false|string|N/A|Data reference point, e.g., `"receiver input"`, `"antenna output"`, `"output of isotropic antenna"`.|

#### YFactorCalibration Object
The purpose of the `YFactorCalibration` is to provide parameters and results for time-domain y-factor calibrations. The `YFactorCalibration` object contains the following name/value pairs:  

|name|required|type|unit|description|
|----|--------------|-------|-------|-----------|
|`last_time_performed`|true|datetime|[ISO-8601](https://github.com/gnuradio/SigMF/blob/master/sigmf-spec.md#the-datetime-pair)|Date and time that calibration was performed.|
|`frequencies`|false|array of floats|Hz|Frequencies that y-factor calibrations are performed.|
|`excess_noise_ratios`|false|array of floats|dB|Excess noise ratio of calibrated noise source at `frequencies` of y-factor calibration.|
|`receiver_setting_name`|false|string|N/A|Name of adjustable receiver setting that affects noise figure, e.g., `"attenuation"`, `"input range"`.|
|`receiver_setting_units`|false|string|N/A|Units corresponding to `receiver_setting_name`, e.g., `"dB"`, `"dBm"`.|
|`reference`|false|string|N/A|Data reference point, e.g., `"receiver input"`, `"antenna output"`, `"preselector input"`.|
|`calibrations`|false|array of objects|dB|Receiver settings and corresponding `gains` and `noise_figures` arrays (i.e., arrays of floats with lengths equal to the length of `frequencies`.)|

Example `YFactorCalibration` object for case where calibrations are performed at two receiver settings and five frequencies:

```
{
  "frequencies": [100000000, 200000000, 300000000, 400000000, 500000000],
  "excess_noise_ratios": [12.3, 12.4, 12.7, 12.6, 12.5],
  "receiver_setting_name": "input range",
  "receiver_setting_units": "dBm",
  "reference": "preselector input",
  "calibrations":
  [
    {
      "receiver_setting": -30,
      "gains": [7.1, 7.2, 7.0, 7.1, 7.4],
      "noise_figures": [9.1, 9.2, 9.0, 9.1, 9.4]
    },
    {
      "receiver_setting": -20,
      "gains": [6.1, 6.2, 6.0, 6.1, 6.4],
      "noise_figures": [9.1, 9.2, 9.0, 9.1, 9.4]
    }
  ]
}
```

## 6. Index 
[Annotations](#5-annotations)  
[Antenna Object](#antenna-object)  
[Captures](#4-captures)  
[DigitalFilter Object](#34-digitalfilter-object)  
[Dynamic Sensor Settings](#51-dynamic-sensor-settings)  
[DynamicAntennaSettings Object](#dynamicantennasettings-object)  
[DynamicPreselectorSettings Object](#dynamicpreselectorsettings-object)  
[DynamicReceiverSettings Object](#dynamicreceiversettings-object)  
[Global](#3-global)  
[Measurement Types](#52-measurement-types)  
[Preselector Object](#preselector-object)  
[Receiver Object](#receiver-object)  
[RFPath Object](#rfpath-object)  
[ScheduleEntry Object](#33-scheduleentry-object)  
[Sensor Object](#31-sensor-object)  
[TimeDomainDetection Object](#timedomaindetection-object)  
[FrequencyDomainDetection Object](#frequencydomaindetection-object)  
[SteppedFrequencyMeasurement Object](#steppedfrequencymeasurement-object)  
[SweptTunedMeasurement Object](#swepttunedmeasurement-object)  
[Transmitter Object](#32-transmitter-object)  
[YFactorCalibration Object](#yfactorcalibration-object)  
