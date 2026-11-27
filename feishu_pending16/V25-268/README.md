# V25-268 · Hilbert / Octagram 稀疏填充上内部桥接端点锚定偏移

- **飞书待核实编号**：`V25-268`
- **上游 PR**：`PR #15206`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/Fill/Fill.cpp:1401, :1434 & PrintObject.cpp:2940`
- **实测结论**：**`REPRODUCED`**（耗时 `7491.8 ms`）

---

## 一、Bug 说明

Hilbert 与 Octagram 填充重生成辅助锚线时未传入稀疏挤出角色且未共享区域原点，导致顶层内部桥接端点偏离下层真实稀疏填充线。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/box_157910e7659edf5c.stl`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/v25_268_hilbertcurve.gcode.3mf`
- `outputs/v25_268_octagramspiral.gcode.3mf`
- `outputs/v25_268_grid.gcode.3mf`

## 三、手动复现步骤

1. 导入 `inputs/` 中的 36×36×10 mm 实心方块。
2. 设置 `sparse_infill_density = 15%`、`top_shell_layers = 3`，分别以 `hilbertcurve`、`octagramspiral` 与对照组 `grid` 切片。
3. 提取首层 `Internal bridge` 端点并计算其到下一层 `Sparse infill` 线段的最近距离及悬空（>0.6mm）比例。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case V25-268
```

## 五、实测关键数据

```json
{
  "patterns": {
    "hilbertcurve": {
      "exit_code": 0,
      "anchor_check": {
        "triggered": true,
        "bridge_layer": 46,
        "bridge_moves": 151,
        "underlying_infill_moves": 155,
        "endpoints_checked": 302,
        "median_dist_mm": 0.7814,
        "max_dist_mm": 3.9449,
        "unanchored_gt_0_6mm": 173,
        "unanchored_ratio": 0.5728
      }
    },
    "octagramspiral": {
      "exit_code": 0,
      "anchor_check": {
        "triggered": true,
        "bridge_layer": 46,
        "bridge_moves": 176,
        "underlying_infill_moves": 108,
        "endpoints_checked": 352,
        "median_dist_mm": 0.355,
        "max_dist_mm": 1.365,
        "unanchored_gt_0_6mm": 97,
        "unanchored_ratio": 0.2756
      }
    },
    "grid": {
      "exit_code": 0,
      "anchor_check": {
        "triggered": true,
        "bridge_layer": 46,
        "bridge_moves": 216,
        "underlying_infill_moves": 37,
        "endpoints_checked": 432,
        "median_dist_mm": 0.4135,
        "max_dist_mm": 2.438,
        "unanchored_gt_0_6mm": 174,
        "unanchored_ratio": 0.4028
      }
    }
  }
}
```
