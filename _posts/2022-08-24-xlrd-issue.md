---
layout: post
title: "xlrd.biffh.XLRDError: Excel xlsx file; not supported" 이슈
excerpt: "xlrd.biffh.XLRDError: Excel xlsx file; not supported" 이슈
date: 2022-08-24
tags: [python, django]
toc: false
---

## xlrd.biffh.XLRDError: Excel xlsx file; not supported 이슈

다른 사람이 작성한 django 앱을 수정할 일이 있었는데 라이브 서버에서는 문제가 없던 엑셀 파일 작성 기능이 로컬컴퓨터에서 오류가 발생했다.

xlrd 에서 더이상 xlsx 파일을 지원하지 않아서 생기는 문제라고 한다.

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

requirements 파일에 xlrd 의 버전이 명시 되지 않아있어 최신 버전으로 설치 되어 있었고 버전을 명시해 줌으로써 기존코드를 변경 하지 않고 해결했다.

```bash
# requirements file
...
xlrd==1.2.0
...
```

이후 `pip install -r requirements` 명령으로 패키지 재설치후 정상 작동 확인했다.

---

### Reference

stackoverflow: <https://stackoverflow.com/questions/65254535/xlrd-biffh-xlrderror-excel-xlsx-file-not-supported/65266270#65266270>
