---
title: 树莓派Google Assistant 使用USB声卡（C-Media）的配置
date: 2018-12-24 18:54:24
tags: 树莓派 USB声卡 C-Media  GoogleAssistant
---

只适用于C-Media的USB声卡，淘宝十几块钱的那种。
配置文件：~/.asoundrc

```
pcm.!default {
  type asym
  capture.pcm "mic"
  playback.pcm "speaker"
}
pcm.mic {
  type plug
  slave {
    pcm "hw:1,0"
  }
}
pcm.speaker {
  type plug
  slave {
    pcm "hw:1,0"
  }
}
```

GA启动命令：   
hotword 是监听热词‘Ok Google’
pushtotalk 是按键，命令行下是回车，没啥用  
```bash
googlesamples-assistant-hotword --device_model_id myraspberryhome
```
