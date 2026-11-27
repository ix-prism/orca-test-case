# M-313 · 矩形床旧 CPU 预览路径对 erCustom 自定义圆弧越界误报

- **飞书待核实编号**：`M-313`
- **上游 PR**：`PR #13179`
- **执行通道**：`desktop-gui-bridge`
- **源码定位**：`src/libslic3r/BuildVolume.cpp:335 & src/slic3r/GUI/GCodeViewer.cpp:2740 (CPU 缺 erCustom 过滤) vs :1300 (GPU 已过滤)`
- **实测结论**：**`REPRODUCED`**（耗时 `60360.6 ms`）

---

## 一、Bug 说明

矩形床使用路径包围盒判越界；GPU 预览（`gcode_preview_gpu_accel=true`）在合并圆弧插值点前过滤 `erCustom`，而旧 CPU 预览路径（`gcode_preview_gpu_accel=false`）在 `GCodeViewer.cpp:2740` 漏判 `move.extrusion_role != erCustom`，会把自定义 G-code 里的 `G2/G3` 圆弧挤出插值点纳入包围盒触发越界误报。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/box_454012915153aa2c.stl`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m313_cpu_custom_arc.gcode.3mf`
- `outputs/m313_cpu_custom_arc_preview.png`

## 三、手动复现步骤

1. 在 Snapmaker Orca 中载入 `inputs/` 中的测试方块并置于床中央 `(135, 135)`。
2. 在 `layer_change_gcode` 中注入归属 `erCustom` 的 `G2` 圆弧挤出测试指令（其圆弧超出床边）以及对照用的 `G1` 直线自定义挤出。
3. 分别在关闭 GPU 预览加速（`gcode_preview_gpu_accel=false`，走 CPU loader）和开启 GPU 加速（`true`）下切片预览，对比 `plate_state.toolpath_outside` 与越界警告。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-313
```

## 五、实测关键数据

```json
{
  "cpu_legacy_custom_arc_exp": {
    "slice_result_valid": true,
    "toolpath_outside": true,
    "active_notifications": [
      "",
      "Slice ok.",
      "Object name: box_454012915153aa2c.stl\nSize: 24 x 24 x 4 mm\nVolume: 2304 mm³\nTriangles: 12\n<Error></Error>",
      "Error:\nA G-code path goes beyond the plate boundaries."
    ],
    "load_toolpaths_logs": []
  },
  "cpu_legacy_custom_linear_ctrl": {
    "slice_result_valid": true,
    "toolpath_outside": false,
    "active_notifications": [
      "",
      "Slice ok.",
      "Object name: box_454012915153aa2c.stl\nSize: 24 x 24 x 4 mm\nVolume: 2304 mm³\nTriangles: 12\n<Error></Error>"
    ],
    "load_toolpaths_logs": [
      "[info]\t2026-11-27 11:56:48.211854[Thread 0x0000f67c]:Slic3r::GUI::GCodeViewer::load_toolpaths,m_paths_bounding_box {-32.0238, -7.02587}-{146.79, 146.79}",
      "[info]\t2026-11-27 11:56:48.211854[Thread 0x0000f67c]:Slic3r::GUI::GCodeViewer::load_toolpaths,m_contained_in_bed 0",
      "[info]\t2026-11-27 11:57:04.762728[Thread 0x0000f67c]:Slic3r::GUI::GCodeViewer::load_toolpaths,m_paths_bounding_box {123.21, 123.21}-{146.79, 146.79}"
    ]
  },
  "gpu_loader_custom_arc_ctrl": {
    "slice_result_valid": true,
    "toolpath_outside": false,
    "active_notifications": [
      "",
      "Slice ok.",
      "Object name: box_454012915153aa2c.stl\nSize: 24 x 24 x 4 mm\nVolume: 2304 mm³\nTriangles: 12\n<Error></Error>"
    ]
  }
}
```
