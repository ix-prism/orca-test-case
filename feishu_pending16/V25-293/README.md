# V25-293 · 未使用喷头的 max_layer_height 限制导致擦料塔等层高误报

- **飞书待核实编号**：`V25-293`
- **上游 PR**：`PR #17000`
- **执行通道**：`desktop-gui-bridge`
- **源码定位**：`src/libslic3r/Slicing.cpp:109 & src/libslic3r/Slicing.hpp:117`
- **实测结论**：**`REPRODUCED`**（耗时 `17863.8 ms`）

---

## 一、Bug 说明

物体传入 0 起始喷头编号但在查表读取最大/最小层高限制时再次减 1，导致修改未使用的 2 号喷头 `max_layer_height=0.16` 会触发 1/3 号喷头擦料塔红条报错。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/multi_cube_grid_89be866522f8a6b5.stl`
- `inputs/multi_cube_grid_89be866522f8a6b5_part1.stl`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/v25_293_false_error.png`

## 三、手动复现步骤

1. 在 Snapmaker Orca（U1 四头）载入 `inputs/` 中的两个方块，分别指派到 1 号与 3 号喷头，开启擦料塔且层高设为 0.20mm。
2. 实验组：将未使用的 2 号喷头 `max_layer_height` 设为 `0.16`（其余 `0.28`），切片观察红条警告。
3. 对照组：恢复 2 号为 `0.28`，将实际使用的 3 号设为 `0.16`（或未使用的 4 号设为 `0.16`），对比警告差异。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case V25-293
```

## 五、实测关键数据

```json
{
  "experiment_unused_ext2": {
    "tag": "unused_ext2_0.16",
    "max_layer_height": [
      0.28,
      0.16,
      0.28,
      0.28
    ],
    "slice_result_valid": false,
    "has_wipe_tower": false,
    "prime_tower_layer_height_error": true,
    "active_notifications": [
      "",
      "Slice ok.",
      "Object name: multi_cube_grid_89be866522f8a6b5_part1.stl\nSize: 18 x 18 x 6 mm\nVolume: 1944 mm³\nTriangles: 12\n",
      "Error:\nThe prime tower requires that all objects are sliced with the same layer heights."
    ]
  },
  "control_unused_ext4": {
    "tag": "unused_ext4_0.16",
    "max_layer_height": [
      0.28,
      0.28,
      0.28,
      0.16
    ],
    "slice_result_valid": true,
    "has_wipe_tower": true,
    "prime_tower_layer_height_error": false,
    "active_notifications": [
      "",
      "Slice ok.",
      "Object name: multi_cube_grid_89be866522f8a6b5_part1.stl\nSize: 18 x 18 x 6 mm\nVolume: 1944 mm³\nTriangles: 12\n"
    ]
  },
  "control_active_ext3": {
    "tag": "active_ext3_0.16",
    "max_layer_height": [
      0.28,
      0.28,
      0.16,
      0.28
    ],
    "slice_result_valid": true,
    "has_wipe_tower": true,
    "prime_tower_layer_height_error": false,
    "active_notifications": [
      "",
      "Slice ok.",
      "Object name: multi_cube_grid_89be866522f8a6b5_part1.stl\nSize: 18 x 18 x 6 mm\nVolume: 1944 mm³\nTriangles: 12\n"
    ]
  }
}
```
