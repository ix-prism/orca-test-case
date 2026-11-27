# M-348 · 不支持空气过滤的机型载入开启过滤耗材时仍输出 M106 P3

- **飞书待核实编号**：`M-348`
- **上游 PR**：`PR #13868`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/GCode.cpp:3201, :3493 & GCodeWriter.cpp:953`
- **实测结论**：**`REPRODUCED`**（耗时 `5533.4 ms`）

---

## 一、Bug 说明

打印中与打印结束只检查耗材的 `activate_air_filtration`，未结合机型 `support_air_filtration = 0` 拦截，导致无过滤硬件的机型仍输出 `M106 P3`。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/box_1d6d5c13a937c6db.stl`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m348_exp_no_hw.gcode.3mf`
- `outputs/m348_ctrl_off.gcode.3mf`

## 三、手动复现步骤

1. 导入 `inputs/` 中的标准测试方块。
2. 机型设置 `support_air_filtration = 0`；实验组耗材开启 `activate_air_filtration = 1`、`during_print_exhaust_fan_speed = 70`、`complete_print_exhaust_fan_speed = 90`；对照组 `activate_air_filtration = 0`。
3. 切片后搜索 G-code 中是否出现 `M106 P3` 指令。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-348
```

## 五、实测关键数据

```json
{
  "config_mismatches": [],
  "exp_m106_p3_count": 2,
  "exp_m106_p3_events": [
    {
      "line_no": 142,
      "layer_idx": 0,
      "pwm": 178,
      "raw": "M106 P3 S178"
    },
    {
      "line_no": 4016,
      "layer_idx": 30,
      "pwm": 229,
      "raw": "M106 P3 S229"
    }
  ],
  "ctrl_m106_p3_count": 0
}
```
