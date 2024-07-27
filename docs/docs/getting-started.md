# Getting Started

Welcome to the getting started guide. This guide will help you set up and use our project.

## Installation

<iframe 
  src="//player.bilibili.com/player.html?bvid=BV1RZ421E7Ji&autoplay=0" 
  width="360" 
  height="200" 
  scrolling="yes" 
  frameborder="1" 
  style="border: 5px solid #ccc;"
  allowfullscreen="true">
</iframe>

<!--To install the project, run the following command:
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="360" height="86" src="https://music.163.com/outchain/player?type=2&id=1312528250&auto=0&height=66"></iframe>-->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>音频播放器</title>
    <style>
    .responsive-iframe {
        position: relative;
        overflow: hidden;
        width: 360px;
        height: 86px; /* 设置一个固定高度 */
    }

    .responsive-iframe iframe {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        border: 0;
    }
    </style>
</head>
<body>
    <div class="responsive-iframe">
        <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" src="//music.163.com/outchain/player?type=2&id=1312528250&auto=0&height=66"></iframe>
    </div>
    <p>如果无法播放，请<a href="https://music.163.com/#/song?id=1312528250" target="_blank">点击这里</a>。</p>
</body>
</html>


```bash
pip install myproject

#### `docs/docs/api.md`

```markdown
# API Reference

This section provides a detailed API reference for our project.

## Modules

### `module1`

#### `function1(param1, param2)`

Description of `function1`.

#### `function2(param1, param2)`

Description of `function2`.

### `module2`

#### `function3(param1, param2)`

Description of `function3`.

#### `function4(param1, param2)`

Description of `function4`.
