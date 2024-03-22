---
aliases:
  - Yahee
tags:
  - "#WPT"
  - UWPT
  - MPET
  - AI
  - kNN
  - Full-Bridge
---
# Abstract

Undersea wireless power transfer(UWPT) 所會遇到的問題:
在海中，TX 與 RX的位置不會固定導致偶和係數會不斷變化

本篇提出了一種新的最大功率效率追踪（MPET）方法，該方法使用kNN來估計系統的耦合係數，並追踪功率（> 85％）通過自是應轉換器控制。
透過模擬及實驗驗證了WPT與MWPT的在水下環境的穩健性。

---

# Introduction

## WPT簡介
WPT被認為可以使用在離散式的海洋系統，因AUV及海洋相關sensor因傳統上的充電or換電池耗時且導致操作範圍受限和服務中斷。另一種解決方案為建造水下對接站但這樣成本會過高，且限制近岸使用。

目前WPT的研究主要集中於**頻率控制**為主(藉由調整操作頻率已達到系統效率最大化)。此外亦有其他方法如:
>1. 阻抗匹配
>2. 中間場重塑
>3.  以超材料實現

以上都是為了提高在空氣中WPT轉換效率而去研究。

## UWPT所遇到問題
若要以WPT系統在海水做為介質進行水下無線電能傳輸時會遇到下列幾項問題:
>1. 高導電性海水對WPT系統的電氣參數有什麼影響？
 2. 海水對其線圈輻射阻抗有什麼影響？
 3. 如果損失高度依賴於頻率，那麼如何選擇一個在高效率下操作的頻率範圍？

此外，因為海流特性使TX與RX的相對位置不斷改變，導致偶和係數跟著改變。需要從發射器（Tx）和接收器（Rx）獲取信息以追踪系統參數的變化，這對於一個無線通訊的水下系統而言是不可避免的。因此，一個**不依賴通訊網絡**的系統，只使用發射或接收端的信息是至關重要的。

## 本文貢獻
本文提出了一種全面的UWPT效率分析和MPET控制，以實時最大化海底電力傳輸效率。本文的主要貢獻有三個方面：
>1. 量化了在海水中的UWPT線圈的數值。分析了線圈阻抗的頻率特性、自感、互感以及海水對WPT線圈輻射與空氣中WPT的影響。
 2. 根據上述分析結果，開發了一種優化線圈設計方法論，以提高電力傳輸效率。
 3. 提出了一種新的MPET控制方法，用於在不通過發射端的無線通訊系統實現最大功率效率。特別是，開發了一種基於k最近鄰居（kNN）的機器學習方法，用於實時估計RX側耦合係數的機器的學習方法，並直接通過直流-直流轉換器控制輸出電壓以追踪最大電力傳輸效率。

---

# UWPT SYSTEM OVERVIEW

![](D:\Obsidian\Data\Picture\MPETofUWPT\CircuitStructure.png)

本文採用SS架構是因為它允許大功率在接收器（Rx）側流動，而且它已被證明比串聯-並聯和並聯-串聯電路架構更有效。在主側是Tx線圈，在次側是Rx線圈，兩者通過耦合係數$k_\frac{p}{s}$磁性耦合。這裡的Lp、Ls、Cp、Cs、Rp和Rs分別代表電感、串聯共振電容以及一、二次側電路的內阻，而一、二次側電壓分別被定義為Vp和Vs。負載電阻是RL。二次側電壓到一次側電壓的比率可以導出為：
$$\frac{V_s}{V_P} = \frac{(\omega L_m)R_L}{(R_L+R_s)(R_p R_s+(\omega_0 L_m)^2)} \tag{1}$$
假設傳輸頻率與UWPT的共振頻率相同，則功率傳輸效率ηp1可以寫成：
$$\eta_{p1} = \frac{(\omega_0 L_m)^2 R_L}{(R_L + R_s)(R_L R_p + R_p R_s + (\omega_0 L_m)^2)} \tag{2}$$
又
$$\omega_0 = \frac{1}{\sqrt{L_p C_p}} = \frac{1}{\sqrt{L_s C_s}} \tag{3}$$
$\omega_0$對應於共振頻率，而$L_m$是互感，直接與線圈之間的距離相關。$L_m$負責傳輸多少功率，它的標準化形式，耦合係數$k_\frac{p}{s}$（如下所定義）用於指示UWPT是否過耦合、臨界耦合或未耦合:
$$k_\frac{p}{s} = \frac{L_m}{\sqrt{L_p L_S}}， 0 \leq k_\frac{p}{s} \leq 1 \tag{4}$$
但由於整個WPT系統在海底環境，會導致耦合程度下降。下一個問題是如何優化線圈設計以提高效率？第三段會說明。

