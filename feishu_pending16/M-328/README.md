# M-328 · Organic / 3D 树支撑仅在打印板上（build_plate_only）时缺少支撑界面层

- **飞书待核实编号**：`M-328`
- **上游 PR**：`PR #13192`
- **执行通道**：`desktop-cli`
- **源码定位**：`src/libslic3r/Support/TreeSupport.cpp:2453 vs TreeSupport3D.cpp:3946`
- **实测结论**：**`REPRODUCED`**（耗时 `9527.6 ms`）

---

## 一、Bug 说明

经典树支撑已支持仅底板模式的界面层恢复，但 Organic/3D 树支撑的界面层恢复仍在禁用分支中，开启 support_on_build_plate_only=1 时可能缺少支撑界面层。

## 二、本目录案例文件

### 输入模型（`inputs/`，可直接拖入 Snapmaker Orca 复现）
- `inputs/stepped_overhang_pedestal_227cd24b3c08d982.stl`

### 产物与证据（`outputs/` 与 `result.json`）
- `result.json`（完整结构化实测指标、配置回读与触发证明）
- `outputs/m328_organic_bp_only.gcode.3mf`
- `outputs/m328_organic_everywhere.gcode.3mf`
- `outputs/m328_slim_bp_only.gcode.3mf`

## 三、手动复现步骤

1. 导入 `inputs/` 中的阶梯悬垂底座模型。
2. 设置 `enable_support = 1`、`support_type = tree(auto)`、`support_style = organic`、`support_interface_top_layers = 2`。
3. 对比 `support_on_build_plate_only = 1`（实验组）与 `0`（对照组）以及 `tree_slim` 的 Support interface 挤出条数。

## 四、自动化单条重跑命令

```powershell
python python/tests/test_feishu_pending16.py --exe <pack>/snapmaker-orca.exe --out <out-dir> --case M-328
```

## 五、实测关键数据

```json
{
  "organic_build_plate_only": {
    "support_moves": 16716,
    "interface_moves": 129,
    "interface_len_mm": 360.679
  },
  "organic_everywhere_ctrl": {
    "support_moves": 31122,
    "interface_moves": 156,
    "interface_len_mm": 1071.415
  },
  "tree_slim_build_plate_only_ctrl": {
    "support_moves": 41182,
    "interface_moves": 1021,
    "interface_len_mm": 2647.726
  }
}
```
