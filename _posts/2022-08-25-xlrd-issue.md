---
layout: post
title: xlrd.biffh.XLRDError Excel xlsx file; not supported 이슈
excerpt: xlrd.biffh.XLRDError Excel xlsx file; not supported 이슈
date: 2022-08-25
tags: [python, django]
---

## xlrd.biffh.XLRDError: Excel xlsx file; not supported 이슈

xlrd 2.0.0 이상 에서 잠재적 보안 취약성 때문에 xlsx 파일을 더이상 지원하지 않아서 생기는 문제라고 한다.

다음과 같은 방법으로 해결 할수 있다.

1. pandas version 1.0.1 이상

2. openpyxl 설치

3. pandas 코드를 아래와 같이 변경

```python
df1 = pd.read_excel(
     os.path.join(APP_PATH, "Data", "aug_latest.xlsm"),
     engine='openpyxl',
)
```

다른방법으로는 xlrd의 버전을 xlsx를 지원 하는 버전으로 변경하는것이다.

requirements 파일에 xlrd 의 버전이 명시 되지 않아있어 최신 버전으로 설치 되어 있었고 버전을 명시해 줌으로써 1.2.0 버전을 설치할수 있다.

```bash
# requirements file
...
xlrd==1.2.0
...
```

이후 `pip install -r requirements` 명령으로 패키지를 재설치 한다.

---

### Reference

stackoverflow: <https://stackoverflow.com/questions/65254535/xlrd-biffh-xlrderror-excel-xlsx-file-not-supported/65266270#65266270>
