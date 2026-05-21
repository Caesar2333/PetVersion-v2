 1. Spritesheet 物理布局

  spritesheet.webp: 2304 × 2592 px (9列 × 9行, 每格 256×288)

```json
          col 0   col 1   col 2   ...   col 8
         ┌───────┬───────┬───────┬─────┬───────┐
  row 0  │ idle  │ idle  │ idle  │ ... │ idle  │  ← idle 动作，frame 00~08
         │ 00    │ 01    │ 02    │     │ 08    │
         ├───────┼───────┼───────┼─────┼───────┤
  row 1  │ mv-L  │ mv-L  │ mv-L  │ ... │ mv-L  │  ← move-left 动作
         ├───────┼───────┼───────┼─────┼───────┤
  row 2  │ mv-R  │ mv-R  │ mv-R  │ ... │ mv-R  │  ← move-right 动作
         ├───────┼───────┼───────┼─────┼───────┤
   ...   │  ...  │  ...  │  ...  │ ... │  ...  │
         ├───────┼───────┼───────┼─────┼───────┤
  row 8  │special│special│special│ ... │special│  ← special 动作
         └───────┴───────┴───────┴─────┴───────┘
```



```json

  帧坐标公式：
  x = frameIndex × 256
  y = rowIndex × 288
  w = 256, h = 288

  2. pet.json 当前元数据

  {
    "id": "zhengke-youyu",
    "displayName": "zhengke",
    "description": "...",
    "spritesheetPath": "spritesheet.webp",
    "spritesheet": {
      "width": 2304, "height": 2592,
      "columns": 9, "rows": 9,
      "cellWidth": 256, "cellHeight": 288
    },
    "actions": {
      "idle":       { "row": 0, "frameCount": 9 },
      "move-left":  { "row": 1, "frameCount": 9 },
      "move-right": { "row": 2, "frameCount": 9 },
      "greet":      { "row": 3, "frameCount": 9 },
      "working":    { "row": 4, "frameCount": 9 },
      "waiting":    { "row": 5, "frameCount": 9 },
      "success":    { "row": 6, "frameCount": 9 },
      "failed":     { "row": 7, "frameCount": 9 },
      "special":    { "row": 8, "frameCount": 9 }
    }
  }
```



## 播放器的设置速度

* #### 播放器的设置速度可以是一样的。





