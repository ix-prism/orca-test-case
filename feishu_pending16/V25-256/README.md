# V25-256 · Arachne 开启顶面单墙（only_one_wall_top）误删同层窄壁管内墙

- **飞书待核实编号**：`V25-256`
- **上游 PR**：`PR #16174`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/PerimeterGenerator.cpp:2180, :2210`
- **实测结论**：**`REPRODUCED`**（耗时 `5717.6 ms`）

---

## 一、Bug 说明

按上游 #16174 几何（壁厚 1.186 mm 的 20×30×10 空心方管 + 20×8×5 台阶），Arachne 开启 `only_one_wall_top = 1` 时，先对整层按单墙生成，导致台阶顶面所在层（`Z = 5.00 mm`）上同层延伸的窄壁管内墙几乎全部消失（由 75.05 mm 暴跌至 0.30 mm）。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/narrow_tube_with_step_930c41dfc26a8004.stl`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/v25_256_one_wall_on.gcode.3mf`
- `outputs/v25_256_one_wall_off.gcode.3mf`

## 三、手动复现步骤

1. 导入 `inputs/` 中的窄壁方管+台阶组合模型（管壁厚 1.186 mm，台阶顶面 Z=5.0 mm，管高 10 mm）。
2. 设置 `wall_generator = arachne`、`wall_loops = 2`、`outer_wall_line_width = 0.42`、`inner_wall_line_width = 0.45`、`min_bead_width = 85%`、`precise_outer_wall = 1`。
3. 对比 `only_one_wall_top = 1`（实验组）与 `0`（对照组）在 `Z = 5.00 mm` 纯方管区域内的 `Inner wall` 挤出总长。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case V25-256
```

## 五、实测关键数据

```json
{
  "only_one_wall_top_on": {
    "tube_y_min_filter": 125.96,
    "layers": {
      "z_4.80": {
        "inner_wall_moves": 19,
        "inner_wall_len_mm": 75.0,
        "outer_wall_moves": 10,
        "outer_wall_len_mm": 167.33
      },
      "z_5.00": {
        "inner_wall_moves": 5,
        "inner_wall_len_mm": 0.32,
        "outer_wall_moves": 25,
        "outer_wall_len_mm": 150.56
      },
      "z_5.20": {
        "inner_wall_moves": 19,
        "inner_wall_len_mm": 75.0,
        "outer_wall_moves": 10,
        "outer_wall_len_mm": 152.27
      }
    }
  },
  "only_one_wall_top_off_ctrl": {
    "tube_y_min_filter": 125.96,
    "layers": {
      "z_4.80": {
        "inner_wall_moves": 19,
        "inner_wall_len_mm": 75.0,
        "outer_wall_moves": 10,
        "outer_wall_len_mm": 167.33
      },
      "z_5.00": {
        "inner_wall_moves": 19,
        "inner_wall_len_mm": 75.0,
        "outer_wall_moves": 10,
        "outer_wall_len_mm": 167.33
      },
      "z_5.20": {
        "inner_wall_moves": 19,
        "inner_wall_len_mm": 75.0,
        "outer_wall_moves": 10,
        "outer_wall_len_mm": 152.27
      }
    }
  },
  "z500_inner_wall_retention_ratio": 0.0043
}
```
