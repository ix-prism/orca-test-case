# M-307 · 固件回抽（G10/G11）结合圆弧拟合导致 M73 进度/时间块错位

- **飞书待核实编号**：`M-307`
- **上游 PR**：`PR #12888`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/GCode/GCodeProcessor.cpp:3593 (process_G10/G11) & :2748 (m_line_id)`
- **实测结论**：**`REPRODUCED`**（耗时 `5605.2 ms`）

---

## 一、Bug 说明

开启固件回抽（G10/G11）后，GCodeProcessor 内部将 G10/G11 模拟为 G1 并重复递增时间统计行号，导致输出 G-code 中 M73 进度与实际行号提前错位。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/m307_two_cylinders.3mf`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m307_fw_retract_off.gcode.3mf`
- `outputs/m307_fw_retract_on.gcode.3mf`
- `outputs/m307_gui_fw_off.gcode.3mf`
- `outputs/m307_gui_fw_on.gcode.3mf`
- `outputs/m307_gui_preview.png`
- `outputs/m307_gui_fw_on.gcode.3mf`
- `outputs/m307_gui_fw_off.gcode.3mf`
- `outputs/m307_gui_preview.png`

## 三、手动复现步骤

1. 打开 `inputs/m307_two_cylinders.3mf`（双圆柱以保证每层产生大量回抽与圆弧）。
2. 实验组：关闭 `wipe`（设为 0），开启 `use_firmware_retraction = 1` 与 `enable_arc_fitting = 1`；对照组：`use_firmware_retraction = 0`。
3. 切片后对比两组 G-code 中 G10/G11 次数、G2/G3 圆弧数以及 M73 P99/P100 出现位置占全文件行号的比例。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-307
```

## 五、实测关键数据

```json
{
  "fw_on": {
    "g10_count": 233,
    "g11_count": 232,
    "arc_moves": 1522,
    "m73_count": 106,
    "max_pct_before_100": 95,
    "pct_jump_to_100": 5,
    "first_m73_p99_line": 8716,
    "last_extrusion_line": 8700,
    "first_p99_line_ratio": 1.0018,
    "last_m73": {
      "line_no": 8716,
      "layer_idx": 50,
      "p": 100,
      "r": 0,
      "raw": "M73 P100 R0"
    },
    "m73_sample_tail": [
      {
        "line_no": 8161,
        "layer_idx": 50,
        "p": 92,
        "r": 0,
        "raw": "M73 P92 R0"
      },
      {
        "line_no": 8339,
        "layer_idx": 50,
        "p": 93,
        "r": 0,
        "raw": "M73 P93 R0"
      },
      {
        "line_no": 8476,
        "layer_idx": 50,
        "p": 94,
        "r": 0,
        "raw": "M73 P94 R0"
      },
      {
        "line_no": 8596,
        "layer_idx": 50,
        "p": 95,
        "r": 0,
        "raw": "M73 P95 R0"
      },
      {
        "line_no": 8716,
        "layer_idx": 50,
        "p": 100,
        "r": 0,
        "raw": "M73 P100 R0"
      }
    ]
  },
  "fw_off": {
    "g10_count": 0,
    "g11_count": 0,
    "arc_moves": 1522,
    "m73_count": 109,
    "max_pct_before_100": 99,
    "pct_jump_to_100": 1,
    "first_m73_p99_line": 8559,
    "last_extrusion_line": 8703,
    "first_p99_line_ratio": 0.9835,
    "last_m73": {
      "line_no": 8719,
      "layer_idx": 50,
      "p": 100,
      "r": 0,
      "raw": "M73 P100 R0"
    },
    "m73_sample_tail": [
      {
        "line_no": 8232,
        "layer_idx": 50,
        "p": 96,
        "r": 0,
        "raw": "M73 P96 R0"
      },
      {
        "line_no": 8401,
        "layer_idx": 50,
        "p": 97,
        "r": 0,
        "raw": "M73 P97 R0"
      },
      {
        "line_no": 8483,
        "layer_idx": 50,
        "p": 98,
        "r": 0,
        "raw": "M73 P98 R0"
      },
      {
        "line_no": 8559,
        "layer_idx": 50,
        "p": 99,
        "r": 0,
        "raw": "M73 P99 R0"
      },
      {
        "line_no": 8719,
        "layer_idx": 50,
        "p": 100,
        "r": 0,
        "raw": "M73 P100 R0"
      }
    ]
  },
  "ratio_shift": -0.0183,
  "gui_crosscheck": {
    "lane": "desktop-gui-bridge",
    "slice_result_valid_on": true,
    "fw_on": {
      "g10_count": 239,
      "g11_count": 238,
      "arc_moves": 1540,
      "m73_count": 107,
      "max_pct_before_100": 95,
      "pct_jump_to_100": 5,
      "first_m73_p99_line": 9617,
      "last_extrusion_line": 9601,
      "first_p99_line_ratio": 1.0017,
      "last_m73": {
        "line_no": 9617,
        "layer_idx": 50,
        "p": 100,
        "r": 0,
        "raw": "M73 P100 R0"
      },
      "m73_sample_tail": [
        {
          "line_no": 9046,
          "layer_idx": 50,
          "p": 92,
          "r": 0,
          "raw": "M73 P92 R0"
        },
        {
          "line_no": 9217,
          "layer_idx": 50,
          "p": 93,
          "r": 0,
          "raw": "M73 P93 R0"
        },
        {
          "line_no": 9365,
          "layer_idx": 50,
          "p": 94,
          "r": 0,
          "raw": "M73 P94 R0"
        },
        {
          "line_no": 9478,
          "layer_idx": 50,
          "p": 95,
          "r": 0,
          "raw": "M73 P95 R0"
        },
        {
          "line_no": 9617,
          "layer_idx": 50,
          "p": 100,
          "r": 0,
          "raw": "M73 P100 R0"
        }
      ]
    },
    "fw_off": {
      "g10_count": 0,
      "g11_count": 0,
      "arc_moves": 1540,
      "m73_count": 110,
      "max_pct_before_100": 99,
      "pct_jump_to_100": 1,
      "first_m73_p99_line": 9438,
      "last_extrusion_line": 9604,
      "first_p99_line_ratio": 0.9827,
      "last_m73": {
        "line_no": 9620,
        "layer_idx": 50,
        "p": 100,
        "r": 0,
        "raw": "M73 P100 R0"
      },
      "m73_sample_tail": [
        {
          "line_no": 9080,
          "layer_idx": 50,
          "p": 96,
          "r": 0,
          "raw": "M73 P96 R0"
        },
        {
          "line_no": 9242,
          "layer_idx": 50,
          "p": 97,
          "r": 0,
          "raw": "M73 P97 R0"
        },
        {
          "line_no": 9356,
          "layer_idx": 50,
          "p": 98,
          "r": 0,
          "raw": "M73 P98 R0"
        },
        {
          "line_no": 9438,
          "layer_idx": 50,
          "p": 99,
          "r": 0,
          "raw": "M73 P99 R0"
        },
        {
          "line_no": 9620,
          "layer_idx": 50,
          "p": 100,
          "r": 0,
          "raw": "M73 P100 R0"
        }
      ]
    },
    "verdict": "REPRODUCED"
  }
}
```