---

# 線圈於水下環境中的分析

## 海水中線圈的電阻

線圈具有以下幾種電阻:
>直流電組($R_{dc}$):取決於導體大小
>交流電阻($R_{ac}$):由於其表皮深度
>輻射電阻($R_{rad}$):描述由於電磁能量的輻射而產生的等效電阻

由於R_dc和R_ac取決於線圈材質。然而，由於海水的導電性，電流會被線圈周圍的磁場誘導。因此，R_rad在海水中與空氣中差別很大。

$$R^{sea}_{rad} = \omega \mu a [\frac{4}{3}(\beta a)^2 - \frac{\pi}{3}(\beta a)^3 +\frac{2 \pi}{15}(\beta a)^5 - ......] \tag{5}$$
$$R^{air}_{rad} = \frac{\pi}{6} \frac{\omega^4 \mu a^4}{c^3} \tag{6}$$
$\omega$為頻率(rad)，$\mu$為介質之導磁率，a為線圈迴路半徑(m)，$\beta = (\mu \omega \frac{\sigma}{2})^{\frac{1}{2}}$，$\sigma$為介質之導電性。

![](D:\Obsidian\Data\Picture\MPETofUWPT\R_rad.png)

由上圖我們可以知道$R_{rad}$會因為介質不同跟著改變，我們可以跟據上圖推導:
>1. 空氣中線圈的R_rad微乎其微或可以忽略不計。
>2. 當頻率大於200 kHz時，R_rad在海水中指數級增加。

這意味著UWPT Tx線圈應該在某個閾值以下運作（例如，200 kHz）。否則，海水開始產生不利影響並導致更多損失。


## 海水中線圈的感抗

當Tx和Rx線圈處於靠近狀態時，流經其中一個裝置的電流會在另一個裝置上感應出電壓。這種能量以感應方式傳輸的能力通過互感來衡量，如下式所示：
$$L_m = \frac{\mu_0 N_p N_s}{4 \pi} \iint \frac{e^{-\gamma |R_s - R_p|}}{|R_s - R_p|}dL_p.dL_s\tag{7}$$
其中$\gamma ≈ \sqrt{j \omega \mu \sigma}$。Np​ 和 Ns​ 是主線圈和次線圈的匝數，圓形積分是在每個線圈的單匝路徑上進行。以上方程式適用於大多數線圈結構，除了當線圈匝數很少且節距與線徑相比非常大的情況。下圖顯示了在海水中UWPT線圈結構的磁通量分佈。

![](D:\Obsidian\Data\Picture\MPETofUWPT\UWPTCoilStructure.png)

