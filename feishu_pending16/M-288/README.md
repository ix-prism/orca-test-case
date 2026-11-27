# M-288 · 擦料塔首层冲刷间距 180% 时相邻冲刷段之间产生缝隙

- **飞书待核实编号**：`M-288`
- **上游 PR**：`PR #12491`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/GCode/WipeTower2.cpp:2387 (m_extra_flow) vs :3129 (m_extra_spacing_wipe)`
- **实测结论**：**`REPRODUCED`**（耗时 `5233.7 ms`）

---

## 一、Bug 说明

单喷头多耗材开启塔内冲刷时，擦料塔首层规划深度按 wipe_tower_extra_spacing，而实际绘制冲刷线按 wipe_tower_extra_flow；当 spacing=180%、flow=100% 时，相邻冲刷段间会出现约 0.40 mm 缝隙。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/m288_three_cubes.3mf`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m288_gui_100.gcode.3mf`
- `outputs/m288_gui_180.gcode.3mf`
- `outputs/m288_gui_preview.png`
- `outputs/m288_spacing100.gcode.3mf`
- `outputs/m288_spacing180.gcode.3mf`
- `outputs/m288_gui_180.gcode.3mf`
- `outputs/m288_gui_100.gcode.3mf`
- `outputs/m288_gui_preview.png`

## 三、手动复现步骤

1. 打开 `inputs/m288_three_cubes.3mf`（含 3 个分别指派到 1/2/3 号耗材的立方体）。
2. 在机型中开启 `single_extruder_multi_material = 1` 与 `purge_in_prime_tower = 1`，开启擦料塔及非零冲刷矩阵。
3. 分别设置 `wipe_tower_extra_spacing = 180%`（实验组）与 `100%`（对照组），切片后查看第 1 层擦料塔冲刷线间距。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-288
```

## 五、实测关键数据

```json
{
  "exit_180": 0,
  "exit_100": 0,
  "config_mismatches_180": [],
  "spacing_180": {
    "triggered": true,
    "axis": "Y",
    "layer1_wipe_moves": 70,
    "scanlines": 68,
    "median_spacing_mm": 0.5,
    "max_spacing_mm": 0.7,
    "max_gap_mm": 0.2,
    "spacings_sample": [
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5
    ]
  },
  "spacing_100": {
    "triggered": true,
    "axis": "Y",
    "layer1_wipe_moves": 53,
    "scanlines": 51,
    "median_spacing_mm": 0.5,
    "max_spacing_mm": 0.5,
    "max_gap_mm": 0.0,
    "spacings_sample": [
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5,
      0.5
    ]
  },
  "gui_crosscheck": {
    "lane": "desktop-gui-bridge",
    "has_wipe_tower_180": true,
    "has_wipe_tower_100": true,
    "spacing_180": {
      "triggered": true,
      "axis": "Y",
      "layer1_wipe_moves": 70,
      "scanlines": 68,
      "median_spacing_mm": 0.5,
      "max_spacing_mm": 0.6,
      "max_gap_mm": 0.1,
      "spacings_sample": [
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5
      ]
    },
    "spacing_100": {
      "triggered": true,
      "axis": "Y",
      "layer1_wipe_moves": 52,
      "scanlines": 50,
      "median_spacing_mm": 0.5,
      "max_spacing_mm": 0.5,
      "max_gap_mm": 0.0,
      "spacings_sample": [
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5,
        0.5
      ]
    },
    "verdict": "REPRODUCED"
  }
}
```
