---
layout: post
title: "利用MATLAB 5G Toolbox生成下行载波"
comments: true
description: "利用MATLAB生成SSB、CORESET0、PDCCH、PDSCH"
keywords: "SSB, CORESET0, PDCCH, PDSCH"
---

5GNR下行信道和信号包括：
- SSB
- PDCCH和DMRS-PDCCH
- PDSCH和DMRS-PDSCH

Table 5.3.2-1: Maximum transmission bandwidth configuration NRB
| SCS \(kHz\) | 5 MHz | 10 MHz | 15 MHz | 20 MHz | 25 MHz | 30 MHz | 40 MHz | 50 MHz | 60 MHz | 80 MHz | 90 MHz | 100 MHz |
|-------------|-------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|---------|
|             | NRB   | NRB    | NRB    | NRB    | NRB    | NRB    | NRB    | NRB    | NRB    | NRB    | NRB    | NRB     |
| 15          | 25    | 52     | 79     | 106    | 133    | 160    | 216    | 270    | N/A    | N/A    | N/A    | N/A     |
| 30          | 11    | 24     | 38     | 51     | 65     | 78     | 106    | 133    | 162    | 217    | 245    | 273     |
| 60          | N/A   | 11     | 18     | 24     | 31     | 38     | 51     | 65     | 79     | 107    | 121    | 135     |


Table 5.3.3-1: Minimum guardband for each UE channel bandwidth and SCS (kHz)
| SCS \(kHz\) | 5 MHz  | 10 MHz | 15 MHz | 20 MHz | 25 MHz | 30 MHz | 40 MHz | 50 MHz | 60 MHz | 80 MHz | 90 MHz | 100 MHz |
|-------------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|--------|---------|
| 15          | 242\.5 | 312\.5 | 382\.5 | 452\.5 | 522\.5 | 592\.5 | 552\.5 | 692\.5 | N/A    | N/A    | N/A    | N/A     |
| 30          | 505    | 665    | 645    | 805    | 785    | 945    | 905    | 1045   | 825    | 925    | 885    | 845     |
| 60          | N/A    | 1010   | 990    | 1330   | 1310   | 1290   | 1610   | 1570   | 1530   | 1450   | 1410   | 1370    |


