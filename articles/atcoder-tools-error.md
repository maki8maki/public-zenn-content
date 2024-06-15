---
title: "atcoder-toolsの実行時のエラーと対処法"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["atcoder", "python"]
published: false
---

## はじめに

[C++用のAtCoder環境](https://github.com/maki8maki/atcoder-cpp-docker.git)作成時に遭遇したatcoder-tools実行時のエラーとその対処法を記録用に残します。他のモジュールを実行した際にも同様のエラーが出るみたいなので、そのような人も参考にしてください。

## 実行環境

- DockerのPython公式イメージのpython:3.11-slim-bookworm

## エラー内容

`atcoder-tools gen {contest_id}` を実行した際に以下のエラーが発生する。

```bash
Traceback (most recent call last):
  File "/usr/local/bin/atcoder-tools", line 5, in <module>
    from atcodertools.atcoder_tools import main
  File "/usr/local/lib/python3.11/site-packages/atcodertools/atcoder_tools.py", line 8, in <module>
    from atcodertools.tools.envgen import main as envgen_main
  File "/usr/local/lib/python3.11/site-packages/atcodertools/tools/envgen.py", line 14, in <module>
    from atcodertools.client.atcoder import AtCoderClient, Contest, LoginError, PageNotFoundError
  File "/usr/local/lib/python3.11/site-packages/atcodertools/client/atcoder.py", line 15, in <module>
    from atcodertools.common.language import Language
  File "/usr/local/lib/python3.11/site-packages/atcodertools/common/language.py", line 4, in <module>
    from atcodertools.codegen.code_generators import cpp, java, rust, python, nim, d, cs, swift, go, julia
  File "/usr/local/lib/python3.11/site-packages/atcodertools/codegen/code_generators/cpp.py", line 2, in <module>
    from atcodertools.codegen.template_engine import render
  File "/usr/local/lib/python3.11/site-packages/atcodertools/codegen/template_engine.py", line 5, in <module>
    from jinja2 import Environment
  File "/usr/local/lib/python3.11/site-packages/jinja2/__init__.py", line 12, in <module>
    from .environment import Environment
  File "/usr/local/lib/python3.11/site-packages/jinja2/environment.py", line 25, in <module>
    from .defaults import BLOCK_END_STRING
  File "/usr/local/lib/python3.11/site-packages/jinja2/defaults.py", line 3, in <module>
    from .filters import FILTERS as DEFAULT_FILTERS  # noqa: F401
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.11/site-packages/jinja2/filters.py", line 13, in <module>
    from markupsafe import soft_unicode
ImportError: cannot import name 'soft_unicode' from 'markupsafe' (/usr/local/lib/python3.11/site-packages/markupsafe/__init__.py)
```

## 対処法

[このサイト](https://keep-loving-python.hatenablog.com/entry/2022/08/29/121515)で紹介されているように `pip install markupsafe==2.0.1` とバージョンを固定してインストールする。ちなみに元のバージョンを2.1.5でした。

## 参考

- [解決策。ImportError: cannot import name 'soft_unicode' from 'markupsafe'](https://keep-loving-python.hatenablog.com/entry/2022/08/29/121515)
