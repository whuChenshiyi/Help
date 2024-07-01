## AIIMU位移增量观测方程
&emsp;&emsp;**Neural-PDR**：由一段时间窗口内的加速度计和陀螺仪输出，与PDR深度学习神经网络模型，可以得到这一段时间内的位移增量，这段位移增量可以以观测向量的形式以克隆卡尔曼滤波的形式融合到捷联PDR中。位移增量的坐标系与输入的加速度和角速度所在的坐标系决定。如果输入到网络中的加速度和角速度所在的坐标系为当地水平坐标系，则位移增量为当地水平坐标系下的；如果输入到网络中的加速度和角速度所在的坐标系为导航坐标系，则位移增量为导航坐标系下的。

&emsp;&emsp;由于输入的位移增量是当前时刻与之前某一时刻之间位置之差，因此需要用到至少两个时刻下的状态，因此需要对状态量进行扩维，同时还需要将卡尔曼滤波从扩展卡尔曼滤波更改为克隆卡尔曼滤波。扩维后的状态量为

$$
X=
\begin{bmatrix}
P_n & V_n & A & B_g & B_a & M_g & M_n & S & P_n'
\end{bmatrix}
$$

$$
x=
\begin{bmatrix}
\delta{p_n} &\delta{v_n} &\phi &\delta{b_g} &\delta{b_a} &\delta{m_g} &\delta{m_n} &\delta{s} &\delta{p_n'}
\end{bmatrix}
$$

其中，各状态量的解释和维度如下

|State|Explaination|
|:----:|:---:|
|$P_n$|n系下的位置，3×1|
|$V_n$|n系下的速度，3×1|
|$A$|姿态，3×1|
|$b_g$|陀螺零偏，3×1|
|$b_a$|加表零偏，3×1|
|$M_g$|磁力计零偏，3×1|
|$M_n$|当地磁向量，3×1|
|$S$|比例因子，1×1|
|$P_n'$|扩维的位置，3×1|

&emsp;&emsp;由于位移增量为两个位置点的差值，因此物理方程可列为

$$
P-P'=S \cdot pos\_vec
$$

### 1.位移增量 (导航坐标系)
&emsp;&emsp;如果需要得到导航坐标系下的位移增量，首先需要先将加速度和角速度由传感器坐标系投影到导航坐标系，即

$$
C_b^nAcc_b = Acc_n
$$

$$
C_b^nGyro_b = Gyro_n
$$

&emsp;&emsp;将导航坐标系下的加速度和角速度输入到网络模型中，即可得到导航坐标系n系下的位移增量。根据n系下的位移增量$dpos_n$，即可列出观测方程$H(x)$，既有

$$
H(x) = (P_n-P'_n)-S\cdot dpos_n = 0
$$

在观测方程中，对$P_n$，$P'_n$，$S$做扰动，即有

$$
(\widehat{P}_n - \widehat{P}'_n) - \widehat{S} \cdot dpos_n - \delta{z} = 0 + n_p
$$

式中，$\delta{z}$为误差的整体扰动，$n_p$为位移增量的观测噪声。将上式展开，则会得到

$$
\begin{aligned}
\delta{z} &= (\widehat{P}_n - \widehat{P}'_n) - \widehat{S} \cdot dpos_n \\
          &= (P_n + \delta{p_n} - P_n' - \delta{p_n'}) -( S + \delta{s}) \cdot dpos_n \\
          &= (P_n + P_n' - S \cdot dpos_n) + \delta{p_n} + \delta{p_n'} -\delta{s} \cdot dpos_n \\
          &= \delta{p_n} - \delta{p_n'} -\delta{s} \cdot dpos_n
\end{aligned}
$$

观测方程的观测矩阵为

$$
H = 
\begin{bmatrix}
I & 0 & 0 & 0 & 0 & 0 & 0 & -dpos_n & -I
\end{bmatrix}
$$

### 2.位移增量 (当地水平坐标系)
&emsp;&emsp;当地水平坐标系n'与导航坐标系n一样，z轴指向垂向，x轴与y轴也指向地形方向。与导航坐标系不同，导航坐标系x轴y轴分别指向北向和东向，而当地水平坐标系则是只可以指向诸如建筑道路等周围环境特征的方向，其轴系也满足右手定则。相较于导航坐标系少了一个航向信息。

&emsp;&emsp;从载体坐标系b转到导航坐标系n的方向余弦矩阵$C_b^n$可分解为三个矩阵$C_{yaw}$,$C_{pitch}$,$C_{roll}$，相较于当地水平坐标系n'，多了一个航向信息，即有

$$
C_b^{n'}=C_{pitch} \cdot C_{roll}
$$

$$
C_b^n=C_{yaw} \cdot C_{pitch} \cdot C_{roll}
$$

$$
C_b^n = C_{yaw} \cdot C_b^{n'}
$$

$$
C_b^{n'} = C_{yaw}^T \cdot C_b^n
$$


&emsp;&emsp;如果需要得到当地水平坐标系下的位移增量，首先需要先将加速度和角速度由传感器坐标系投影到当地水平坐标系，即

$$
C_b^{n'}Acc_b = Acc_{n'}
$$

$$
C_b^{n'}Gyro_b = Gyro_{n'}
$$

&emsp;&emsp;将当地水平坐标系下的加速度和角速度输入到网络模型中，即可得到当地水平坐标系n'系下的位移增量$dpos_{n'}$

$$
\begin{aligned}
H(x) &= (P_{n'}-P'_{n'})-S\cdot dpos_{n'}        \\ 
     &= C_{yaw}^T(P_n-P_n') - S \cdot dpos_{n'}  \\
     &= 0
\end{aligned}
$$

&emsp;&emsp;由于网络输出的位移增量是当地水平坐标系下的位移增量，不包含绝对航向信息，因此该位移增量并不能直接作用到航向修正上，因此在观测方程中不能对航向$C_{yaw}$进行扰动。在观测方程中，对$P_n$，$P'_n$，$S$做扰动，即有

$$
C_{yaw}^T(\widehat{P}_n - \widehat{P}'_n) - \widehat{S} \cdot dpos_{n'} - \delta{z} = 0 + n_p
$$

式中，$\delta{z}$为误差的整体扰动，$n_p$为位移增量的观测噪声。将上式展开，则会得到

$$
\begin{aligned}
\delta{z} &= C_{yaw}^T(\widehat{P}_n - \widehat{P}'_n) - \widehat{S} \cdot dpos_{n'} \\
          &= C_{yaw}^T(P_n + \delta{p_n} - P_n' - \delta{p_n'}) -( S + \delta{s}) \cdot dpos_{n'} \\
          &= C_{yaw}^T(P_n + P_n') - S \cdot dpos_{n'} + C_{yaw}^T\delta{p_n} + C_{yaw}^T\delta{p_n'} -\delta{s} \cdot dpos_{n'} \\
          &= C_{yaw}^T\delta{p_n} - C_{yaw}^T\delta{p_n'} -\delta{s} \cdot dpos_{n'}
\end{aligned}
$$

观测方程的观测矩阵为

$$
H = 
\begin{bmatrix}
C_{yaw}^T & 0 & 0 & 0 & 0 & 0 & 0 & -dpos_n & -C_{yaw}^T
\end{bmatrix}
$$