![5G NR信道频段分配](https://raw.githubusercontent.com/ykqin/imageData/master/blog/38101-1-fig5.3.1-1-NRchannel.png)


利用MATLAB进行BWP配置，包括以下步骤： 
[TOC]


# 1 波形和载波配置

配置载波频段、带宽、小区ID
```matlab
waveconfig = [];
waveconfig.NCellID = 0;            % Cell identity
waveconfig.ChannelBandwidth = 100; % Channel bandwidth (MHz)
waveconfig.FrequencyRange = 'FR1'; % 'FR1' or 'FR2'
waveconfig.NumSubframes = 10;      % Number of 1ms subframes in generated waveform (1,2,4,8 slots per 1ms subframe, depending on SCS)
waveconfig.DisplayGrids = 1;       % Display the resource grids after signal generation

carriers(1).SubcarrierSpacing = 30;
carriers(1).NRB = 273;
carriers(1).RBStart = 0;           % RRC参数 offsetToCarrier

% 计算Point A
scs = carriers(1).SubcarrierSpacing;
rbNum = carriers(1).NRB;
rbStart = carriers(1).RBStart;                    
point_a = - (rbNum*12/2)*scs - (rbStart*12*scs); % 单位 kHz
```

# 2 SSB配置

```matlab
ssburst = [];
ssburst.Enable = 1;                             % Enable SS Burst
ssburst.BlockPattern = 'Case B';                % Case B (30kHz) subcarrier spacing
ssburst.SSBTransmitted = [1 0 1 0 1 0 1 0];     % Bitmap indicating blocks transmitted in a 5ms half-frame burst
ssburst.SSBPeriodicity = 20;                    % SS burst set periodicity in ms (5, 10, 20, 40, 80, 160)
ssburst.Power = 0;                              % Power scaling in dB

offsetToPointA = 50;                            % RRC参数 RB级偏移 [RB] 
ssbSubcarrierOffset = 8;                        % RRC参数 RE级偏移 [RE]
ssbFrequencyOffset = (offsetToPointA+10)*12*30 + ssbSubcarrierOffset*15; % SSB起点相对于Point A的频率偏移 [kHz]
ssburst.FrequencySSB = (point_a + ssbFrequencyOffset)*1000;              % 相对于载波中心频点的偏移 (5kHz的倍数)
```

# 3 BWP配置
![BWP](https://ww2.mathworks.cn/help/examples/5g/win64/xxbwp.png)

```matlab
bwp = [];
bwp(1).SubcarrierSpacing = 30;          % BWP Subcarrier Spacing
bwp(1).CyclicPrefix = 'Normal';         % BWP Cyclic prefix for 15 kHz
bwp(1).NRB = 100;                       % Size of BWP
bwp(1).RBOffset = 0;                    % BWP和载波之间的偏移                 
```

# 4 CORESET和搜索空间配置

![BWP](https://ww2.mathworks.cn/help/examples/5g/win64/xxcoresetAlloc.png)

```matlab
coreset = [];
coreset(1).AllocatedSymbols = [0];      % First symbol of each CORESET monitoring opportunity in a slot
coreset(1).AllocatedSlots = [0];        % Allocated slots within a period
coreset(1).AllocatedPeriod = 5;         % Allocated slot period (empty implies no repetition)
coreset(1).Duration = 2;                % CORESET symbol duration (1,2,3)
coreset(1).AllocatedPRB = 6*[1,2,3,4];  % 0111100 RRC参数 frequencyDomainResources
```

```matlab
pdcch = [];
pdcch(1).Enable = 1;                    % Enable PDCCH config
pdcch(1).BWP = 1;                       % Bandwidth part
pdcch(1).Power = 1.1;                   % Power scaling in dB

% 配置搜索空间
pdcch(1).AllocatedSearchSpaces = [0,3]; % Index within the CORESET 
pdcch(1).CORESET = 1;                     % Control resource set ID which carries this PDCCH
pdcch(1).AllocatedPeriod = [];            % Allocation slot period (empty implies no repetition)
% CCE资源分配
pdcch(1).NumCCE = 2;                    % Number of CCE used by PDCCH
pdcch(1).StartCCE = 2;                  % Starting CCE of PDCCH

pdcch(1).RNTI = 0;                      % RNTI
pdcch(1).NID = 1;                       % PDCCH and DM-RS scrambling NID 
pdcch(1).PowerDMRS = 0;                 % Additional power boosting in dB
pdcch(1).DataBlkSize = 20;              % DCI payload size
pdcch(1).DataSource = 'PN9';            % DCI data source
```

# 5 PDSCH配置

![PDSCH分配](https://ww2.mathworks.cn/help/examples/5g/win64/xxpdschAlloc.png)

```matlab
pdsch = [];
pdsch(1).Enable = 1;                    % Enable PDSCH config
pdsch(1).BWP = 1;                       % Bandwidth part
pdsch(1).Power = 0;                     % Power scaling in dB
pdsch(1).DataSource = 'PN9';            % Transport block data source 
pdsch(1).TargetCodeRate = 0.4785;       % Code rate used to calculate transport block sizes
pdsch(1).Xoh_PDSCH = 0;                 % Rate matching overhead
pdsch(1).Modulation = 'QPSK';           % 'QPSK', '16QAM', '64QAM', '256QAM'
pdsch(1).NLayers = 1;                   % Number of PDSCH layers
pdsch(1).RVSequence = [0,2,3,1];        % RV sequence to be applied cyclically across the PDSCH allocation sequence

% PDSCH配置
pdsch(1).AllocatedSymbols = 2:10;      % Range of symbols in a slot
pdsch(1).AllocatedSlots = [0,1,2,3,5,6,7];       % Allocated slot indices
pdsch(1).AllocatedPeriod = 10;         % Allocation period in slots (empty implies no repetition)
pdsch(1).AllocatedPRB = [5:20];        % PRB allocation
pdsch(1).RNTI = 0;                     % RNTI
pdsch(1).NID = 1;                      % Scrambling for data part

pdsch(1).RateMatch(1).CORESET = [1];                  % Rate matching pattern, defined by one CORESET
pdsch(1).RateMatch(1).Pattern.AllocatedPRB = [];      % Rate matching pattern, defined by set of 'bitmaps'
pdsch(1).RateMatch(1).Pattern.AllocatedSymbols = [];
pdsch(1).RateMatch(1).Pattern.AllocatedSlots = [];
pdsch(1).RateMatch(1).Pattern.AllocatedPeriod = [];

% DMRS配置
pdsch(1).PortSet = 0:pdsch(1).NLayers-1; % DM-RS ports to use for the layers
pdsch(1).PDSCHMappingType = 'A';         % PDSCH mapping type ('A'(slot-wise),'B'(non slot-wise))
pdsch(1).DL_DMRS_typeA_pos = 2;          % Mapping type A only. First DM-RS symbol position (2,3)
pdsch(1).DL_DMRS_max_len = 1;            % Number of front-loaded DM-RS symbols (1(single symbol),2(double symbol))      
pdsch(1).DL_DMRS_add_pos = 0;            % Additional DM-RS symbol positions (max range 0...3)
pdsch(1).DL_DMRS_config_type = 2;        % DM-RS configuration type (1,2)
pdsch(1).NIDNSCID = 1;                   % Scrambling identity (0...65535)
pdsch(1).NSCID = 0;                      % Scrambling initialisation (0,1)
pdsch(1).PowerDMRS = 0;                  % Additional power boosting in dB
```


# 6 波束生成

```matlab
waveconfig.SSBurst = ssburst;
waveconfig.Carriers = carriers;
waveconfig.BWP = bwp;
waveconfig.CORESET = coreset;
waveconfig.PDCCH = pdcch;
waveconfig.PDSCH = pdsch;

% Generate complex baseband waveform
[waveform,bwpset] = hNRDownlinkWaveformGenerator(waveconfig);

disp('Information associated to BWP 1:')
disp(bwpset(1).Info)
```

```
Information associated to BWP 1:
           SamplingRate: 122880000
                   Nfft: 4096
              Windowing: 10
    CyclicPrefixLengths: [352 288 288 288 288 288 288 288 288 288 288 288 288 288 352 288 288 288 288 288 288 288 288 288 288 288 288 288]
          SymbolLengths: [1×28 double]
           NSubcarriers: 3276
      SubcarrierSpacing: 30
         SymbolsPerSlot: 14
       SlotsPerSubframe: 2
     SymbolsPerSubframe: 28
     SamplesPerSubframe: 122880
         SubframePeriod: 1.0000e-03
              Midpoints: [1×281 double]
          WindowOverlap: [10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10]
                     k0: 0
```



![BWP配置](https://raw.githubusercontent.com/ykqin/imageData/master/blog/DownlinkResourceinCarrier.jpg)


![载波配置](https://raw.githubusercontent.com/ykqin/imageData/master/blog/carrierConfig.jpg)


![PDCCH与PDSCH的关系](https://raw.githubusercontent.com/ykqin/imageData/master/blog/BWPinCarrier.jpg)