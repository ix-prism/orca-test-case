# M-364 · 单喷头 BBL 擦拭塔旧换料路径（append_tcr）耗材结束 G-code 缺 layer_z 变量

- **飞书待核实编号**：`M-364`
- **上游 PR**：`PR #14462`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/GCode.cpp:570 (append_tcr) vs :9156 (append_tcr2)`
- **实测结论**：**`REPRODUCED`**（耗时 `4819.5 ms`）

---

## 一、Bug 说明

Snapmaker U1 走 `append_tcr2` 能正确展开 `{layer_z}`；但当 `is_BBL_printer() == true`（如 `printer_model = Bambu Lab X1 Carbon`）且启用擦拭塔时进入 `append_tcr`（`GCode.cpp:570`），展开 `filament_end_gcode` 缺少 `layer_z` 动态上下文，直接抛出 `Not a variable name: {layer_z}` 导致切片失败。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/m364_two_cubes.3mf`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m364_u1_multi_ctrl.gcode.3mf`

## 三、手动复现步骤

1. 打开 `inputs/m364_two_cubes.3mf`（1/2 号双挤出机方块），在 `filament_end_gcode` 中添加 `; DIAG_LAYER_Z={layer_z}`。
2. 实验组：设置 `printer_model = Bambu Lab X1 Carbon` + `single_extruder_multi_material = 1` + `purge_in_prime_tower = 1` + `enable_prime_tower = 1`（命中 `append_tcr`）；对照组：Snapmaker U1 原生多喷头（命中 `append_tcr2`）。
3. 对比两组切片退出码、错误日志（是否报 `filament_end_gcode Parsing error: Not a variable name`）及 `; DIAG_LAYER_Z=` 展开结果。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-364
```

## 五、实测关键数据

```json
{
  "bbl_append_tcr_exit": 4294967196,
  "bbl_append_tcr_layer_z_error": true,
  "bbl_append_tcr_stderr_tail": "Failed to generate G-code for invalid custom G-code.\n\nfilament_end_gcode Parsing error at line 1: Not a variable name\n; DIAG_LAYER_Z={layer_z}\n                ^\n\nPlease check the custom G-code or use the default custom G-code.\n",
  "bbl_append_tcr_diag_count": 0,
  "u1_append_tcr2_ctrl_exit": 0,
  "u1_append_tcr2_ctrl_diag_count": 22,
  "u1_append_tcr2_ctrl_sample": [
    {
      "line_no": 351,
      "layer_idx": 1,
      "planned_z": 0.25,
      "value": "0.25",
      "raw": "; DIAG_LAYER_Z=0.25"
    },
    {
      "line_no": 1951,
      "layer_idx": 2,
      "planned_z": 0.45,
      "value": "0.45",
      "raw": "; DIAG_LAYER_Z=0.45"
    },
    {
      "line_no": 3194,
      "layer_idx": 3,
      "planned_z": 0.65,
      "value": "0.65",
      "raw": "; DIAG_LAYER_Z=0.65"
    }
  ]
}
```