觀察上圖可以發現以下幾點:
>1. 隨著其線圈線徑從0.4毫米增加到1毫米，Tx和Rx的自感值及其互感值略微增加
>2. Rx的自感隨著Tx和Rx之間的距離而變化，而Tx的自感保持不變 (參見圖5(a)和(b))。假設Tx和Rx線圈半徑相同，Rx的自感峰值可能會比Tx的高，導致功率傳輸效率受損 (參見圖5(d))。觀察到Rx線圈的變化是由於當線圈從Tx線圈移開時距離迅速變化所致，尤其是在高導電性的鹽水環境中。我們懷疑在這種變化過程中，Rx線圈的行為就像它有一個固態導體核心一樣。然而，可以在圖5(b)中注意到，當距離開始接近其原始值（距離接近6厘米）時，Rx感應的增加相對較小，並且開始返回到其原始值。

因此，為了實現最大功率傳輸，Tx和Rx線圈的參數不能相同。相反，Rx線圈的尺寸應該比Tx線圈的小。

## 線圈架構

在本文中，我們比較了海底環境中螺旋型線圈（參見圖4）和螺線型線圈的特性。設計參數如線圈半徑、線徑、間距和匝數均被假定為相同，以便公平地比較線圈形狀的效應。模擬結果顯示於圖5(e)至(h)。其結果如下：

>1. 隨著距離的改變，螺線型線圈之間的耦合係數遠低於螺旋型線圈。因此，螺旋型線圈在互感較大，因此耦合係數較高方面，顯著優於螺線型線圈。且當線圈之間的距離增加時，螺旋型線圈的效率優於螺線型線圈。因此，螺旋型線圈架構對UWPT應用幾乎是不可行的(見圖5(e)和(g))。
>2. 在螺旋型和螺線型線圈拓撲結構中增加匝數，出人意料地並未改善耦合係數。相反，增加螺旋型線圈的匝數時耦合係數反而下降(見圖5(g)和(h))。

![](D:\Obsidian\Data\Picture\MPETofUWPT\UWPTcoilAnalysis.png)

根據上述特性的分析，我們可以得出以下結論，設計和操作UWPT應考慮以下規則：

>1. 低頻可行性（在低頻下操作）
>2. 差異化標準（避免Tx和Rx完全相同）
>3. 表面接近性（優化線圈拓撲以增強耦合）

---

# 最優化線圈設計

根據第三節提出了一個系統化的方法來進行最優化線圈設計。

>1. 根據應用施加UWPT系統設計限制。確定線圈形狀、繞線的相對位置、導線半徑以及圈數，並在有限元素（FE）軟體中構建3-D模型。建立一個海水環境，並確立相關的麥克斯韋方程式。
>2. 在Tx線圈和Rx線圈之間應用初始距離值。
>3. 使用三維FE磁場求解器計算線圈參數。
>4. 獲得線圈間自感、互感和耦合係數的值。
>5. 根據共振頻率和線圈的自感選擇補償電容器。
>6. 對線圈進行參數掃描，並在不同距離下獲得最優化線圈參數。
>7. 通過測量共振頻率來驗證最優化線圈設計，確認最優化頻率等於或非常接近操作頻率。
>8. 根據線圈的最優化參數，從磁靜態分析計算線圈的磁場。

由於需要非常精確的參數以獲得最佳結果，所以該程序嚴重依賴於由FE電磁求解器進行的瞬態分析，這需要大量計算資源。因此，需要進行高性能運算（HPC）以便在縮短設計週期內解決大規模模擬。例如，對一個有20圈的UWPT線圈在個人電腦上的模擬可能需要超過72小時才能保證找到一個解決方案，而使用8核心的HPC可以將模擬時間減少到少於12小時並實現更快的收斂。

模擬流程圖:
![](D:\Obsidian\Data\Picture\MPETofUWPT\coilOptimizeFlowchart.png)

---

# 最大功率效率追蹤

## 最大功率效率追蹤概述

最大功率效率追踪（MPET）的目的是為了確保在所有可能的功率傳輸效率下達到最高效率。如此可以減少能量損失和保存能源至關重要。本節的目的是開發一種新的最大功率效率追蹤（MPET）方法，以在海洋環境中部署UWPT時維持最高的可能功率傳輸效率。本文MPET與傳統MPPT的主要不同之處在於，MPET不需要Tx和Rx之間的任何無線通信。

