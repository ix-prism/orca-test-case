# M-324 · 两圈墙实心沉孔牺牲桥（sacrificiallayer）吞掉整个 ExPolygon 导致牺牲层外墙清零

- **飞书待核实编号**：`M-324`
- **上游 PR**：`PR #13018`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/PerimeterGenerator.cpp:1727, :1770, :1900`
- **实测结论**：**`REPRODUCED`**（耗时 `5435.0 ms`）

---

## 一、Bug 说明

仅 2 圈墙的薄壁沉孔柱启用 `sacrificiallayer` 时，`PerimeterGenerator.cpp:1770` 以完整 `BRIDGE_INFILL_MARGIN (1.0 mm)` 膨胀桥区，随后第 1900 行 `diff_ex(last, unsupported_filtered)` 吞掉整个外轮廓，导致牺牲桥层（`Z = 5.20 mm`）的外墙与内墙全部消失（0 条）。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/thin_counterbore_boss_eb9c56f5837b3f0d.stl`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m324_boss_sacrificial_w2.gcode.3mf`
- `outputs/m324_boss_none_w2.gcode.3mf`

## 三、手动复现步骤

1. 导入 `inputs/` 中的薄壁沉孔柱模型（`outer_r=6.20, bore_r=5.00, thru_r=2.50, bore_depth=5.0`）。
2. 设置 `enable_support = 0`、`wall_loops = 2`，分别以 `counterbore_hole_bridging = sacrificiallayer`（实验组）与 `none`（对照组）切片。
3. 对比牺牲桥层（`Z = 5.20 mm`）及上下相邻层（`Z = 5.00 / 5.40 mm`）的 `Outer wall`、`Inner wall` 与 `Bridge` 轨迹条数和总长。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-324
```

## 五、实测关键数据

```json
{
  "sacrificiallayer_w2": {
    "z_5.00": {
      "outer_wall_moves": 129,
      "outer_wall_len_mm": 70.22,
      "inner_wall_moves": 0,
      "inner_wall_len_mm": 0,
      "bridge_moves": 0,
      "bridge_len_mm": 0
    },
    "z_5.20": {
      "outer_wall_moves": 0,
      "outer_wall_len_mm": 0,
      "inner_wall_moves": 0,
      "inner_wall_len_mm": 0,
      "bridge_moves": 72,
      "bridge_len_mm": 291.56
    },
    "z_5.40": {
      "outer_wall_moves": 129,
      "outer_wall_len_mm": 54.52,
      "inner_wall_moves": 129,
      "inner_wall_len_mm": 54.52,
      "bridge_moves": 0,
      "bridge_len_mm": 0
    }
  },
  "none_w2_ctrl": {
    "z_5.00": {
      "outer_wall_moves": 129,
      "outer_wall_len_mm": 70.22,
      "inner_wall_moves": 0,
      "inner_wall_len_mm": 0,
      "bridge_moves": 0,
      "bridge_len_mm": 0
    },
    "z_5.20": {
      "outer_wall_moves": 65,
      "outer_wall_len_mm": 37.56,
      "inner_wall_moves": 65,
      "inner_wall_len_mm": 35.09,
      "bridge_moves": 105,
      "bridge_len_mm": 156.62
    },
    "z_5.40": {
      "outer_wall_moves": 128,
      "outer_wall_len_mm": 54.52,
      "inner_wall_moves": 128,
      "inner_wall_len_mm": 54.52,
      "bridge_moves": 0,
      "bridge_len_mm": 0
    }
  }
}
```
