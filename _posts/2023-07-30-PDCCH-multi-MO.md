---
layout: post
title: "PDCCH多MO"
comments: true
description: ""
keywords: "URLLC"
---


# 1 PDCCH MO Case
case1：PDCCH盲检周期大于等于14symbol
case 1-1：slot的前三个符号     ---- eMBB（FG3-1）
case 1-2：slot的任意连续三个符号    ---- LTE-NR共存场景（FG3-2）
case2：PDCCH盲检周期小于14symbol  ---- URLLC场景（FG3-5b）

# 2 UE feature
| **Index** | **Feature group** | **Components** | **Field name in TS 38.331 [2]** |
| --- | --- | --- | --- |
| 3-1 | Basic DL control channel | 1) One configured CORESET per BWP per cell in addition to CORESET0
- CORESET resource allocation of 6RB bit-map and duration of 1 – 3 OFDM symbols for FR1
- For type 1 CSS without dedicated RRC configuration and for type 0, 0A, and 2 CSSs, CORESET resource allocation of 6RB bit-map and duration 1-3 OFDM symbols for FR2
- For type 1 CSS with dedicated RRC configuration and for type 3 CSS, UE specific SS, CORESET resource allocation of 6RB bit-map and duration 1-2 OFDM symbols for FR2
- REG-bundle sizes of 2/3 RBs or 6 RBs
- Interleaved and non-interleaved CCE-to-REG mapping
- Precoder-granularity of REG-bundle size
- PDCCH DMRS scrambling determination
- TCI state(s) for a CORESET configuration
2) CSS and UE-SS configurations for unicast PDCCH transmission per BWP per cell
- PDCCH aggregation levels 1, 2, 4, 8, 16
- UP to 3 search space sets in a slot for a scheduled SCell per BWP
This search space limit is before applying all dropping rules.
- For type 1 CSS with dedicated RRC configuration, type 3 CSS, and UE-SS, the monitoring occasion is within the first 3 OFDM symbols of a slot
- For type 1 CSS without dedicated RRC configuration and for type 0, 0A, and 2 CSS, the monitoring occasion can be any OFDM symbol(s) of a slot, with the monitoring occasions for any of Type 1- CSS without dedicated RRC configuration, or Types 0, 0A, or 2 CSS configurations within a single span of three consecutive OFDM symbols within a slot
3) Monitoring DCI formats 0_0, 1_0, 0_1, 1_1
4) Number of PDCCH blind decodes per slot with a given SCS follows Case 1-1 table
5) Processing one unicast DCI scheduling DL and one unicast DCI scheduling UL per slot per scheduled CC for FDD
6) Processing one unicast DCI scheduling DL and 2 unicast DCI scheduling UL per slot per scheduled CC for TDD |  |
| 3-2 | PDCCH monitoring on any span of up to 3 consecutive OFDM symbols of a slot | For a given UE, all search space configurations are within the same span of 3 consecutive OFDM symbols in the slot. | _pdcchMonitoringSingleOccasion_ |
| 3-5b | All PDCCH monitoring occasion can be any OFDM symbol(s) of a slot for Case 2 with a span gap | PDCCH monitoring occasions of FG-3-1, plus additional  PDCCH monitoring occasion(s) can be any OFDM symbol(s) of a slot for Case 2, and for any two PDCCH monitoring occasions belonging to different spans, where at least one of them is not the monitoring occasions of FG-3-1, in same or different search spaces, there is a minimum time separation of X OFDM symbols (including the cross-slot boundary case) between the start of two spans, where each span is of length up to Y consecutive OFDM symbols of a slot. Spans do not overlap. Every span is contained in a single slot. The same span pattern repeats in every slot. The separation between consecutive spans within and across slots may be unequal but the same (X, Y) limit must be satisfied by all spans.  Every monitoring occasion is fully contained in one span. In order to determine a suitable span pattern, first a bitmap b(l), 0<=l<=13 is generated, where b(l)=1 if symbol l of any slot is part of a monitoring occasion, b(l)=0 otherwise. The first span in the span pattern begins at the smallest l for which b(l)=1. The next span in the span pattern begins at the smallest l not included in the previous span(s) for which b(l)=1. The span duration is max{maximum value of all CORESET durations, minimum value of Y in the UE reported candidate value} except possibly the last span in a slot which can be of shorter duration. A particular PDCCH monitoring configuration meets the UE capability limitation if the span arrangement satisfies the gap separation for at least one (X, Y) in the UE reported candidate value set in every slot, including cross slot boundary.
For the set of monitoring occasions which are within the same span:
•	Processing one unicast DCI scheduling DL and one unicast DCI scheduling UL per scheduled CC across this set of monitoring occasions for FDD
•	Processing one unicast DCI scheduling DL and two unicast DCI scheduling UL per scheduled CC across this set of monitoring occasions for TDD
•	Processing two unicast DCI scheduling DL and one unicast DCI scheduling UL per scheduled CC across this set of monitoring occasions for TDD
The number of different start symbol indices of spans for all PDCCH monitoring occasions per slot, including PDCCH monitoring occasions of FG-3-1, is no more than floor(14/X) (X is minimum among values reported by UE).
The number of different start symbol indices of PDCCH monitoring occasions per slot including PDCCH monitoring occasions of FG-3-1, is no more than 7.
The number of different start symbol indices of PDCCH monitoring occasions per half-slot including PDCCH monitoring occasions of FG-3-1 is no more than 4 in SCell. | _pdcch-MonitoringAnyOccasionsWithSpanGap_