事實上，kNNs基於機器學習的方法被用來估計耦合係數，使得MPET在實際應用中，例如在海底環境中無法進行通訊時，變得更實用。首先，重新去看（1）以獲得次級電壓：
$$V_s = \frac{(\omega_0 L_m) R_L}{(R_L + R_s)(R_p R_s + (\omega_0 L_m)^2)}V_p \tag{8}$$

當Tx和Rx之間的距離變化時，Vs​也會產生變化。如下圖所示:
![](D:\Obsidian\Data\Picture\MPETofUWPT\outputVoltage.png)

可以看出，為了在任何特定的$k_\frac{p}{s}$下達到最大功率效率，$V_s$ 應該被調整到一個理想的 $V_{smax}$。因為 $V_{smax}$​ 對應於理想的負載阻抗 $R_L$​，滿足阻抗匹配條件，如公式（9）所示。

$$V_{smax} = \sqrt{\frac{R_s}{R_p}} \frac{\omega k_{\frac{p}{s}}\sqrt{L_p L_s}}{\sqrt{R_p R_s}+\sqrt{(\omega_0 k_\frac{p}{s})^2 L_p L_s}}V_p\tag{9}$$

一種找到$k_\frac{p}{s}$方法是使用頻帶內無線通訊在Tx和Rx之間傳遞信息以實現阻抗匹配。然而，在海洋環境中由於缺少有效的水下通訊而變得不可行。為了解決這個挑戰，我們提出了一種新的MPET方法，只用Tx一側的信息即時強制 $V_s$→$V_{smax}$​，而不依賴無線通訊。

![](D:\Obsidian\Data\Picture\MPETofUWPT\mpetControlBlock.png)

上圖為引入了kNN來僅使用Rx信息（Vs​ 和 Is​）估計$k_\frac{p}{s}$。之後，使用公式（9）計算 $V_{smax}$​，並作為反饋控制追踪的參考值。我們的MPET是一個適應性追踪器，實時更新PI控制器，並使用DC-DC轉換器完成改變輸出電壓到所需電壓的任務。通過調整工作週期比例來完成，這個特性使UWPT系統能更好地適應$k_\frac{p}{s}$的快速變化。

## 耦合係數的估算

By KVL:
$$V_p = R_p I_p + \omega k_{\frac{p}{s}} \sqrt{L_P L_s}$$
$$V_s = \omega k_\frac{p}{s} \sqrt{L_p L_s} I_s + R_s I_s\tag{10}$$
因此，$k_{\frac{p}{s}}$可以估算為
$$k_{\frac{p}{s}} = \frac{V_p \pm \sqrt{V_p^2 - 4R_p I_s (V_s + R_p I_S)}}{2 I_s \omega_0 \sqrt{L_p L_s}}\tag{11}$$
因為耦合係數(k)會受到Tx與Rx相對位置的不確定性影響，因此沒辦法在沒有通信的情況下追蹤Tx與Rx之狀態。為解決這項挑戰，本文提出種新的數據驅動，kNN，用於在不確定的情況下對$k_\frac{s}{p}$進行估算。而$k_\frac{s}{p}$來自次級側的電壓電流學習。

## 直流-直流轉換器的建模

