# GitBook

### 注意事项 <a href="#zhu-yi-shi-xiang" id="zhu-yi-shi-xiang"></a>

> * 截止日期：_\*\*_
> * 提交邮箱：ruicao@stu.xmu.edu.cn
> * **逾期提交每天会扣除5%的分数，最高扣40%**

### 简介 <a href="#jian-jie" id="jian-jie"></a>

在本实验，我们将实现基本的帧同步（frame synchronization）模块，并将通过Pluto设备进行发送和接收。通过实现该模块，对帧同步有一定的了解，同时也了解如何配置Pluto硬件。

对于任何的MAC协议，参与无线通信的设备都需要进行帧同步，以发现正在传输的帧。实际传输过程中，会让发送机节点发送已知信号（preambles，通常称为“前导码”），而接收机节点将监听该信号。具体来说，前导码是在数据开始时发送前的波形，用于指示数据的开始。前导码波形是预先定义好的，因此不携带任何数据。接收机将侦听前导码，当检测到前导码后，接收机将开始解调数据包的其余部分。

首先，我们使用已经采集好的信号实现帧同步，采用课上提到的多种方式。然后，我们使用Pluto进行发送和接收，发送已知的信号，并对记录的信号执行帧同步。最后，我们将在Pluto上实现一个实时帧同步模块。

### 第一部分：实现帧同步检测函数（60%） <a href="#di-yi-bu-fen-shi-xian-zhen-tong-bu-jian-ce-han-shu-60" id="di-yi-bu-fen-shi-xian-zhen-tong-bu-jian-ce-han-shu-60"></a>

在这部分中，我们将探讨如何检测并同步到前导码。

检测和同步都可以使用相关操作来完成。两个离散信号x和y的相关结果是另一个离散信号，其定义为

( x \* y ) \[ k ] = ∑ n = - ∞ ∞ x \* \[ n ] · y \[ n + k ]

换句话说，在每个索引$k$处，取第一个信号$x$的共轭，将第二个信号$y$偏移$k$个索引，然后将得到的两个信号相加。直观地说，相关性是衡量一个信号相对于另一个信号的相似度。在本课程中，我们会提到两种相关方法：自相关（auto-correlation）和互相关（cross-correlation）。虽然它们都使用相关的方法，但在他们的计算复杂度会有所不同。

此外，我们还提到了另外两种方法：能量检测和双滑动窗口检测。你需要实现这些方法。

#### 问题1.1：使用互相关进行前导码检测（20%） <a href="#wen-ti-11-shi-yong-hu-xiang-guan-jin-hang-qian-dao-ma-jian-ce-20" id="wen-ti-11-shi-yong-hu-xiang-guan-jin-hang-qian-dao-ma-jian-ce-20"></a>

实现基于互相关的前导码检测器：

* 完善下面的detect\_preamble\_cross\_correlation函数，通过下面的测试代码。该函数应接收两个信号，预先定义的preamble，需要做相关的信号signal。若找到前导码，则返回前导码开始位置的索引，若未找到，则返回None。
* 提交代码

```
# import settings  
import math  
import numpy as np  
from matplotlib import pyplot
```

```
# Compare the correlation magnitude against this value to determine whether there is a preamble or not  
def detect_preamble_cross_correlation(preamble, signal):  
    pass
```

```
# This cell will test your implementation of `detect_preamble`  
preamble_length = 100  
signal_length = 1000  
preamble = (np.random.random(preamble_length) + 1j * np.random.random(preamble_length))
signalA = np.random.random(signal_length) + 1j * np.random.random(signal_length)  
signalB = np.random.random(signal_length) + 1j * np.random.random(signal_length)  
preamble_start_idx = 123  
signalB[preamble_start_idx:preamble_start_idx + preamble_length] += preamble  

np.testing.assert_equal(detect_preamble_cross_correlation(preamble, signalA), None)
np.testing.assert_equal(detect_preamble_cross_correlation(preamble, signalB), preamble_start_idx)
```

