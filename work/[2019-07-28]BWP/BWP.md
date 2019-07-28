# BWP



**BWP定义**

由于5G采用大带宽，若每个UE都使用全带宽，则对UE能力要求很高。BWP指载波的一部分，让UE只使用其中的一部分，降低UE的成本。

>38.211-4.4.5
A **bandwidth part** is a subset of contiguous common resource blocks for a given numerology   in bandwidth part on a given carrier. 

对应载波和BWP，出现了CRB和PRB。CRB在载波上编号，PRB在BWP上编号，两者之间映射关系如下图：
![CRB-to-PRB](http://www.sharetechnote.com/html/5G/image/NR_CarrierBandwidthPart_38_211_v_2_0_0_01.png "CRB和PRB的映射关系")

$n_{CRB}=n_{PRB}+N_{BWP,j}^{start}$

**BWP配置**

根据载波中心频点和带宽、SSB中心频点进行计算，确定BWP起始位置和大小。
```mermaid
graph LR
A[载波中心频点和带宽] -->B[Point A]
B --> |SSB中心频点|C[SSB偏移]
C --> D[CORESET0 offset]
D --> E[BWP]
```

载波、SSB、CORESET0和BWP的频域关系如下图所示：

![BWP配置](/BWP配置.png "BWP配置")


```c++
SCS-SpecificCarrier ::=             SEQUENCE {
    offsetToCarrier                     INTEGER (0..2199),
    subcarrierSpacing                   SubcarrierSpacing,
    carrierBandwidth                    INTEGER (1..maxNrofPhysicalResourceBlocks),
}
```


```c++
FrequencyInfoDL ::=                 SEQUENCE {
    absoluteFrequencySSB                ARFCN-ValueNR                                                   
    frequencyBandList                   MultiFrequencyBandListNR,
    absoluteFrequencyPointA             ARFCN-ValueNR,
    scs-SpecificCarrierList             SEQUENCE (SIZE (1..maxSCSs)) OF SCS-SpecificCarrier,
    ...
}
```

```c++
FrequencyInfoDL-SIB ::=             SEQUENCE {
    frequencyBandList                   MultiFrequencyBandListNR-SIB,
    offsetToPointA                      INTEGER (0..2199),
    scs-SpecificCarrierList             SEQUENCE (SIZE (1..maxSCSs)) OF SCS-SpecificCarrier
}
```


```c++
MIB ::=                             SEQUENCE {
    systemFrameNumber                   BIT STRING (SIZE (6)),
    subCarrierSpacingCommon             ENUMERATED {scs15or60, scs30or120},
    ssb-SubcarrierOffset                INTEGER (0..15),
    dmrs-TypeA-Position                 ENUMERATED {pos2, pos3},
    pdcch-ConfigSIB1                    PDCCH-ConfigSIB1,
    cellBarred                          ENUMERATED {barred, notBarred},
    intraFreqReselection                ENUMERATED {allowed, notAllowed},
    spare                               BIT STRING (SIZE (1))
}
```
```c++
BWP ::=                             SEQUENCE {
    locationAndBandwidth                INTEGER (0..37949),
    subcarrierSpacing                   SubcarrierSpacing,
    cyclicPrefix                        ENUMERATED { extended }                                                 OPTIONAL    -- Need R
}
```

**如何确定BWP的位置和带宽？**

通过参数locationAndBandwidth进行配置，利用RIV指示起始位置和BWP带宽，默认$N_{BWP}^{size}=275$

RIV计算如下图所示：

![RIV](http://www.sharetechnote.com/html/5G/image/NR_RIV_01.png "RIV")

```python {cmd=true,output="html"}
# 基站侧：根据BWP起始位置和带宽计算RIV
rbLength = 273
rbStart = 0
bwpSize = 275
if ((rbLength-1) <= (bwpSize//2)):
    RIV = bwpSize*(rbLength-1)+rbStart
else
    RIV = bwpSize*(bwpSize-rbLength+1)+(bwpSize-1-rbStart)
print("RIV=%d",RIV)
```

```python {cmd=true,output="html"}
# UE侧：根据RIV计算BWP起始位置和带宽
RIV = 1099
bwpSize = 275

rbLength = RIV//bwpSize + 1
rbStart = RIV%bwpSize
if ((rbLength-1) > (bwpSize//2)):
    rbLength = bwpSize - rbLength + 1
    rbStart = bwpSize - 1 - rbStart

print("rbStart=%d,rbLength=%d",rbStart,rbLength)

```
**BWP切换**
此部分暂时不理解，后续更新。
