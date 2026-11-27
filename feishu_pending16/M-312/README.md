# M-312 · 独立支撑 + 避免穿越外墙(reduce_crossing_wall=1)在同XY换层处漏发抬Z导致整层错Z挤出

- **飞书待核实编号**：`M-312`
- **上游 PR**：`PR #13327, #13460`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/GCode.cpp:7932-7941 (提前清 m_need_change_layer_lift_z) & :8806, :8856`
- **实测结论**：**`REPRODUCED`**（耗时 `49058.5 ms`）

---

## 一、Bug 说明

开启普通 snug 支撑 + 独立支撑层高（0.16 顶距）+ 避免穿越外墙（`reduce_crossing_wall=1`）且关擦拭（`wipe=0`）、关抬升（`z_hop=0`）时，相邻支撑层起止 XY 严格重合触发 `travel.size()<2` 跳过 `travel_to_xyz`，随后 `GCode.cpp:7936` 提前清空 `m_need_change_layer_lift_z`，导致下一支撑层未发 `G1 Z` 而直接在旧层高错 Z 挤出。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/Stanford_Bunny.3mf`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m312_u1_rcw1_wipe0.gcode.3mf`
- `outputs/m312_u1_rcw0_ctrl.gcode.3mf`

## 三、手动复现步骤

1. 导入 `inputs/Stanford_Bunny.3mf`（上游 PR #13327 复现模型）。
2. 在 Snapmaker U1 配置下设置 `enable_support=1`、`support_type=normal(auto)`、`support_style=snug`、`independent_support_layer_height=1`、`support_top_z_distance=0.16`、`z_hop=0`、`wipe=0`、`retract_when_changing_layer=1`。
3. 对比 `reduce_crossing_wall=1`（实验组）与 `0`（对照组）切片结果中所有支撑挤出段的物理 Z 与计划 Z（`;Z:`）偏差。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-312
```

## 五、实测关键数据

```json
{
  "u1_rcw1_wipe0_exp": {
    "exit_code": 0,
    "support_z_checked": 102160,
    "wrong_z_total_segments": 724,
    "wrong_z_layers": [
      64,
      74,
      547
    ],
    "wrong_z_sample": [
      {
        "layer_idx": 64,
        "role": "Support",
        "actual_z": 6.44,
        "planned_z": 6.56,
        "line_no": 69753
      },
      {
        "layer_idx": 64,
        "role": "Support",
        "actual_z": 6.44,
        "planned_z": 6.56,
        "line_no": 69754
      },
      {
        "layer_idx": 64,
        "role": "Support",
        "actual_z": 6.44,
        "planned_z": 6.56,
        "line_no": 69755
      },
      {
        "layer_idx": 64,
        "role": "Support",
        "actual_z": 6.44,
        "planned_z": 6.56,
        "line_no": 69756
      },
      {
        "layer_idx": 64,
        "role": "Support",
        "actual_z": 6.44,
        "planned_z": 6.56,
        "line_no": 69757
      },
      {
        "layer_idx": 64,
        "role": "Support",
        "actual_z": 6.44,
        "planned_z": 6.56,
        "line_no": 69758
      }
    ],
    "same_xy_layer_transitions": 29
  },
  "u1_rcw0_wipe0_ctrl": {
    "exit_code": 0,
    "support_z_checked": 98179,
    "wrong_z_total_segments": 0,
    "wrong_z_layers": [],
    "wrong_z_sample": [],
    "same_xy_layer_transitions": 23
  }
}
```