到達穩態時，圖8中的直流-直流轉換器可以用如下的平均狀態空間模型來表示:
$$\begin{align}
0 &= A_{av}X + B_{av}U \\
V_{dc} &= C_{av}X \tag{12}
\end{align}$$
其中，
$$A_{av} = \begin{bmatrix} 0 & -\frac{D}{C} \\ \frac{D}{L} & \frac{R_b}{L} \end{bmatrix} ， B_{av} = \begin{bmatrix} \frac{1}{C} \\ 0 \end{bmatrix} ， C_{av} = \begin{bmatrix} 1 & 0 \end{bmatrix}$$
且，
$$X = \begin{bmatrix} V_{dc} \\ I_L \end{bmatrix} ， U = \begin{bmatrix} I_{dc} \\ V_b \end{bmatrix}$$
此 L = 電感，$R_b$ = 電池電阻，$V_{dc}$ = 直流電壓，C = 濾波電容。電池電壓為$V_{b}$流入電池之電流為$I_L$。然吼可以得到DC-DC Converter之狀態:
$$I_L = \frac{I_{dc}}{D} \tag{13}$$
$$V_{dc} = \frac{V_b D + I_{dc} R_b}{D^2} \tag{14}$$
又
$$D = \frac{V_b \pm \sqrt{V_b^2 + 4 V_{dc} I_{dc} R_b}}{2 V_{dc}} \tag{15}$$
此處$I_{dc}$、$I_{L}$、$V_{dc}$與D是$i_{dc}(t)$、$i_L(t)$、$v_{dc}(t)$和$d(t)$的穩態值。在瞬態狀態下，為了維持MPET討論的性能水準，使用了一個PI控制器來調整輸出電壓 $V_{dc}$和參考電壓 $V_{dcmax}​$之間的差異，使之回歸原始的穩態值。

## 控制器設計