(X, Y):
_set1_ = (7, 3);
_set2_ = (4, 3) and (7, 3);
_set3_ = (2, 2) and (4, 3) and (7, 3).

 |


```
 pdcch-MonitoringAnyOccasionsWithSpanGap SEQUENCE {
        scs-15kHz                               ENUMERATED {set1, set2, set3}                OPTIONAL,
        scs-30kHz                               ENUMERATED {set1, set2, set3}                OPTIONAL,
        scs-60kHz                               ENUMERATED {set1, set2, set3}                OPTIONAL,
        scs-120kHz                              ENUMERATED {set1, set2, set3}                OPTIONAL
}   
```

# 3 多MO的确定方法
A UE can indicate a capability to monitor PDCCH according to one or more of the combinations  = (2, 2), (4, 3), and (7, 3) per SCS configuration of  μ=1 and μ=2. 
A span is a number of consecutive symbols in a slot where the UE is configured to monitor PDCCH. Each PDCCH monitoring occasion is within one span. 
If a UE monitors PDCCH on a cell according to combination (X,Y), the UE supports PDCCH monitoring occasions in any symbol of a slot with minimum time separation of  X symbols between the first symbol of two consecutive spans, including across slots. A span starts at a first symbol where a PDCCH monitoring occasion starts and ends at a last symbol where a PDCCH monitoring occasion ends, where the number of symbols of the span is up to Y. 

# 4 UE盲检能力

```
monitoringCapabilityConfig-r16      ENUMERATED { r15monitoringcapability,r16monitoringcapability }   OPTIONAL,   -- Need M
monitoringCapabilityConfig-v1710    ENUMERATED { r17monitoringcapability }
```
**monitoringCapabilityConfig**
Configures either Rel-15 PDCCH monitoring capability, Rel-16 PDCCH monitoring capability or Rel-17 PDCCH monitoring capability for PDCCH monitoring on a serving cell (see TS 38.213 [13], clause 10.1). Value r15monitoringcapability enables the Rel-15 monitoring capability, and value r16monitoringcapability enables the Rel-16 PDCCH monitoring capability. r17monitoringcapability enables the Rel-17 PDCCH multi-slot monitoring capability. For 480 and 960 kHz SCS, only value r17monitoringcapability is applicable.


