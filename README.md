# HubeRSoundX
无需任何高级权限的强力音频处理App<br>
兼容Android 12及以上系统<br>
具体发布时间详见 BiliBili（同名）
## 认识并了解 HubeRSoundX (以下简称HSX)
HSX理论上是全局音频处理，但如下场景可能无效：<br>
1. 通话<br>
2. 带有Offload且低延迟模式的游戏，如部分音游或MOBA类<br>
3. 开启了AAudio的低延迟的音乐播放器或游戏类App<br><br>
**HSX(5.1)内置了独创HxCore曲线算法，同时借助Android DynamicsProcessing API，在可调的10段均衡器基础上同时进行二次音频渲染，能够尽可能地发挥硬件极限<br><br>
不同于Wavelet和Poweramp，HSX将通过多个Eq曲线，大幅度推动扬声器、耳机、音箱的低频动态范围。<br><br>**
HSX支持多频段前级脉冲，能够在音频进入后级均衡器渲染前，进行一次音频 * **预渲染** * ，模拟各个设备的最优频响优化，甚至模拟环绕声、监听频响曲线
## 关于算法 - HxCore
HxCore是在DynamicsProcessing API的基础上进行Eq和频响曲线计算，同时借助多声道逻辑计算每个声道独立的Eq，模拟不同的声音相位等<br><br>
如下是示例（前级预渲染）：
```java
private void initializeEqBands(float[] freq, float[] freq_t, float[] gain_on, float[] gain_on_t) {
    eqBands = new DynamicsProcessing.EqBand[HxControlCore.maxBandCount_Pre];
    eqBands_T = new DynamicsProcessing.EqBand[HxControlCore.maxBandCount_Pre];
    if (freq == null) {
        freq = HxControlCore.frequencies_jbl_flip;
    }
    if (freq_t == null) {
        freq_t = HxControlCore.frequencies_jbl_flip;
    }
    if (gain_on_t == null) {
        gain_on_t = Default;
    }
    for (int i = 0; i < HxControlCore.maxBandCount_Pre; i++) {
        eqBands[i] = new DynamicsProcessing.EqBand(true, freq[i], gain_on[i]);
        eqBands_T[i] = new DynamicsProcessing.EqBand(true, freq_t[i], gain_on_t[i]);
    }
}
private void initializeEqBands(float[] gain_on) {
    initializeEqBands(null, null, gain_on, null);
}
```
```java
/* 频响曲线 */
public static float[] HxCore_Surround_L = {0, 6f, 3, 0, 5, 2, 5, 3, 0, 0, 0};
public static float[] HxCore_Surround_R = {0, 5, 0, 0, 2.5f, 0, 3, 3, 0, 0, 0};
public static float[] JBL_Flip = {0, 0, 10, 7, 1.2f, 0, -2, -2.5f, 0, 0, 0};
public static float[] JBL_PartyBox = {0, 9, 3.6f, 0, 0, 0, 0, 0, 0, 0, 0};
public static float[] Origin_Voice = {4f, 1.5f, -0.44f, 0.7f, 2f, -4.5f, 0.2f, 1.7f, 3f, 0.8f, 0};
public static float[] N12T_Origin = {9, -1, -6.6f, -4, 8.5f, -7.7f, -2.35f, -6, 1, 1, 1};
public static float[] JBL_PS3300 = {1.5f, 5f, 3.7f, 1.9f, -3.6f, 1.5f, 3f, 1.9f, 1.75f, 1f, 0.6f};
public static float[] Default = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};
```
Mbc多频段压缩同样经过HxCore算法进行了传参优化：
```java
/* HXCORE LIMITER */
public static float[] JBL_PartyBox_Mbc = {6, 4, 3, 0, 0, 0, 0, 0, 0, 0, 0};
public static float[] N12T_Origin_Mbc = {1, 8, 7, 7, 10, 0, 10, -6, 6, 0, 4};

private static final float LIMITER_DEFAULT_ATTACK_TIME = 1;
private static final float LIMITER_DEFAULT_RELEASE_TIME = 60;
private static final float LIMITER_DEFAULT_RATIO = 10;
private static final float LIMITER_DEFAULT_THRESHOLD = -2; 
private static float LIMITER_DEFAULT_POST_GAIN = 0;
public static DynamicsProcessing.Limiter HxCore_LIMITER;
private void initializeMbcBands(float[] freq, float[] gain_on) {
    eqBands_Mbc = new DynamicsProcessing.MbcBand[HxControlCore.maxBandCount_Pre];
    if (freq == null) {
        freq = HxControlCore.frequencies_jbl_flip;
    }
    for (int i = 0; i < HxControlCore.maxBandCount_Pre; i++) {
        eqBands_Mbc[i] = new DynamicsProcessing.MbcBand(true, freq[i],
                LIMITER_DEFAULT_ATTACK_TIME,
                LIMITER_DEFAULT_RELEASE_TIME,
                LIMITER_DEFAULT_RATIO,
                LIMITER_DEFAULT_THRESHOLD,
                3.5f,
                2f,
                1.1f,
                2,
                gain_on[i]);
    }
}
private void initializeMbcBands(float[] gain_on) {
    initializeEqBands(null, null, gain_on, null);
}
```
并与DynamicsProcessing.LIMITER协同工作，尽可能防止过高的低频电流声，同时防止音频爆音现象
