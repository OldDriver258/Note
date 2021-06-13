# Markdown 进阶
更多信息可以访问[MPE简介](https://shd101wyy.github.io/markdown-preview-enhanced/#/zh-cn/)
## 自动目录
[TOC]
## 画图
### 流程图
``` mermaid
graph LR
    A-->B;
    B-->C;
C-->A;  
```
### 时序图
```wavedrom
{ signal: [
    {name:'clk', wave:'p..Pp..P'},
    ['Master',
        ['ctrl',
            {name:'wirte', wave:'01.0....'},
            {name:'read', wave:'0...1..0'}
        ],
        {name:'addr', wave:'x3.x4..x', data:'A1 A2'},
        {name:'wdata', wave:'x3.x....', data: 'D1'}
    ],
    ['Slave',
        ['ctrl',
            {name:'ack', wave:'x01x0.1x'}
        ],
        {name:'rdata', wave:'x.....4x', data: 'Q2'}
    ]
]}
```

### 代码块
```python {cmd=true matplotlib=true}
import matplotlib.pyplot as plt
plt.plot([1,2,3, 4])
plt.show() # show figure
```