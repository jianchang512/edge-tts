## edge-tts-cn with Sec-MS-GEC

这是原始 [edge-tts](https://github.com/rany2/edge-tts) 的fork，用于解决在中国地区因缺少 `Sec-MS-GEC` `Sec-MS-GEC-Version` 参数而返回403错误的问题。

原理是在 communicate.py 文件的  `Communicate.__init__` 中，增加一个get_sec函数，分别从`https://edge-sec.myaitool.top/?key=edge` 和  `http://123.207.46.66:8086/api/getGec` 接口获取这2个参数(以第一个成功的为准)。

原理及讨论见  
[使用python实时从edge浏览器中抓取最新Sec-MS-GEC的一种方法](https://github.com/rany2/edge-tts/issues/299)   
[403 error is back/need to implement Sec-MS-GEC token](https://github.com/rany2/edge-tts/issues/290)

## 安装与更新方法
```
pip install aiohttp certifi requests 
pip install --upgrade  git+https://github.com/jianchang512/edge-tts#egg=edge-tts

```

## communicate.py

```
def get_sec():
    SecMSGEC=''
    SecMSGECVersion=''
    import requests
    try:
        res=requests.get(f"https://edgeapi.pyvideotrans.com/token.json?{time.time()}",headers={"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36","Accept-Encoding": "gzip, deflate, br","Accept-Language": "en-US,en;q=0.9",})
        if res.status_code==200:
            d=res.json()
            SecMSGEC=d.get('Sec-MS-GEC','')
            SecMSGECVersion=d.get('Sec-MS-GEC-Version','')
            return SecMSGEC,SecMSGECVersion
    except:
        pass
    try:
        res=requests.get("http://123.207.46.66:8086/api/getGec",headers={"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36","Accept-Encoding": "gzip, deflate, br","Accept-Language": "en-US,en;q=0.9",})
        if res.status_code==200:
            d=res.json()
            SecMSGEC=d.get('Sec-MS-GEC','')
            SecMSGECVersion=d.get('Sec-MS-GEC-Version','')
            return SecMSGEC,SecMSGECVersion
    except:
        pass
        
    return SecMSGEC,SecMSGECVersion

```

```
# communicate.py __init__.py 修改

...

self.SecMSGEC,self.SecMSGECVersion=get_sec()

...

if self.SecMSGECVersion and self.SecMSGEC:
    global WSS_URL
    WSS_URL+=f'&Sec-MS-GEC={self.SecMSGEC}&Sec-MS-GEC-Version={self.SecMSGECVersion}'

```

`edge-tts --text "你好啊朋友们"  --voice "zh-CN-YunyangNeural"  --write-media hello.mp3`


# edge-tts

`edge-tts` is a Python module that allows you to use Microsoft Edge's online text-to-speech service from within your Python code or using the provided `edge-tts` or `edge-playback` command.

## Installation

To install it, run the following command:

    $ pip install edge-tts

If you only want to use the `edge-tts` and `edge-playback` commands, it would be better to use pipx:

    $ pipx install edge-tts

## Usage

### Basic usage

If you want to use the `edge-tts` command, you can simply run it with the following command:

    $ edge-tts --text "Hello, world!" --write-media hello.mp3 --write-subtitles hello.vtt

If you wish to play it back immediately with subtitles, you could use the `edge-playback` command:

    $ edge-playback --text "Hello, world!"

Note the above requires the installation of the `mpv` command line player.

All `edge-tts` commands work in `edge-playback` as well.

### Changing the voice

If you want to change the language of the speech or more generally, the voice. 

You must first check the available voices with the `--list-voices` option:

    $ edge-tts --list-voices
    Name: Microsoft Server Speech Text to Speech Voice (af-ZA, AdriNeural)
    ShortName: af-ZA-AdriNeural
    Gender: Female
    Locale: af-ZA

    Name: Microsoft Server Speech Text to Speech Voice (am-ET, MekdesNeural)
    ShortName: am-ET-MekdesNeural
    Gender: Female
    Locale: am-ET

    Name: Microsoft Server Speech Text to Speech Voice (ar-EG, SalmaNeural)
    ShortName: ar-EG-SalmaNeural
    Gender: Female
    Locale: ar-EG

    Name: Microsoft Server Speech Text to Speech Voice (ar-SA, ZariyahNeural)
    ShortName: ar-SA-ZariyahNeural
    Gender: Female
    Locale: ar-SA

    ...

    $ edge-tts --voice ar-EG-SalmaNeural --text "مرحبا كيف حالك؟" --write-media hello_in_arabic.mp3 --write-subtitles hello_in_arabic.vtt

### Custom SSML

Support for custom SSML has been removed since 5.0.0 because Microsoft has taken the initiative to prevent it from working. You cannot use custom SSML anymore.

### Changing rate, volume and pitch

It is possible to make minor changes to the generated speech.

    $ edge-tts --rate=-50% --text "Hello, world!" --write-media hello_with_rate_halved.mp3 --write-subtitles hello_with_rate_halved.vtt
    $ edge-tts --volume=-50% --text "Hello, world!" --write-media hello_with_volume_halved.mp3 --write-subtitles hello_with_volume_halved.vtt
    $ edge-tts --pitch=-50Hz --text "Hello, world!" --write-media hello_with_pitch_halved.mp3 --write-subtitles hello_with_pitch_halved.vtt

In addition, it is required to use `--rate=-50%` instead of `--rate -50%` (note the lack of an equal sign) otherwise the `-50%` would be interpreted as just another argument.

### Note on the `edge-playback` command

`edge-playback` is just a wrapper around `edge-tts` that plays back the generated speech. It takes the same arguments as the `edge-tts` option.

## Python module

It is possible to use the `edge-tts` module directly from Python. For a list of example applications:

* https://github.com/souzatharsis/podcastfy/blob/main/podcastfy/tts/providers/edge.py
* https://github.com/rany2/edge-tts/tree/master/examples
* https://github.com/rany2/edge-tts/blob/master/src/edge_tts/util.py
* https://github.com/hasscc/hass-edge-tts/blob/main/custom_components/edge_tts/tts.py
