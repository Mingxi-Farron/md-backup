# 胶片扫描文件批量重命名工作流程

## 目标

将按卷号分类的胶片扫描文件批量重命名并复制到统一文件夹。

## 适用场景

- 扫描仪按卷号输出到不同文件夹
- 需要将所有扫描文件汇总到同一目录
- 需要按 `卷号_帧号` 格式统一命名

## 目录结构

```
<工作目录>/
├── <卷号1>/
│   └── <子文件夹>/        # 可选，如 "BMP转JPG"
│       ├── 000001.jpg
│       ├── 000002.jpg
│       └── ...
├── <卷号2>/
│   └── <子文件夹>/
│       └── ...
├── <输出文件夹>/          # 如 preprocess
└── ...
```

## 命名规则

- 格式: `<卷号>_<序号>.jpg`
- 示例: `05847/子文件夹/000001.jpg` → `05847_01.jpg`
- 序号按文件顺序从 01 开始递增

## 执行脚本

```bash
# ============ 配置区域 ============
WORK_DIR="/path/to/work/dir"      # 工作目录
FOLDER_PATTERN="*"                 # 卷号文件夹匹配模式，如 "05*" 或 "*"
SUB_FOLDER="BMP转JPG"              # 子文件夹名称，留空则直接读取卷号文件夹
OUTPUT_DIR="preprocess"            # 输出文件夹名称
FILE_EXT="jpg"                     # 文件扩展名
# ==================================

mkdir -p "$WORK_DIR/$OUTPUT_DIR"

for dir in "$WORK_DIR"/$FOLDER_PATTERN/; do
  folder_name=$(basename "$dir")

  # 跳过输出文件夹本身
  [ "$folder_name" = "$OUTPUT_DIR" ] && continue

  # 确定源文件路径
  if [ -n "$SUB_FOLDER" ]; then
    src_path="$dir/$SUB_FOLDER"
  else
    src_path="$dir"
  fi

  [ ! -d "$src_path" ] && continue

  count=1
  for file in "$src_path"/*."$FILE_EXT"; do
    if [ -f "$file" ]; then
      new_name=$(printf "%s_%02d.%s" "$folder_name" "$count" "$FILE_EXT")
      cp "$file" "$WORK_DIR/$OUTPUT_DIR/$new_name"
      count=$((count + 1))
    fi
  done

  [ $count -gt 1 ] && echo "$folder_name: $((count - 1)) files copied"
done

echo "Done! Files saved to: $WORK_DIR/$OUTPUT_DIR"
```

## 配置说明

| 变量 | 说明 | 示例 |
|------|------|------|
| `WORK_DIR` | 工作目录绝对路径 | `/home/user/scans` |
| `FOLDER_PATTERN` | 卷号文件夹匹配模式 | `05*`, `roll_*`, `*` |
| `SUB_FOLDER` | 子文件夹名称 | `BMP转JPG`, 留空 |
| `OUTPUT_DIR` | 输出文件夹名称 | `preprocess`, `output` |
| `FILE_EXT` | 文件扩展名 | `jpg`, `tif`, `png` |

## 备注

- 使用 `cp` 复制文件，原文件保持不变
- 序号格式为两位数（01-99），如需更多位数可修改 `%02d` 为 `%03d`
- 自动跳过输出文件夹，避免重复处理
- 支持 Windows (Git Bash/MSYS2) 和 Linux/macOS
