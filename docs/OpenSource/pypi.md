# Pypi

> [Pypi 官网](https://pypi.org/)

> requirement

- python3
- pip
- setuptools
- wheel
- twine

> 库文件架构
```bash
-- packaging_tutorial      # 项目名字
    -- <your pkg>          # 包名字
        -- __init__.py     # 一定要有init文件
        -- print_str.py    # 功能实现
    -- __init__.py         # 一定要有init文件
    -- README.md           # 一般记录具体使用方法
    -- setup.py            # 打包程序
```

> setup.py [For Detail setuptools](https://packaging.python.org/guides/distributing-packages-using-setuptools/)

```python
import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="debug-world",                                     # 包的分发名称，使用字母、数字、_、-
    version="0.0.1",                                        # 版本号, 版本号规范：https://www.python.org/dev/peps/pep-0440/
    author="liheyou",                                       # 作者名字
    author_email="author@example.com",                      # 作者邮箱
    description="PyPI Tutorial",                            # 包的简介描述
    long_description=long_description,                      # 包的详细介绍(一般通过加载README.md)
    long_description_content_type="text/markdown",          # 和上条命令配合使用，声明加载的是markdown文件
    url="https://github.com/",                              # 项目开源地址，我这里写的是同性交友官网，大家可以写自己真实的开源网址
    packages=setuptools.find_packages(),                    # 如果项目由多个文件组成，我们可以使用find_packages()自动发现所有包和子包，而不是手动列出每个包，在这种情况下，包列表将是example_pkg
    classifiers=[                                           # 关于包的其他元数据(metadata)
        "Programming Language :: Python :: 3",              # 该软件包仅与Python3兼容
        "License :: OSI Approved :: MIT License",           # 根据MIT许可证开源
        "Operating System :: OS Independent",               # 与操作系统无关
    ],
)
```

> 本地打包

```bash
# 运行setup.py
python setup.py sdist
# 或者
python setup.py sdist bdist_wheel
```

> 上传Pypi

```bash
pip install twine     # 如果已经安装twine，跳过次步骤
> twine upload dist/*

# 接着会让你输入PyPI用户名和密码，注意不是邮箱和密码
# 出现下面信息说明成功，如有错误信息，检查setup.py配置信息
>>> Uploading distributions to https://upload.pypi.org/legacy/
>>> Uploading debug-world-0.0.1.tar.gz 
>>> 100%█████████████████████████| 5.56k/5.56k [00:01<00:00, 3.98kB/s]

如果不想每次上传都输入账号和密码，可以创建用户验证文件 ** ~/.pypirc**
# 而且此方法不安全，容易泄露密码, 因为密码是明文
[distutils]
index-servers =
    pypi

[pypi]
repository: https://upload.pypi.org/legacy/
username: <username>
password: <password>
```

> 去 [pypi](https://pypi.org/) 验证

> 安装

```bash
pip install <your pkg> -i https://pypi.python.org/pypi
```

> ⚠️ 注意

- 包名一定是别人没用过的
- 项目文件一定要有** init.py**
- 运行setup.py文件一定要同级目录
- 在上传PyPI的是时候输入的是用户名和密码，不是邮箱和密码
- 上传之后需要等一段时间，才能下载最新版本的包
- 更改包的时候一定要修改版本号
- pip 按照版本号安装，==前后没有空格

---

> 📚 reference

[1] https://packaging.python.org/