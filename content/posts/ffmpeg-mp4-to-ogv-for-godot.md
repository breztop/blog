+++
date = '2026-03-08T12:00:00+08:00'
draft = true
title = '如何使用 ffmpeg 把 MP4 转成 Godot 可用的 OGV'
slug = 'ffmpeg-mp4-to-ogv-for-godot'
description = '一篇实用教程：使用 ffmpeg 将 mp4 视频转换为 Godot 可直接使用的 ogv 格式，并说明常见参数与注意事项。'
author = 'breeze'
tags = ['ffmpeg', 'godot', '视频转换', 'ogv']
categories = ['技术教程']
series = []
keywords = ['ffmpeg mp4 转 ogv', 'godot ogv', 'godot 视频格式']
aliases = []
language = 'zh-CN'

[cover]
  image = ''
  alt = ''
  caption = ''
  relative = false

[images]
  - = ""

[sitemap]
  changefreq = 'monthly'
  priority = 0.5

[extra]
  toc = true
  reading_time = true
  comments = false
+++

在 Godot 中播放视频时，OGV 往往是最常见、兼容性也更直接的一种格式。如果你手里已经有一个 MP4 文件，最简单的办法就是用 ffmpeg 重新编码，输出为 Godot 更容易识别和使用的 OGV 文件。

这篇文章直接给出可用命令，并解释几个关键参数，避免转换后出现无法播放、没有声音、体积过大或画面卡顿的问题。

## 为什么要转成 OGV

Godot 对视频播放的支持并不是“什么格式都能直接塞进去”。实际项目里，更常见的做法是准备好 `.ogv` 文件，再通过 `VideoStreamPlayer` 或相关节点进行播放。

把 MP4 转成 OGV，主要有几个好处：

1. 更符合 Godot 的常见使用场景。
2. 可以主动控制视频编码参数，减少兼容性问题。
3. 可以顺手压缩分辨率、帧率和码率，降低游戏包体与运行时压力。

## 先确认已经安装 ffmpeg

在终端里执行下面的命令：

```bash
ffmpeg -version
```

如果能看到版本信息，说明 ffmpeg 已经安装成功。

如果你还没有安装：

```powershell
winget install ffmpeg
```

安装完成后，重新打开终端再执行一次 `ffmpeg -version`。

## 最简单的转换命令

假设当前目录下有一个 `input.mp4`，想把它转成 `output.ogv`：

```bash
ffmpeg -i input.mp4 -c:v libtheora -c:a libvorbis output.ogv
```

这条命令的含义如下：

1. `-i input.mp4` 表示输入文件。
2. `-c:v libtheora` 表示视频编码使用 Theora。
3. `-c:a libvorbis` 表示音频编码使用 Vorbis。
4. `output.ogv` 是输出文件名。

如果你的目标只是“先转出来一个能在 Godot 里试播的视频”，这条命令通常已经够用了。

## 更适合项目使用的推荐命令

如果你希望输出结果更稳定、体积更合理，建议直接使用下面这个版本：

```bash
ffmpeg -i input.mp4 -c:v libtheora -q:v 7 -c:a libvorbis -q:a 5 -r 30 -vf scale=1280:-2 output.ogv
```

几个核心参数说明如下：

1. `-q:v 7`
控制视频质量，数值越高，画质通常越好，体积也越大。一般可以从 `6` 到 `8` 之间试。

2. `-q:a 5`
控制音频质量。大多数游戏内过场或 UI 视频，`4` 到 `6` 已经足够。

3. `-r 30`
把输出帧率固定为 30fps。对于很多游戏场景，这样能明显减少文件体积和解码压力。

4. `-vf scale=1280:-2`
把宽度压到 1280，高度按比例自动计算，并确保结果是偶数，避免某些编码场景下出现尺寸问题。

如果你的原视频本来就很小，比如 UI 动画或像素风过场，也可以把分辨率改成更低：

```bash
ffmpeg -i input.mp4 -c:v libtheora -q:v 6 -c:a libvorbis -q:a 4 -r 24 -vf scale=854:-2 output.ogv
```

## 无音频视频的转换方式

如果你的视频不需要声音，可以直接去掉音轨，这样文件会更小：

```bash
ffmpeg -i input.mp4 -c:v libtheora -q:v 7 -an output.ogv
```

其中 `-an` 的意思就是不写入音频流。

## 批量转换多个 MP4

如果你有多个 MP4 文件要处理，在 Windows PowerShell 里可以这样批量转换：

```powershell
Get-ChildItem *.mp4 | ForEach-Object {
    ffmpeg -i $_.FullName -c:v libtheora -q:v 7 -c:a libvorbis -q:a 5 "$($_.BaseName).ogv"
}
```

这样会把当前目录下所有 MP4 文件逐个转换成同名的 OGV 文件。

## 在 Godot 中如何使用

转换完成后，把 `.ogv` 文件放进项目目录中，然后在 Godot 里：

1. 导入视频资源。
2. 添加 `VideoStreamPlayer` 节点。
3. 把生成的 `.ogv` 文件指定给它的流资源。
4. 在需要时调用播放逻辑。

如果只是做开场动画、结算过场或者简单叙事视频，这套流程通常已经足够。

## 常见问题

### 1. 转出来的文件太大

优先从这几个地方下手：

1. 降低分辨率，比如改成 `scale=854:-2`。
2. 降低帧率，比如从 `30` 改到 `24`。
3. 降低视频质量，比如把 `-q:v 7` 改成 `-q:v 5`。

### 2. Godot 里能导入，但播放不流畅

通常不是“格式不对”，而是视频太重。尤其是高分辨率、高帧率视频，在低配置设备上更容易卡顿。优先压低分辨率和帧率，而不是只盯着编码格式。

### 3. 没有声音

先检查原始 MP4 是否本身带音轨，再确认转换时没有误加 `-an`。如果仍然有问题，可以单独指定音频参数：

```bash
ffmpeg -i input.mp4 -c:v libtheora -q:v 7 -c:a libvorbis -ar 48000 -ac 2 output.ogv
```

这里：

1. `-ar 48000` 表示采样率为 48000Hz。
2. `-ac 2` 表示双声道。

## 一条实用结论

如果你不想折腾太多参数，可以先记住这一条：

```bash
ffmpeg -i input.mp4 -c:v libtheora -q:v 7 -c:a libvorbis -q:a 5 -r 30 -vf scale=1280:-2 output.ogv
```

它适合作为大多数 Godot 项目的起点。等你真正遇到包体、清晰度或性能问题，再继续微调质量、分辨率和帧率即可。

## 总结

用 ffmpeg 把 MP4 转成 Godot 可用的 OGV，核心并不复杂，关键点只有三个：

1. 视频编码使用 `libtheora`。
2. 音频编码使用 `libvorbis`。
3. 根据项目需要控制分辨率、帧率和质量参数。

对于多数场景，先转出一个稳定可播放的 `.ogv`，再针对体积和性能逐步优化，比一开始就追求“最完美参数”更有效率。