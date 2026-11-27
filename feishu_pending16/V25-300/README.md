# V25-300 · TPMS-D / TPMS-FK 稀疏填充裁剪边界近重合端点未重新接合

- **飞书待核实编号**：`V25-300`
- **上游 PR**：`PR #17334`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/Fill/FillTpmsD.cpp:136 & FillTpmsFK.cpp:427`
- **实测结论**：**`REPRODUCED`**（耗时 `9035.7 ms`）

---

## 一、Bug 说明

TPMS-D / TPMS-FK 填充曲线穿过圆形等裁剪边界后未重新拼接近重合端点，造成大量多余断口与空驶绕行。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/cylinder_arc_70a6f6a2f201d5a6.stl`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/v25_300_tpmsd.gcode.3mf`
- `outputs/v25_300_tpmsfk.gcode.3mf`
- `outputs/v25_300_gyroid.gcode.3mf`

## 三、手动复现步骤

1. 导入 `inputs/` 中的半径 18mm 圆柱模型（圆形边界充分穿过 TPMS 轮廓）。
2. 设置 `sparse_infill_density = 20%`，分别以 `tpmsd`、`tpmsfk` 与对照组 `gyroid` 切片。
3. 统计每层 `Sparse infill` 独立多段线数、断口空驶长度及距离 <1.5mm 的未缝合近重合端点数。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case V25-300
```

## 五、实测关键数据

```json
{
  "patterns": {
    "tpmsd": {
      "exit_code": 0,
      "fragmentation": {
        "triggered": true,
        "infill_layers": 32,
        "total_polylines": 129,
        "avg_polylines_per_layer": 4.03,
        "total_travel_breaks": 97,
        "intra_infill_jump_mm": 890.45,
        "near_boundary_splits": 0,
        "avg_near_splits_per_layer": 0.0
      }
    },
    "tpmsfk": {
      "exit_code": 0,
      "fragmentation": {
        "triggered": true,
        "infill_layers": 32,
        "total_polylines": 337,
        "avg_polylines_per_layer": 10.53,
        "total_travel_breaks": 305,
        "intra_infill_jump_mm": 2011.65,
        "near_boundary_splits": 10,
        "avg_near_splits_per_layer": 0.31
      }
    },
    "gyroid": {
      "exit_code": 0,
      "fragmentation": {
        "triggered": true,
        "infill_layers": 32,
        "total_polylines": 108,
        "avg_polylines_per_layer": 3.38,
        "total_travel_breaks": 76,
        "intra_infill_jump_mm": 1125.13,
        "near_boundary_splits": 0,
        "avg_near_splits_per_layer": 0.0
      }
    }
  }
}
```
