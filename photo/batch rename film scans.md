# 照片批量重命名工作流程

## 目标

将数字文件夹中的照片批量重命名并复制到 preprocess 文件夹。

## 目录结构

```
2026_01_21/
├── 05847/
│   └── BMP转JPG/
│       ├── 000001.jpg
│       ├── 000002.jpg
│       └── ...
├── 05848/
│   └── BMP转JPG/
│       └── ...
├── preprocess/          <- 目标文件夹
└── ...
```

## 命名规则

- 格式: `文件夹名_序号.jpg`
- 示例: `05847/BMP转JPG/000001.jpg` → `05847_01.jpg`
- 序号按文件顺序从01开始递增

## 执行脚本

```bash
for dir in C:/cache/film/2026_01_21/05*/; do
  folder_name=$(basename "$dir")
  count=1
  for file in "$dir/BMP转JPG"/*.jpg; do
    if [ -f "$file" ]; then
      new_name=$(printf "%s_%02d.jpg" "$folder_name" "$count")
      cp "$file" "C:/cache/film/2026_01_21/preprocess/$new_name"
      count=$((count + 1))
    fi
  done
  echo "$folder_name: $((count - 1)) files copied"
done
```

## 执行结果

| 文件夹 | 文件数 |
|--------|--------|
| 05847  | 26     |
| 05848  | 24     |
| 05849  | 26     |
| 05850  | 26     |
| 05851  | 37     |
| 05853  | 37     |
| 05854  | 26     |
| 05855  | 26     |
| **总计** | **228** |

## 备注

- 脚本使用 `cp` 复制文件，原文件保持不变
- 序号格式为两位数（01-99），如需更多位数可修改 `%02d`
- 仅处理 `.jpg` 文件
