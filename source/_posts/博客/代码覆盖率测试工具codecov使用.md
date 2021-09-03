---
title: 代码覆盖率测试工具codecov使用
description: codecov只是一个测试结果展示平台，并不是测试工具(平台)
abbrlink: 5a4d11d
date: 2021-09-02 14:56:51
categories:
- 博客
tags:
- 测试
---
### codecov是什么
codecov是一个开源的测试结果展示平台，并将测试结果可视化。
> 网址 https://codecov.io

### 如何使用codecov
1. 访问codecov.io网址，使用github账号进行登录
2. 选择需要访问的测试覆盖率的仓库,如https://app.codecov.io/gh/ 首页没有找到对应的repo，可以先点击Not yet setup查看能否找到，再点击setup repo,如果这时还未找到，可以手动在浏览器中输入https://app.codecov.io/gh/${org}/${repo} 进行查看。
![pic-1](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202109021512747.png)
![pic-2](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202109021512836.png)
3. 项目setup以后，在项目中复制CODECOV_TOKEN,并在github中配置对应的secrets
![pic-3](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202109021514408.png)
4. 使用github action触发上传
```yaml
name: Workflow for codecov
on: [push, pull_request]
jobs:
  run:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - name: Setup Python
      uses: actions/setup-python@master
      with:
        python-version: 3.8
    - name: Generate coverage report
      run: |
        pip install coverage
        coverage erase
        coverage run setup.py test
        coverage xml
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v2
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        files: ./coverage.xml
```
5. 访问https://app.codecov.io/gh/${org}/${repo} 查看对应项目的代码覆盖率
![pic-4](https://raw.githubusercontent.com/zhousicong/imagehost/main/img/202109021535218.png)