控制目標在於將DC-DC Converter的輸出電壓調整至負載側指定的輸出電壓，在所有運行條件下。直流-直流轉換器的輸出電壓是占空比$\Delta d$的函數。為了調節輸出電壓，需要控制轉換器以調節其占空比。
![](D:\Obsidian\Data\Picture\MPETofUWPT\PI_Controler.png)
我們引入了一種自適應PI控制器（見圖9），該控制器可以實時調整PI參數。這種方法在於它能在MPET控制系統中實現所期望的性能水平，當$\Delta d$值隨時間變化時。比例常數$K_p$和積分常數$K_i$由PI控制器通過調整$\alpha$和$\beta$的值來自適應調整。PI控制器的傳遞函數如下所示：
$$G_{PI}(s) = K_p + \frac{K_I}{s}$$
其中$\alpha$和$\beta$ (在穩態時，其值分別為$K_p$和$K_i$​）根據誤差信號$v_{err}$的值自適應變化，從而驅動穩態誤差趨近於零。結果出來顯示，動態輸出電壓偏差和輸出電壓的穩定時間都減少了。

![](D:\Obsidian\Data\Picture\MPETofUWPT\Algorithm.png)

---

# 模擬結果

MPET的有效性已在一個水下系統上得到驗證，該系統具有150V的主電壓$V_p$​，線圈操作頻率為178kHz，以及其直流-直流轉換器的開關頻率為100kHz。通過使用MATLAB/Simulink和ANSYS Maxwell以及Simplorer進行了三個測試案例。第一個測試案例是檢查在預期UWPT高功率傳輸效率的共振頻率。第二個測試案例是驗證基於kNN的耦合係數估算性能。第三個測試案例是驗證所提出的MPET方法。用於案例研究中的優化UWPT參數列於下表中。

>| Parameters                 | Tx side | Rx side |
|----------------------------|---------|---------|
| Coil radius (cm)           | 6       | 5       |
| Number of turns            | 20      | 18      |
| Wire diameter (mm)         | 0.6     | 1       |
| Self-inductance (µH)       | 47.8    | 20.3    |
| Parasitic resistance (ohms)| 1.3     | 1.3     |
| Capacitor (nF)             | 53.1    | 38.5    |
| Operating frequency (kHz)  | 178     | 178     |
| AC voltage source (V)      | 150     | -       |
| Switching frequency (kHz)  | -       | 100     |
| Load resistance (Ω)        | -       | 50      |

## UWPT共振頻率分析

為了瞭解共振頻率，UWPT系統的負載從10$\Omega$變化到100$\Omega$，步長為10$\Omega$，同時Tx和Rx線圈之間的距離固定在1cm（即，$k_\frac{p}{s}$​固定在0.18）。

![](D:\Obsidian\Data\Picture\MPETofUWPT\Frequency_ana.png)

正如上圖所示，在178kHz存在一個最佳共振頻率，即使負載在10歐姆到100歐姆之間變化，也能持續達到最大效率。因此，測試UWPT系統將在接下來的測試中以178kHz運行。然而，隨著Tx線圈和Rx線圈之間的耦合係數變化，功率傳輸效率會有很大的變化。這意味著在不斷變化的海洋環境中維持高UWPT效率，線上**耦合係數估計**和**MPET**都是不可或缺的。
![](D:\Obsidian\Data\Picture\MPETofUWPT\K_Effi.png)

## 耦合係數估算

基於kNN的耦合係數實時估算。為了驗證目的，Tx線圈和Rx線圈之間的精確$k_{\frac{p}{s}}$​值可以從主次電路參數的模擬數據集中分析獲得。在特定條件下（例如，在頻率為178kHz和負載阻抗為50歐姆時）進行kNN在線估算。估算$k_{\frac{p}{s}}$值與精確值之間的比較展示了所提出方法的準確性和效率（見圖12）。實際上，它在6.123秒的CPU時間內就能得到1000個估算的$k_{\frac{p}{s}}$值，最大誤差大約為5%。可以看出，最大的誤差發生在由於輻射損失導致距離增加時。這種誤差可以通過在更長的時間內採樣更多數據並過濾測量誤差來減少。這證明了基於kNN的方法非常適合實時MPET。
![](D:\Obsidian\Data\Picture\MPETofUWPT\Noise_added.png)

## UWPT 效率最大化

MPET的有效性在一個動態UWPT模型上進行了評估，該模型受到位置變化的影響，評估是使用ANSYS Maxwell和Simplorer以及MATLAB/Simulink來進行。前者用於執行UWPT的瞬態分析，以獲得優化的線圈參數。後者執行模擬，使用Tustin solver每10$\mu s$運行一次以驗證MPET。

在這個測試中，Tx線圈是固定的，而Rx線圈最初放置在1cm處，然後迅速移動到更遠的4、8和10cm處。測量和展示了UWPT系統在有和沒有MPET時的輸出功率和效率（見下圖）。有MPET時，隨著距離的增加，能夠一致追踪到最大85%的功率效率，並實現34瓦的功率傳輸。然而，沒有MPET時，輸出功率和效率隨著距離的增加而急劇下降，從85%降至39%。也可以看出，當線圈之間的距離發生突然（或近似步進）變化時，MPET總能快速恢復最大功率效率，在0.1秒內，這證明了MPET的有效性和穩健性。

![](D:\Obsidian\Data\Picture\MPETofUWPT\MPET_sim.png)

為了進一步驗證MPET的穩健性，在一個雜訊環境中進行了相同的測試，其中輸出電壓反饋信號和PI控制器受到信噪比為6分貝的加性白高斯雜訊的影響。下圖顯示，即使在這樣的雜訊環境下，MPET仍然保持高效和穩健。

---

# 實驗驗證

為了驗證上述分析，一個UWPT原型在圖(15)中實施。試驗設置包括一個電源，Tx和Rx線圈，微控制器和功率電子轉換器，以及一個數位示波器。目標共振頻率設定為178 kHz，本實驗系統中使用的參數設定與模擬中的一致，除了輸入電壓設定為30 V，這是由於硬體限制。

所有的水下測試都在一個裝滿3.63%鹽分海水的20加侖魚缸中進行。因此，這些解決方案合理地模擬了一個海洋環境。根據第三節中的線圈分析，這些線圈是在22 AWG的線上，以平面螺旋盤的形式製造的，並且它們的自感量使用一個複雜的阻抗分析儀在現場測量。如圖(16)，線圈被放置在一個3D打印的盤子上以固定它們。輸入和輸出電纜分別連接到Tx和Rx線圈，並安裝在一個調節線圈距離的測試架上。
![](D:\Obsidian\Data\Picture\MPETofUWPT\experiment_structure.png)
![](D:\Obsidian\Data\Picture\MPETofUWPT\experiment_structure2.png)

On the transmitting side of the UWPT system, a dc source is used to provide a constant voltage to an operational power amplifier (APEX PA 94a) at 178 kHz generated by a function generator. The function generator used here could be replaced by either a stand alone function generator IC, or be created using software on a microcontroller. The dc power supply of the power amplifier is set to its maximum range ±30 V, creating an ac voltage of 30 Vpp across the TX coil.

Meanwhile, on the receiving end of the UWPT system, four SBR10U40CT diodes are used to build a standard full bridge rectifier, which converts the transferred ac voltage signal to dc. A dc–dc buck converter described in Section V subsequently used to step-down the voltage to a desired voltage level. The control unit ATmega328p includes a 10-b analog-to-digital converter, which converts the voltage and current into digitized data used to execute MPET.


圖17展示了UWPT系統的測量結果；當接收線圈（Rx coil）放在距離發射線圈（Tx coil）1、2、3和4cm的地方時。可以看出接收線圈收到的電壓隨著Tx線圈和Rx線圈之間距離的增加而逐漸減小。圖17(a)至(d)對應於功率傳輸效率分別為80%、73%、66%和58%。
![](D:\Obsidian\Data\Picture\MPETofUWPT\Wave_pic.png)

圖18給出了與模擬結果相比的測量功率效率，顯示了模擬與實驗之間緊密的一致性。
![](D:\Obsidian\Data\Picture\MPETofUWPT\Efficiency-dis.png)

圖19(a)通過對示波器的原始數據應用快速傅立葉變換得到了共振頻率。結果顯示，在目標共振頻率下，功率傳輸效率是最大化的。隨著距離從1厘米增加到10厘米（UWPT的目標距離範圍），效率仍然大於80%。然而，當距離增加超過目標範圍時，海水開始對系統產生不良影響，效率迅速降低到50%以下。
![](D:\Obsidian\Data\Picture\MPETofUWPT\FFt.png)

圖19(b)展示了UWPT系統在共振頻率偏離時的敏感性，即使當共振頻率偏移到165 kHz至185 kHz的頻率範圍時，仍能維持高效率的80%是可行的。不過，建議在Tx線圈端使用可變電容器，以將共振頻率調整回其精確值，如文獻[34]中所述。
![](D:\Obsidian\Data\Picture\MPETofUWPT\Range_FFt.png)

在圖20(a)中，當Tx線圈和Rx線圈之間的距離增加而未受控制時，整流後的輸出電壓$V_{dc}$​顯著下降。實際上，如圖20(b)所示，降壓轉換器可以被控制以調節輸出電壓在12伏，儘管Tx線圈和Rx線圈之間的距離發生變化。
![](D:\Obsidian\Data\Picture\MPETofUWPT\MPET_Fin.png)

---

# 結論

本文提供了一種在動態海底環境中無線傳輸能源的深入理解。影響能源傳輸效率的因素被詳細分析。隨後開發了一種系統化的設計方法來優化線圈特性，從而提高整體系統效率。然後開發了一種新穎的MPET方法，通過機器學習實時估計UWPT耦合係數，並在UWPT受到海洋運動影響時有效追踪最大功率效率（> 85%）。廣泛的模擬和實驗測試驗證了分析、設計和MPET方法的有效性。本文中的系統方法為UWPT研究開啟了新的方法。

我們未來的研究將聚焦於多源海洋能源系統的MPET，新的電力電子架構，這些架構允許負載變化，涵蓋更廣範圍以更好地估計MPET，以及MPET的先進控制策略。

