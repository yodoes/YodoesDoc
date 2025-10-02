---
description: facefusion3.4.1配置默认目录&解除限制
---

# facefusion3.4.1配置

## 更改默认缓存及生成目录

1、创建目录temp和output用于存储缓存和输出图片

2、修改配置文件，文件路径：\facefusion3.4.1\facefusion\facefusion.ini
```bash
[paths]
temp_path = E:\facefusion3.4.1\temp
jobs_path =
source_paths =
target_path =
output_path = E:\facefusion3.4.1\output
```

## 解除限制

1、解除文件完整性校验，文件路径：\facefusion3.4.1\facefusion\core.py
```bash
修改函数：common_pre_check()

def common_pre_check() -> bool:
	# 原有代码保持不变...    
	return True  # 直接返回True，跳过校验
	common_modules =\
	[
		content_analyser,
		face_classifier,
		face_detector,
		face_landmarker,
		face_masker,
		face_recognizer,
		voice_extractor
	]

	content_analyser_content = inspect.getsource(content_analyser).encode()
	content_analyser_hash = hash_helper.create_hash(content_analyser_content)

	return all(module.pre_check() for module in common_modules) and content_analyser_hash == '803b5ec7'
```

2、解除内容检测限制，文件路径：\facefusion3.4.1\facefusion\content_analyser.py
```bash
修改函数：detect_nsfw()

def detect_nsfw(vision_frame : VisionFrame) -> bool:
	return False  # 跳过内容检测
	is_nsfw_1 = detect_with_nsfw_1(vision_frame)
	is_nsfw_2 = detect_with_nsfw_2(vision_frame)
	is_nsfw_3 = detect_with_nsfw_3(vision_frame)

	return is_nsfw_1 and is_nsfw_2 or is_nsfw_1 and is_nsfw_3 or is_nsfw_2 and is_nsfw_3
```