|  | slot盲检 | span盲检 |
| --- | --- | --- |
| 最大盲检次数 | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/2329343/1660973195124-def77ab1-8676-4d9f-8041-4ce16b2e1ed7.png#averageHue=%23e8e6e5&clientId=u1ed3f334-b538-4&from=paste&height=157&id=J4miX&originHeight=157&originWidth=718&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12392&status=done&style=none&taskId=uc11e1d2d-4ff5-4602-a84e-0e9588f3c93&title=&width=718) | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/2329343/1660973206158-a712599b-3d65-474b-9671-ed8fb7ba1eff.png#averageHue=%23e8e6e4&clientId=u1ed3f334-b538-4&from=paste&height=164&id=pvtfK&originHeight=164&originWidth=646&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15887&status=done&style=none&taskId=u0e333358-ca53-4f6f-afa6-e0978ceba1a&title=&width=646) |
| 最大盲检CCE数 | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/2329343/1660973239874-ba494060-55d1-4c5c-b283-5d117ad860cc.png#averageHue=%23e9e6e5&clientId=u1ed3f334-b538-4&from=paste&height=152&id=M6MGd&originHeight=152&originWidth=700&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11845&status=done&style=none&taskId=u6c5296e0-7998-4f04-be8a-c75e7849f19&title=&width=700) | ![image.png](https://cdn.nlark.com/yuque/0/2022/png/2329343/1660973256564-f033af95-d893-4bf9-9f91-a554483bf082.png#averageHue=%23eae7e6&clientId=u1ed3f334-b538-4&from=paste&height=161&id=k09dP&originHeight=161&originWidth=669&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15828&status=done&style=none&taskId=uff2cb71d-5dbb-4751-9ead-c07c17233cd&title=&width=669) |


| 11-2 | Rel-16 PDCCH monitoring capability | 1.	Supported combination(s) of (X, Y, m). For each reported combination, the UE supports the limit C on the maximum number of non-overlapped CCEs for channel estimation per PDCCH monitoring span and the limit M on the maximum number of monitored PDCCH candidates per PDCCH monitoring span
2.	Maximum number of DL and UL unicast DCI formats in a span
For the set of monitoring occasions which are within the same span:
-	Processing one unicast DCI scheduling DL and one unicast DCI scheduling UL per scheduled CC across this set of monitoring occasions for FDD
-	Processing one unicast DCI scheduling DL and two unicast DCI scheduling UL per scheduled CC across this set of monitoring occasions for TDD
-	Processing two unicast DCI scheduling DL and one unicast DCI scheduling UL per scheduled CC across this set of monitoring occasions for TDD | _pdcch-Monitoring-r16 _ |
| --- | --- | --- | --- |


**_pdcch-Monitoring-r16_**
Indicates whether the UE supports PDCCH search space monitoring occasions in any symbol of the slot with minimum time separation between two consecutive transmissions of PDCCH with span up to two OFDM symbols for two OFDM symbols or span up to three OFDM symbols for four and seven OFDM symbols. The different value can be reported for PDSCH processing type 1 and PDSCH processing type 2, respectively. For each sub-carrier spacing, the leading / leftmost bit (bit 0) corresponds to the supported value set (X,Y) of (7,3). The next bit (bit 1) corresponds to the supported value set (X,Y) of (4,3). The rightmost bit (bit 2) corresponds to the supported value set (X,Y) of (2,2).

```
-- R1 11-2: Rel-16 PDCCH monitoring capability
pdcch-Monitoring-r16               SEQUENCE {
    pdsch-ProcessingType1-r16          SEQUENCE {
        scs-15kHz-r16                      PDCCH-MonitoringOccasions-r16 OPTIONAL,
        scs-30kHz-r16                      PDCCH-MonitoringOccasions-r16 OPTIONAL
    }                                                                    OPTIONAL,
    pdsch-ProcessingType2-r16      SEQUENCE {
        scs-15kHz-r16                  PDCCH-MonitoringOccasions-r16     OPTIONAL,
        scs-30kHz-r16                  PDCCH-MonitoringOccasions-r16     OPTIONAL
    }                                                                    OPTIONAL
}                                                                        OPTIONAL,

PDCCH-MonitoringOccasions-r16 ::= SEQUENCE {
    period7span3-r16                  ENUMERATED {supported}                 OPTIONAL,
    period4span3-r16                  ENUMERATED {supported}                 OPTIONAL,
    period2span2-r16                  ENUMERATED {supported}                 OPTIONAL
}
```


