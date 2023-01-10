
- github 地址：https://github.com/alist-org/alist

- 安装说明：https://alist.nn.ci/zh/guide/install/script.html

- 一键安装脚本：

> 默认安装在 /opt/alist 中。 自定义安装路径，将安装路径作为第二个参数添加，必须是绝对路径（如果路径以 alist 结尾，则直接安装到给定路径，否则会安装在给定路径 alist 目录下），如 安装到 /root：

```
# Install
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s install /root

# update
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s update /root

# Uninstall
curl -fsSL "https://alist.nn.ci/v3.sh" | bash -s uninstall /root
```

- 第三方介绍：https://www.bilibili.com/read/cv17271986