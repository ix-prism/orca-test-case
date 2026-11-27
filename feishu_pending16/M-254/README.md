# M-254 · 精确 Z 高度与擦料塔同时启用时的告警提示与校验入口

- **飞书待核实编号**：`M-254`
- **上游 PR**：`PR #12821`
- **执行通道**：`desktop-gui-bridge`
- **源码定位**：`src/slic3r/GUI/Tab.cpp:1889 (UI 提示框) vs Print::validate`
- **实测结论**：**`REPRODUCED`**（耗时 `12393.2 ms`）

---

## 一、Bug 说明

同时启用 precise_z_height 与 enable_prime_tower 时，当前仅在设置界面提示擦料塔尺寸可能增加，未在 Print::validate 中给出切片可能失败的校验警告。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/multi_cube_grid_89be866522f8a6b5.stl`
- `inputs/multi_cube_grid_89be866522f8a6b5_part1.stl`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m254_precise_z_tower.gcode.3mf`
- `outputs/m254_gui.png`

## 三、手动复现步骤

1. 将 `inputs/` 中的多挤出机测试方块拖入 Snapmaker Orca，分配不同挤出机（如 1 号与 2 号）。
2. 开启 `enable_prime_tower = 1`（擦料塔）并开启 `precise_z_height = 1`（精确 Z 高度）。
3. 观察是否弹出设置提示框、点击切片并查看通知栏与切片警告内容。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-254
```

## 五、实测关键数据

```json
{
  "slice_result_valid": true,
  "has_wipe_tower": true,
  "slice_warnings": [
    {
      "error_code": "1000C001",
      "level": 1,
      "msg": "bed_temperature_too_high_than_filament",
      "params": [
        "0",
        "1"
      ]
    }
  ],
  "active_notifications": [
    "",
    "Slice ok.",
    "Object name: multi_cube_grid_89be866522f8a6b5_part1.stl\nSize: 18 x 18 x 6 mm\nVolume: 1944 mm³\nTriangles: 12\n",
    "{'error_code': '1000C001', 'level': 1, 'msg': 'bed_temperature_too_high_than_filament', 'params': ['0', '1']}"
  ],
  "modals_seen": [],
  "print_validate_warn_present": false
}
```
