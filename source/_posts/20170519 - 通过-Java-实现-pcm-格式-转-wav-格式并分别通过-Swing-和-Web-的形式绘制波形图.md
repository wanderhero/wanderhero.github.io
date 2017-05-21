---
title: 使用 Java 实现 PCM 格式 转 WAV 格式并分别通过 Swing 和 Web 的形式绘制波形图  
categories: [Java,Audio,Wave]  
tags: [Java,Audio,Wave,pcm,wav]  
date: 2017-05-19 18:12:10
---
- 使用 Java 实现 pcm 格式 转 wav 格式，具体查看[https://github.com/wanderhero/test-normal-maven/tree/master/src/main/java/com/wander/wave/pcm](https://github.com/wanderhero/test-normal-maven/tree/master/src/main/java/com/wander/wave/pcm)

``` bash
package com.wander.wave.pcm;

import java.io.FileInputStream;
import java.io.FileOutputStream;

public class PcmToWavUtil {

    public static void convert(String src, String target) throws Exception {
        FileInputStream fis = new FileInputStream(src);
        FileOutputStream fos = new FileOutputStream(target);

        // 计算长度
        int pcmSize = fis.available();// 由于音频文件一般不会大于 Int 最大值，所以使用 available

        // 填入参数，比特率等等。这里用的是 16位 单声道 8000 hz
        WavHeader header = new WavHeader();
        // 长度字段 = 内容的大小（PCMSize) + 头部字段的大小(不包括前面4字节的标识符RIFF以及fileLength本身的4字节)
        header.fileLength = pcmSize + (44 - 8);
        header.fmtHdrLeth = 16;
        header.bitsPerSample = 16;
        header.channels = 1;
        header.formatTag = 0x0001;
        header.samplesPerSec = 8000;
        header.blockAlign = (short) (header.channels * header.bitsPerSample / 8);
        header.avgBytesPerSec = header.blockAlign * header.samplesPerSec;
        header.dataHdrLeth = pcmSize;

        byte[] h = header.getHeader();

        assert h.length == 44;// WAV标准，头部应该是44字节
        // write header
        fos.write(h, 0, h.length);
        // write data stream
        byte[] buf = new byte[1024 * 4];
        int size = fis.read(buf);
        while (size != -1) {
            fos.write(buf, 0, size);
            size = fis.read(buf);
        }
        fis.close();
        fos.close();
        System.out.println("Convert OK!");
    }

}
```
<!-- more -->
``` bash
package com.wander.wave.pcm;

import java.io.ByteArrayOutputStream;
import java.io.IOException;

public class WavHeader {

    /**
     * 4 资源交换文件标志（RIFF）
     */
    public final char fileID[] = {'R', 'I', 'F', 'F'};
    /**
     * 4 总字节数
     */
    public int fileLength;
    /**
     * 4 WAV文件标志（WAVE）
     */
    public char wavTag[] = {'W', 'A', 'V', 'E'};
    /**
     * 4 波形格式标志（fmt ），最后一位空格
     */
    public char fmtHdrID[] = {'f', 'm', 't', ' '};
    /**
     * 4 过滤字节（一般为00000010H），若为00000012H则说明数据头携带附加信息
     */
    public int fmtHdrLeth;
    /**
     * 2 格式种类（值为1时，表示数据为线性PCM编码）
     */
    public short formatTag;
    /**
     * 2 通道数，单声道为1，双声道为2
     */
    public short channels;
    /**
     * 4 采样频率
     */
    public int samplesPerSec;
    /**
     * 4 波形数据传输速率（每秒平均字节数）
     */
    public int avgBytesPerSec;
    /**
     * 2 DATA数据块长度，字节
     */
    public short blockAlign;
    /**
     * 2 PCM位宽
     */
    public short bitsPerSample;
    /**
     * 4 数据标志符（data）
     */
    public char dataHdrID[] = {'d', 'a', 't', 'a'};
    /**
     * 4 DATA总数据长度字节
     */
    public int dataHdrLeth;

    public byte[] getHeader() throws IOException {
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        WriteChar(bos, fileID);
        WriteInt(bos, fileLength);
        WriteChar(bos, wavTag);
        WriteChar(bos, fmtHdrID);
        WriteInt(bos, fmtHdrLeth);
        WriteShort(bos, formatTag);
        WriteShort(bos, channels);
        WriteInt(bos, samplesPerSec);
        WriteInt(bos, avgBytesPerSec);
        WriteShort(bos, blockAlign);
        WriteShort(bos, bitsPerSample);
        WriteChar(bos, dataHdrID);
        WriteInt(bos, dataHdrLeth);
        bos.flush();
        byte[] r = bos.toByteArray();
        bos.close();
        return r;
    }

    private void WriteShort(ByteArrayOutputStream bos, int s) throws IOException {
        byte[] mybyte = new byte[2];
        mybyte[1] = (byte) ((s << 16) >> 24);
        mybyte[0] = (byte) ((s << 24) >> 24);
        bos.write(mybyte);
    }

    private void WriteInt(ByteArrayOutputStream bos, int n) throws IOException {
        byte[] buf = new byte[4];
        buf[3] = (byte) (n >> 24);
        buf[2] = (byte) ((n << 8) >> 24);
        buf[1] = (byte) ((n << 16) >> 24);
        buf[0] = (byte) ((n << 24) >> 24);
        bos.write(buf);
    }

    private void WriteChar(ByteArrayOutputStream bos, char[] id) {
        for (int i = 0; i < id.length; i++) {
            char c = id[i];
            bos.write(c);
        }
    }

}
```
- 通过 Swing 的形式绘制波形图，核心代码如下，具体查看[https://github.com/wanderhero/test-normal-maven/tree/master/src/main/java/com/wander/wave/wav](https://github.com/wanderhero/test-normal-maven/tree/master/src/main/java/com/wander/wave/wav)

``` bash
package com.wander.wave.wav;

import com.wander.wave.plot.Plot;
import com.wander.wave.plot.PlotFrame;

public class WavUtil {

    // int 数组 转换到 double数组
    // JavaPlot 只支持double数组的绘制
    private static double[] Integers2Doubles(int[] raw) {
        double[] res = new double[raw.length];
        for (int i = 0; i < res.length; i++) {
            res[i] = raw[i];
        }
        return res;
    }

    // 绘制波形文件
    public static void drawWaveFile(String filename) {
        WaveFileReader reader = new WaveFileReader(filename);

        String[] pamss = new String[]{"-r", "-g", "-b"};
        if (reader.isSuccess()) {
            PlotFrame frame = Plot.figrue(String.format("%s %dHZ %dBit %dCH", filename, reader.getSampleRate(), reader.getBitPerSample(), reader.getNumChannels()));
            frame.setSize(800, 400);
            Plot.hold_on();
            for (int i = 0; i < reader.getNumChannels(); ++i) {
                // 获取i声道数据
                int[] data = reader.getData()[i];
                // 绘图
                Plot.plot(Integers2Doubles(data), pamss[i % pamss.length]);
            }
            Plot.hold_off();
        } else {
            System.err.println(filename + "不是一个正常的wav文件");
        }
    }

}
```
- 通过 Web 的形式绘制波形图，核心代码如下，具体参考[https://github.com/katspaugh/wavesurfer.js](https://github.com/katspaugh/wavesurfer.js)

``` bash
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/wavesurfer.js/1.2.3/wavesurfer.min.js"></script>
</head>
<body>
    <div id="waveform"></div>
    <div style="text-align: center">
        <p class="row"></p>
        <div class="col-xs-1">
            <i class="glyphicon glyphicon-zoom-in"></i>
        </div>

        <div class="col-xs-10">
            <input id="slider" type="range" min="1" max="200" value="1" style="width: 100%"/>
        </div>

        <div class="col-xs-1">
            <i class="glyphicon glyphicon-zoom-out"></i>
        </div>
        <p></p>
    </div>

    <script>
        var wavesurfer = WaveSurfer.create({
            container: '#waveform',
            waveColor: 'red',
            progressColor: 'purple'
        });

        wavesurfer.load('https://ia902606.us.archive.org/35/items/shortpoetry_047_librivox/song_cjrg_teasdale_64kb.mp3');

        var slider = document.querySelector('#slider');

        slider.oninput = function () {
            var zoomLevel = Number(slider.value);
            wavesurfer.zoom(zoomLevel);
        };
    </script>
</body>
</html>
```
***
### 前言  
&emsp;&emsp;平时听歌的时候不会去关注这音频是怎么产生的，最近需要处理 pcm 格式相关的文件，从一脸懵逼到略有所获，还是有点意思的~

### pcm 格式 转 wav 格式
&emsp;&emsp;本来想着有没有什么方法可以一步到位直接把 pcm 的波形绘制出来的，找了半天发现 Adobe Audition 工具是可以直接查看的，却没有可用的代码方式，更多的是如何绘制 wav 波形，那就只能退而求其次，曲线救国了。  
&emsp;&emsp;在这之前，还是先了解下什么是 pcm，什么又是 wav，不然就是瞎子摸象了。   
&emsp;&emsp;根据[百度百科](#参考1)上对 pcm 的解释，大致如下。PCM 即脉冲编码调制(Pulse Code Modulation)，把一个时间连续，取值连续的模拟信号变换成时间离散，取值离散的数字信号后在信道中传输，也就是对模拟信号先抽样，再对样值幅度量化，编码的过程。与模拟信号比，它不易受传送系统的杂波及失真的影响。  
&emsp;&emsp;再看[百度百科](#参考2)对 wav 的解释，WAV 为微软公司开发的一种声音文件格式，它符合 RIFF(Resource Interchange File Format) 文件规范，用于保存 Windows 平台的音频信息资源，被Windows平台及其应用程序所广泛支持，是最接近无损的音乐格式。   
 &emsp;&emsp;简单来说：wav 是一种无损的音频文件格式，pcm 是没有压缩的编码方式。pcm 加上 wav 文件头就可以转为 wav 格式，但 wav 还可以用其它方式编码。  
&emsp;&emsp;那问题也就明确了，网上有多种版本的 pcm 转 wav，有 c++ 的，也有基于 android sdk 的，和 Java 标准库还是有区别的，最后感谢 [xiunai78的专栏 - PCM到WAV的转换(Java)](#参考3)，亲测可用。本文稍微做了修改，由于音频文件一般不会大于 int 最大值，所以使用 available() 方法直接获取 pcm 文件的长度，简单快速。
```
int pcmSize = fis.available();
```
&emsp;&emsp;看了代码以后，再对比了 wav 头文件的组成，对 [**pcm 加上 wav 文件头就可以转为 wav 格式**] 这句话就更有体会了。  
  
大小字节|数据块类型|内容
-------|-------- |---
4      |4字符    |资源交换文件标志（RIFF）
4      |长整数   |从下个地址开始到文件尾的总字节数
4      |4字符    |WAV文件标志（WAVE）
4      |4字符    |波形格式标志（fmt ），最后一位空格。
4      |整数     |过滤字节（一般为00000010H），若为00000012H则说明数据头携带附加信息（见“附加信息”）。
2      |整数     |格式种类（值为1时，表示数据为线性PCM编码）
2      |整数     |通道数，单声道为1，双声道为2
4      |长整数    |采样频率
4      |长整数    |波形数据传输速率（每秒平均字节数）
2      |整数     |DATA数据块长度，字节。
2      |整数     |PCM位宽
4	   |4字符	 |数据标志符（data）
4	   |长整型	 |DATA总数据长度字节
### 通过 Swing 的形式绘制波形图
&emsp;&emsp;拿到了 wav 文件，接下来就是如何绘制了。刚开始想到的是在本地展现，于是就看到了 [sintrb 提供的 WaveAccess 方法](#参考4)，传入文件的绝对路径，就可以方便得绘制出波形。
![](https://raw.githubusercontent.com/sintrb/WaveAccess/master/doc/screenshots/shots_wav_40_16_1_pcm.png)
&emsp;&emsp;这里忠于原著，即用了作者的代码，也盗了作者的图片，本文只做了略微改动，把键盘快捷键的 z 和 x 改成了波形的缩放键。
 ```
@Override
public void keyPressed(KeyEvent e) {
    if (e.getKeyCode() == KeyEvent.VK_Z) {
        panle.zoom(1.0f + 0.1, 1.0f);
        return;
    } else if (e.getKeyCode() == KeyEvent.VK_X) {
        panle.zoom(1.0f - 0.1, 1.0f);
        return;
    }
}
 ```
### 通过 Web 的形式绘制波形图
&emsp;&emsp;Swing 形式只能在本地使用，在远程浏览器上是看不到界面的，没办法，还是得采用 web 的形式来展现。在寻寻觅觅了良久以后，盼到了佳音[wavesurfer.js](#参考5)，接入方便，功能强大，本文只用到了显示和缩放功能
[![](http://wander.u.qiniudn.com/20170519-1812-0.0.png?imageView2/2/h/100/imageslim)](http://wander.u.qiniudn.com/20170519-1812-0.0.png?imageslim) 
&emsp;&emsp;这里需要注意的是 [**wavesurfer.load**] 不支持文件的绝对路径，只能是同域的相对路径和跨域的 URL，原文如下：
You can load files either from the same domain:
```
wavesurfer.load('../audio/song.mp3');
```
Or from another server, if it supports CORS headers. For example:
```
wavesurfer.load('http://ia902606.us.archive.org/35/items/shortpoetry_047_librivox/song_cjrg_teasdale_64kb.mp3');
```
### 感触
&emsp;&emsp;这是一个神奇的时代，一切因为计算机而变得如此多彩而奇妙~~~
### 参考 
- <span id="参考1">[百度百科 - PCM](http://baike.baidu.com/item/PCM/1568054)</span>  
- <span id="参考2">[百度百科 - WAV](http://baike.baidu.com/item/WAV)</span> 
- <span id="参考3">[xiunai78的专栏 - PCM到WAV的转换(Java)](http://blog.csdn.net/xiunai78/article/details/6867331)</span> 
- <span id="参考4">[Github - sintrb/WaveAccess](https://github.com/sintrb/WaveAccess/)</span> 
- <span id="参考5">[wavesurfer.js](https://wavesurfer-js.org/)</span> 