#### 问题1.2：使用自相关进行前导码检测（20%） <a href="#wen-ti-12-shi-yong-zi-xiang-guan-jin-hang-qian-dao-ma-jian-ce-20" id="wen-ti-12-shi-yong-zi-xiang-guan-jin-hang-qian-dao-ma-jian-ce-20"></a>

实现基于自相关的前导码检测器（公式在课程中的ppt中已定义，可以使用迭代或递归公式来计算c\[n]）：

* 完善下面detect\_preamble\_auto\_correlation函数，通过下面的测试代码。该函数应接收两个信号，需要做自相关的信号signal，信号中重复信号的长度short\_preamble\_len。若找到前导码，则返回前导码开始位置的索引，若未找到，则返回None。
* 提交代码

```
def detect_preamble_auto_correlation(signal, short_preamble_len):  
    pass
```

```
# This cell will test your implementation of `detect_preamble`  
short_preamble_length = 20  
signal_length = 1000  
short_preamble = np.exp(2j * np.pi * np.random.random(short_preamble_length))  
preamble = np.tile(short_preamble, 10)  # 重复十次
noise = np.random.normal(size=signal_length) + 1j * np.random.normal(size=signal_length)  
signalA = 0.1 * noise  
signalB = 0.1 * noise  
preamble_start_idx = 321  
signalB[preamble_start_idx:preamble_start_idx + len(preamble)] += preamble  
np.testing.assert_equal(detect_preamble_auto_correlation(preamble, signalA), None)  
np.testing.assert_equal(detect_preamble_auto_correlation(preamble, signalB) in range(preamble_start_idx-5, preamble_start_idx+5), True)
```

#### 问题1.3：使用能量检测进行前导码检测（10%） <a href="#wen-ti-13-shi-yong-neng-liang-jian-ce-jin-hang-qian-dao-ma-jian-ce-10" id="wen-ti-13-shi-yong-neng-liang-jian-ce-jin-hang-qian-dao-ma-jian-ce-10"></a>

实现基于能量检测的前导码检测器：

* 完善下面的detect\_preamble\_by\_energy函数
* 提交代码

```
def detect_preamble_by_energy(signal):  
    pass
```

#### 问题1.4：使用双滑动窗口的前导码检测（10%） <a href="#wen-ti-14-shi-yong-shuang-hua-dong-chuang-kou-de-qian-dao-ma-jian-ce-10" id="wen-ti-14-shi-yong-shuang-hua-dong-chuang-kou-de-qian-dao-ma-jian-ce-10"></a>

要求

实现基于双滑动窗口的前导码检测器：

* 完善下面的detect\_preamble\_sliding函数
* 提交代码

```
def detect_preamble_by_sliding_window(signal, short_preamble_len):  
    pass
```

### 第二部分：使用记录的信号进行帧同步（20%） <a href="#di-er-bu-fen-shi-yong-ji-lu-de-xin-hao-jin-hang-zhen-tong-bu-20" id="di-er-bu-fen-shi-yong-ji-lu-de-xin-hao-jin-hang-zhen-tong-bu-20"></a>

#### 问题2.1：离线前导码检测（20%） <a href="#wen-ti-21-li-xian-qian-dao-ma-jian-ce-20" id="wen-ti-21-li-xian-qian-dao-ma-jian-ce-20"></a>

有两个已事先记录好的信号：一个强信号和一个弱信号。每个信号文件中均有多个数据包。请增强上述函数以找出数据包开头的索引列表。

帧结构：|——10STS——|—2.25LTS—|———20 OFDM data symbols———|

每个帧的前导码由10个短训练序列（STS，1个STS长度为16）和2.25个长训练序列（LTS，1个LTS长度为64）组成。尝试使用preamble\_lts或preamble\_sts执行上述四种帧同步方法。

要求（提交报告）

* 使用能量检测方法执行同步：绘制能量图，计算SNR，并输出所有数据包起始索引。
* 使用双滑动窗口方法执行同步：绘制滑动计算结果图，并输出所有数据包开始索引。
* 使用互相关方法执行同步：绘制互相关结果，并输出所有数据包开头的索引。
* 使用自相关方法执行同步：绘制自相关结果，并输出所有数据包开头的索引。
* 请根据给定的弱信号和强信号的实验结果，对报告中的四种方法进行分析与总结。

