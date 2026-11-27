# M-309 · 开启独立支撑层高时逐对象挡风盾跟随支撑插层导致层厚漂移

- **飞书待核实编号**：`M-309`
- **上游 PR**：`PR #12937`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/GCode.cpp:6881 (perobject draft shield on shared layer)`
- **实测结论**：**`REPRODUCED`**（耗时 `5308.9 ms`）

---

## 一、Bug 说明

逐层打印 + 逐对象挡风盾（skirt_type=perobject, draft_shield=enabled）+ 独立支撑层高（support_top_z_distance=0.16），仅支撑打印的插层会错误生成薄层挡风盾（如 0.04/0.16 mm）。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/m309_two_overhangs.3mf`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m309_indep_on.gcode.3mf`
- `outputs/m309_indep_off.gcode.3mf`

## 三、手动复现步骤

1. 打开 `inputs/m309_two_overhangs.3mf`（双悬臂模型）。
2. 设置 `layer_height = 0.20`、`enable_support = 1`、`support_type = normal(auto)`、`support_top_z_distance = 0.16`、`skirt_type = perobject`、`draft_shield = enabled`。
3. 对比 `independent_support_layer_height = 1`（实验组）与 `0`（对照组）切片结果中 `;TYPE:Skirt` 的层高与 `;HEIGHT:` 列表。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-309
```

## 五、实测关键数据

