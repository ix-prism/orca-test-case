# M-322 · 沉孔部分牺牲桥（partiallybridge）锚点未限深导致薄壁沉孔外墙被啃食及浮空端点

- **飞书待核实编号**：`M-322`
- **上游 PR**：`PR #13052`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/PerimeterGenerator.cpp:1855-1869`
- **实测结论**：**`REPRODUCED`**（耗时 `8152.3 ms`）

---

## 一、Bug 说明

沉孔部分桥接（`partiallybridge`）直接按 `BRIDGE_INFILL_MARGIN`（1.0 mm）向外膨胀并与 `reference` 求交，缺少锚点限深归一化，导致 2 圈墙薄壁沉孔柱在牺牲层外墙被啃断、内墙清零，且圆形沉孔出现指向通孔边缘的浮空桥接端点。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/thin_counterbore_boss_eb9c56f5837b3f0d.stl`
- `inputs/counterbore_block_3522290db2fa419d.stl`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m322_boss_partial.gcode.3mf`
- `outputs/m322_boss_none.gcode.3mf`
- `outputs/m322_circ_partial.gcode.3mf`

## 三、手动复现步骤

1. 导入 `inputs/` 中的薄壁沉孔柱（`outer_r=6.20, bore_r=5.00, thru_r=2.50, bore_depth=5.0`）和标准圆形沉孔块（`bore_r=6.50, thru_r=3.00`）。
2. 设置 `enable_support = 0`、`counterbore_hole_bridging = partiallybridge`、`wall_loops = 2`（并与 `none` 对照）。
3. 切片后检查牺牲层（`Z = 5.20 mm`）的 `Outer wall` / `Inner wall` 长度损失，以及标准圆形沉孔桥接长线段的径向锚定端点。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-322
```

## 五、实测关键数据

```json
{
  "thin_boss_z520_partiallybridge": {
    "outer_wall_moves": 40,
    "outer_wall_len_mm": 21.65,
    "inner_wall_moves": 0,
    "inner_wall_len_mm": 0,
    "bridge_moves": 43,
    "bridge_len_mm": 141.88
  },
  "thin_boss_z520_none_ctrl": {
    "outer_wall_moves": 65,
    "outer_wall_len_mm": 37.56,
    "inner_wall_moves": 65,
    "inner_wall_len_mm": 35.09,
    "bridge_moves": 105,
    "bridge_len_mm": 156.62
  },
  "circular_counterbore_long_spans": 25,
  "circular_counterbore_floating_long_count": 2,
  "circular_counterbore_floating_sample": [
    {
      "len_mm": 5.48,
      "r0_past_bore_mm": 0.949,
      "r1_past_bore_mm": -3.311,
      "z": 5.2
    },
    {
      "len_mm": 5.29,
      "r0_past_bore_mm": -3.311,
      "r1_past_bore_mm": 0.776,
      "z": 5.2
    }
  ]
}
```
