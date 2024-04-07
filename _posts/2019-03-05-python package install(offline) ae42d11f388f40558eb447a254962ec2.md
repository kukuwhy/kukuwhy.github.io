---
layout: post
title:  "python package install(offline)"
tags: python package
---

# 개요

내부망 점검 등 외부 네트워크 사용이 불가능할 경우 python package를 offline에서 설치해서 가져가야한다. 그래서 반입 전 패키징 방법과 반입 후 설치 과정을 정리한다.

* 의존성 문제를 방지하기 위해 페쇄망 python version을 꼭 맞춰주자

## 1. package list 확인

pip list를 이용해서 필요한 package가 있는지 모두 확인한다.

```python
pip list
```

## 2.  package download

python package install 파일이 다운로드 된다(확장자 : whl)

```python
# package 리스트 출력 
pip freeze > pkgList.txt

# package 리스트 다운로드
pip download -r pkgList.txt
```

## 3. package install

리스트와 다운로드된 모듈을 반입하고 python 설치 후 아래 명령어 입력 시 설치가 된다.

```python
pip install --no-index --find-links=. -r pkgList.txt
```