```json
{
  "config_mismatches_on": [],
  "indep_on_skirt": {
    "skirt_layers": 126,
    "heights_seen": [
      0.0042,
      0.0105,
      0.0147,
      0.0189,
      0.0253,
      0.0295,
      0.0337,
      0.04,
      0.0442,
      0.0484,
      0.0589,
      0.0632,
      0.0674,
      0.0737,
      0.0779,
      0.0821,
      0.0884,
      0.0926,
      0.0968,
      0.1032,
      0.1074,
      0.1116,
      0.1179,
      0.1221,
      0.1263,
      0.1326,
      0.1368,
      0.1411,
      0.1516,
      0.1558,
      0.16,
      0.1663,
      0.1705,
      0.1747,
      0.1811,
      0.1853,
      0.1895,
      0.1958,
      0.2
    ],
    "anomalies": [
      {
        "layer_idx": 3,
        "z": 0.507,
        "height": 0.1074,
        "expected": 0.2
      },
      {
        "layer_idx": 4,
        "z": 0.6,
        "height": 0.0926,
        "expected": 0.2
      },
      {
        "layer_idx": 6,
        "z": 0.815,
        "height": 0.0147,
        "expected": 0.2
      },
      {
        "layer_idx": 8,
        "z": 1.122,
        "height": 0.1221,
        "expected": 0.2
      },
      {
        "layer_idx": 9,
        "z": 1.2,
        "height": 0.0779,
        "expected": 0.2
      },
      {
        "layer_idx": 11,
        "z": 1.429,
        "height": 0.0295,
        "expected": 0.2
      },
      {
        "layer_idx": 12,
        "z": 1.6,
        "height": 0.1705,
        "expected": 0.2
      },
      {
        "layer_idx": 13,
        "z": 1.737,
        "height": 0.1368,
        "expected": 0.2
      },
      {
        "layer_idx": 14,
        "z": 1.8,
        "height": 0.0632,
        "expected": 0.2
      },
      {
        "layer_idx": 16,
        "z": 2.044,
        "height": 0.0442,
        "expected": 0.2
      },
      {
        "layer_idx": 17,
        "z": 2.2,
        "height": 0.1558,
        "expected": 0.2
      },
      {
        "layer_idx": 18,
        "z": 2.352,
        "height": 0.1516,
        "expected": 0.2
      },
      {
        "layer_idx": 19,
        "z": 2.4,
        "height": 0.0484,
        "expected": 0.2
      },
      {
        "layer_idx": 21,
        "z": 2.659,
        "height": 0.0589,
        "expected": 0.2
      },
      {
        "layer_idx": 22,
        "z": 2.8,
        "height": 0.1411,
        "expected": 0.2
      },
      {
        "layer_idx": 23,
        "z": 2.966,
        "height": 0.1663,
        "expected": 0.2
      },
      {
        "layer_idx": 24,
        "z": 3.0,
        "height": 0.0337,
        "expected": 0.2
      },
      {
        "layer_idx": 26,
        "z": 3.274,
        "height": 0.0737,
        "expected": 0.2
      },
      {
        "layer_idx": 27,
        "z": 3.4,
        "height": 0.1263,
        "expected": 0.2
      },
      {
        "layer_idx": 29,
        "z": 3.6,
        "height": 0.0189,
        "expected": 0.2
      },
      {
        "layer_idx": 31,
        "z": 3.888,
        "height": 0.0884,
        "expected": 0.2
      },
      {
        "layer_idx": 32,
        "z": 4.0,
        "height": 0.1116,
        "expected": 0.2
      },
      {
        "layer_idx": 34,
        "z": 4.2,
        "height": 0.0042,
        "expected": 0.2
      },
      {
        "layer_idx": 36,
        "z": 4.503,
        "height": 0.1032,
        "expected": 0.2
      },
      {
        "layer_idx": 37,
        "z": 4.6,
        "height": 0.0968,
        "expected": 0.2
      },
      {
        "layer_idx": 39,
        "z": 4.811,
        "height": 0.0105,
        "expected": 0.2
      },
      {
        "layer_idx": 41,
        "z": 5.118,
        "height": 0.1179,
        "expected": 0.2
      },
      {
        "layer_idx": 42,
        "z": 5.2,
        "height": 0.0821,
        "expected": 0.2
      },
      {
        "layer_idx": 44,
        "z": 5.425,
        "height": 0.0253,
        "expected": 0.2
      },
      {
        "layer_idx": 45,
        "z": 5.6,
        "height": 0.1747,
        "expected": 0.2
      },
      {
        "layer_idx": 46,
        "z": 5.733,
        "height": 0.1326,
        "expected": 0.2
      },
      {
        "layer_idx": 47,
        "z": 5.8,
        "height": 0.0674,
        "expected": 0.2
      },
      {
        "layer_idx": 49,
        "z": 6.04,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 50,
        "z": 6.2,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 51,
        "z": 6.24,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 52,
        "z": 6.4,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 53,
        "z": 6.44,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 54,
        "z": 6.6,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 55,
        "z": 6.64,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 56,
        "z": 6.8,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 57,
        "z": 6.84,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 58,
        "z": 7.0,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 59,
        "z": 7.04,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 60,
        "z": 7.2,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 61,
        "z": 7.24,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 62,
        "z": 7.4,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 63,
        "z": 7.44,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 64,
        "z": 7.6,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 65,
        "z": 7.64,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 66,
        "z": 7.8,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 67,
        "z": 7.84,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 68,
        "z": 8.0,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 69,
        "z": 8.04,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 70,
        "z": 8.2,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 71,
        "z": 8.24,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 72,
        "z": 8.4,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 73,
        "z": 8.44,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 74,
        "z": 8.6,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 75,
        "z": 8.64,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 76,
        "z": 8.8,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 77,
        "z": 8.84,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 78,
        "z": 9.0,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 79,
        "z": 9.04,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 80,
        "z": 9.2,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 81,
        "z": 9.24,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 82,
        "z": 9.4,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 83,
        "z": 9.44,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 84,
        "z": 9.6,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 85,
        "z": 9.64,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 86,
        "z": 9.8,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 87,
        "z": 9.84,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 88,
        "z": 10.0,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 89,
        "z": 10.04,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 90,
        "z": 10.2,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 91,
        "z": 10.24,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 92,
        "z": 10.4,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 93,
        "z": 10.44,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 94,
        "z": 10.6,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 95,
        "z": 10.64,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 96,
        "z": 10.8,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 97,
        "z": 10.84,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 98,
        "z": 11.0,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 99,
        "z": 11.04,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 100,
        "z": 11.2,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 101,
        "z": 11.24,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 102,
        "z": 11.4,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 103,
        "z": 11.44,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 104,
        "z": 11.6,
        "height": 0.16,
        "expected": 0.2
      },
      {
        "layer_idx": 105,
        "z": 11.64,
        "height": 0.04,
        "expected": 0.2
      },
      {
        "layer_idx": 106,
        "z": 11.8,
        "height": 0.16,
        "expected": 0.2
      }
    ]
  },
  "indep_off_skirt": {
    "skirt_layers": 79,
    "heights_seen": [
      0.2
    ],
    "anomalies": []
  }
}
```
