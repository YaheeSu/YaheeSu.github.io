---
layout: post
title: Buck Converter 實作
author: [蘇冠豪]
category: [Power Electronic]
tags: [Converter]
math: true
---

# Buck Converter 實作

---
## *Buck Converter 簡介*

Buck轉換器（Buck Converter）是一種DC-DC電壓轉換器，能夠將一個較高的直流電壓轉換成較低的直流電壓，同時增加輸出電流的能力。這種轉換器在各種電子裝置中廣泛應用，特別是在需要電池壽命最大化的便攜式設備中。

### *工作原理*

Buck轉換器的基本工作原理包括利用電感、電容、開關（通常是電晶體）和一個二極體來降低輸入電壓。轉換器的核心運作模式分為兩個階段：

1. 導通階段：當開關導通時，電流通過電感，能量儲存在電感中，輸出電壓主要由通過電感的電流和電容決定。
2. 關斷階段：當開關斷開時，電感釋放其儲存的能量到輸出端，通過二極體維持輸出電壓。

通過調節開關的導通與關斷時間比例（稱為占空比），Buck轉換器能夠精確控制輸出電壓的大小。

---

## *名詞解釋*

|                名詞                |                               解釋                               |
| :------------------------------: | :------------------------------------------------------------: |
|     Output voltage tolerance     |           指的是輸出的電壓職可以在依定的範圍內變動而不影響其正常的功能。通常是以百分比做為單位           |
| Output voltage ripple with Noise |                           含雜訊的輸出電壓漣波                           |
|         Line regulation          | 是評估電源供應在不同工作條件下的穩定性和可靠性。較低的輸入線性調整，這意味著在不同輸入電壓下，輸出電壓的變化較小，相反則反之 |
|         Load regulation          |    是評估電源供應在不同負載情況下的穩定性和可靠性。較低的負載線性調整，這意味著在不同負載電流下，輸出電壓的變化較小    |
|         Over/Under shoot         |               信號在瞬間超過預期的穩定值<br><br>信號在瞬間低於預期的穩定值               |
|          Recovery time           |                  元件在受到變化、干擾或失衡狀態後恢復到正常狀態所需的時間                  |

---

## *元件數值設計*

![設計規格](/graph/Buck/specifications.png){: w="360" h="700" .center}

### 前置運算

$$T_s = \frac{1}{f_s} = \frac{1}{100k} = \mu s \tag{1}$$
$$D_H = \frac{V_O}{V_{IMIN}} = \frac{30}{40} = 0.75\tag{2}$$
$$D_L = \frac{V_O}{V_{IMAX}} = \frac{30}{48} = 0.625\tag{3}$$
### 電感值的計算以及選用

$$L = \frac{V_O * T_S}{2I_{OB}} * (1-D_L) = \frac{30*10 \mu }{2*0.4}*(1-0.625) = 140.6\mu \tag{4}$$
$$140.6\mu* 0.05 + 140.6 \mu = 147.63\mu 取150\mu H\tag{5}$$
### 電容值的計算
$$C = \frac{T_s^2*V_o}{8*L*\Delta V_o}(1-D_L) = 1.5625\mu F \tag{6}$$
$$5\%margin -> 1.5625 \mu F *0.05 + 1.5625 \mu F *0.05 = 1.64 \mu F \tag{7}$$
### ESR值運算
$$ESR = \frac{\Delta V_o}{\Delta I_L} = \frac{600m}{2.664} = 225m\Omega \tag{8}$$

---

## *電路模擬*

### 閉迴路式降壓轉換器

![降壓轉換器模擬電路圖](/graph/Buck/Converter.png){: w="360" h="700" .center}

#### 輸出電壓

![Output Voltage](/graph/Buck/PIC1.png){: w="360" h="700" .center}

---

## *Schematic*

![設計的電路圖](/graph/Buck/schematic.png){: w="360" h="700" .center}

---

## *BOM List*

![材料清單](/graph/Buck/Bom.png){: w="360" h="700" .center}

---

## PCB Layout

![電路佈線&鋪銅](/graph/Buck/PCB_Layout.png)}{: w="360" h="700" .center}

---

## *焊接電路*

![焊接完之電路成品](/graph/Buck/Circuit1.jpg){: w="360" h="700" .center}

---

## *測量*

### 電源供應器

![Power Supply](/graph/Buck/Power.jpg){: w="360" h="700" .center}

### 電路接線

![電路](/graph/Buck/Circuit.jpg){: w="360" h="700" .center}

### 電子負載機

![電子負載機](/graph/Buck/Load.jpg){: w="360" h="700" .center}

### 效率分析

$$V_o = 28.67*6 = 172.02 \tag{1}$$
$$V_i = 40*4.83 = 193.2 \tag{2}$$
$$\eta = \frac{V_o}{V_i} = 0.89 = 89\% \tag{3}$$

---

## 波形

1. 空載
   ![抽載0A](/graph/Buck/0A.PNG){: w="360" h="700" .center}
2. 負載1A
   ![抽載1A](/graph/Buck/1A.PNG){: w="360" h="700" .center}
3. 負載2A
   ![抽載2A](/graph/Buck/2A.PNG){: w="360" h="700" .center}
4. 負載3A
   ![抽載3A](/graph/Buck/3A.PNG){: w="360" h="700" .center}
5. 負載4A
   ![抽載4A](/graph/Buck/4A.PNG){: w="360" h="700" .center}
6. 負載5A
   ![抽載5A](/graph/Buck/5A.PNG){: w="360" h="700" .center}
7. 滿載(6A)
   ![抽載6A](/graph/Buck/6A.PNG){: w="360" h="700" .center}

### 波形分析

1. 隨著抽載變大，可以觀察到占空比正漸漸變小
2. C1為$V_{gs}$，C2為$V_{ds}$，可以看到當$V_{gs}$導通時$V_{ds}$截止，可以看到當$V_{ds}$導通時$V_{gs}$截止，與所學相同
3. 頻率約固定在110kHz，與我們設計的100kHz相差不大

---

在這次的實驗過程中，我深刻體驗到了從電路設計到實際製作的全過程，這不僅僅是一個技術學習的過程，更是一次全面的挑戰與自我超越。從最初的設計思路確定，到模擬測試的反覆調整，再到電路圖的繪制、PCB Layout的布局，以及後續的洗板、焊接和除錯等每一步，都充滿了挑戰。

特別是在使用AD16、Psim等專業軟體進行操作時，我不僅學會了如何利用這些工具來優化設計，更重要的是學會了如何在遇到問題時尋找解決方案，並且堅持到底。過程中遇到了許多挫折，比如電路設計上的疏忽、PCB製作過程中的失誤，甚至是焊接時的小錯誤，這些都讓我感到沮喪。

最終，在完成整個項目的那一刻，我感到了前所未有的成就感。這不僅僅是因為一個項目的完成，更是因為在這個過程中，我親手將一個概念通過無數次的努力和修正，變成了一個實際可用的電路。
