---
title: TBSandbox-Day1
date: 2024-04-04 21:02:38
tags: 沙箱分析
---

# 微步沙箱分析-Day1

> 算是一篇简单的水文,主要是吐槽以前有人提出的 微步沙箱里面有CrowdStrike.  也不知道从哪看来的文章就开始信口开河.

![](../img/TBSandbox-Day1/image-20240404214152937.png)

纪念某泄露版CS

##  正文部分

在查看进程中,总是能发现微步把奇奇怪怪的.exe塞到 `C:\Program Files\`和`C:\Program Files (x86)\`中

![](../img/TBSandbox-Day1/image-20240404214212657.png)

以上进程甚至有Viper和微点(微点都死多少年了还能被拉出来鞭尸)

可以基本判断微步的沙箱会在两个程序目录放一堆假的exe用于进程欺诈.而且文件大小一致,开始令人怀疑是不是同一个文件了.于是糊了一个程序来测试

```rust
use std::fs::File;
use std::io::{BufReader, Read};
use std::path::Path;
use sha2::{Digest, Sha256};
use walkdir::WalkDir;
use std::process::Command;
fn calculate_sha256(file_path: &Path) -> String {
    let file = File::open(file_path).expect("Failed to open file");
    let mut reader = BufReader::new(file);
    let mut hasher = Sha256::new();

    let mut buffer = [0; 1024];
    loop {
        let count = reader.read(&mut buffer).expect("Failed to read file");
        if count == 0 {
            break;
        }
        hasher.update(&buffer[..count]);
    }

    let hash = hasher.finalize();
    let hash_string: String = hash.iter().map(|byte| format!("{:02x}", byte)).collect();
    hash_string
}

fn main() {
    let dir_path = "C:\\Program Files\\";
    let walker = WalkDir::new(dir_path).min_depth(1).max_depth(1).into_iter();

    for entry in walker.filter_map(|e| e.ok()) {
        if entry.file_type().is_file() {
            let file_path = entry.path();
            let hash = calculate_sha256(file_path);
            println!("{}: {}", file_path.display(), hash);
        }
    }
    let _ = Command::new("cmd.exe").arg("/c").arg("pause").status();
}

```

![](../img/TBSandbox-Day1/image-20240404214236898.png)

不得不承认,微步还是挺灵的.这回又给你塞了点安全狗 

![](../img/TBSandbox-Day1/image-20240404214250316.png)

所以绕过微步的灵车技巧可能就是通过判断哈希是否相同来搞了