```
preamble_lts = np.load("preamble_lts.npy")  
preamble_sts = np.load("preamble_sts.npy")  
received_signal_weak = np.load("recorded_signal_weak.npy")  
received_signal_strong = np.load("recorded_signal_strong.npy")
```

### 第三部分：使用Pluto的数据进行帧同步（20%） <a href="#di-san-bu-fen-shi-yong-pluto-de-shu-ju-jin-hang-zhen-tong-bu-20" id="di-san-bu-fen-shi-yong-pluto-de-shu-ju-jin-hang-zhen-tong-bu-20"></a>

#### 问题3.1：尝试使用Pluto设备记录信号（10%） <a href="#wen-ti-31-chang-shi-shi-yong-pluto-she-bei-ji-lu-xin-hao-10" id="wen-ti-31-chang-shi-shi-yong-pluto-she-bei-ji-lu-xin-hao-10"></a>

你可以使用以下程序记录信号。你需要将两个Pluto连接到你的电脑（或者分别连接到一台电脑上），使用下列IP: 192.168.2.1和IP: 192.168.3.2（或程序中的其他IP）。

注意：

* 需要实现设置好Pluto硬件的IP参数
* 你可以调整发射机中的tx\_gain参数（或调整收发机的距离），以获得不同SNR的记录信号。

```
# Transmitter parameters configuration  
from pluto_interface import pluto_transmitter
""" Parameters for PlutoSDR device """  
tx_args = "ip:192.168.2.1"  
tx_freq = 915e6  
bandwidth = 1e6  
tx_gain = -60  
sdr_tx = pluto_transmitter(tx_args, tx_freq, bandwidth, tx_gain, verbose=True).pluto  
# Get transmitted signal and transmit  
transmitted_signal = np.load("tx_signal.npy")  
transmitted_signal = transmitted_signal * (2 ** 14)  
sdr_tx.tx_cyclic_buffer = True  
sdr_tx.tx(transmitted_signal) # Cyclic transmit the signal
```

```
# Receiver parameters configuration  
from pluto_interface import pluto_receiver
""" Parameters for PlutoSDR device """  
rx_args = "ip:192.168.3.2"  
rx_freq = 915e6  
bandwidth = 1e6  
rx_gain = 0  
rx_buffer_size = 1e4  
gain_control_mode = "fast_attack"  
sdr_rx = pluto_receiver(rx_args, rx_freq, bandwidth, rx_gain, rx_buffer_size, gain_control_mode, verbose=True).pluto  
# Receive the signal and record  
received_signal = sdr_rx.rx() # Record a buffer size signal one time  
np.save("recorded_signal.npy", received_signal)
```

#### 问题3.2：使用记录的信号重复第二部分实验 (10%) <a href="#wen-ti-32-shi-yong-ji-lu-de-xin-hao-zhong-fu-di-er-bu-fen-shi-yan-10" id="wen-ti-32-shi-yong-ji-lu-de-xin-hao-zhong-fu-di-er-bu-fen-shi-yan-10"></a>

要求

* 截图：显示已正确记录信号的命令行输出
* 报告：与问题2.1相同

### 第四部分：在线信号的帧同步（可选，加分：5%） <a href="#di-si-bu-fen-zai-xian-xin-hao-de-zhen-tong-bu-ke-xuan-jia-fen-5" id="di-si-bu-fen-zai-xian-xin-hao-de-zhen-tong-bu-ke-xuan-jia-fen-5"></a>

你需要快速处理在线信号，并打印数据包的起始索引。你可以再次使用第三部分中的发送机（但需要修改某些部分以显示你发送了多少数据包）。但你需要开发一个实时接收机，能够连续处理输入信号块。

要求

* 报告：如何设计与实现帧同步模块，截图显示实时发送和接收的数据包的数量。
* 提交代码

### results matching ""

*

### No results matching ""
