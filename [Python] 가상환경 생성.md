해당 페이지는 파이썬 사용 시에 가상환경을 만들기 위한 튜토리얼 페이지입니다. 파이썬을 처음 접하시는 분들은 참고하시면 도움이 되리라 생각합니다.ㅎ
  
  
  
  
## 가상환경은 왜 필요할까?
프로젝트를 진행하다보면 여러 라이브러리를 다운로드하여 사용하게 되는데, 그러다 보면 라이브러리들 간의 충돌이 발생하는 경우가 종종 있다. 이러한 상황을 방지할 수 있는 것이 가상환경이기 때문에, 개발자라면 필수적으로 접하게 되는 것 중 하나이다.

가상환경(virtualenv)은 프로젝트별로 독립적인 가상의 환경을 만들어서 라이브러리/모듈 등의 버전을 별도로 지정할 수 있다. 즉 한 컴퓨터에 여러 개발환경을 실행할 수 있게 한다.<br/><br/> 

 

### 가상환경을 만드는 세가지 방법
가상환경을 만들 수 있는 방법은 다양하지만, 자주 사용되는 방법은 크게 3가지로 

(1) `python venv` : python에서 제공중인 가상환경 모듈

(2) `virtualenv` : 가상환경 라이브러리

(3) `conda` : Anaconda에서 제공중인 가상환경 명령어를 사용

이 있는데, 각각 환경을 셋팅하고 활성화하는 방법은 아래와 같다.<br/><br/>

 
---
#### (1) `python venv` 
`venv`는 파이썬 자체에 내장 모듈로 `virtualenv`의 경량화된 버전이다. 매우 쉽게 가상환경을 생성할 수 있다는 장점이 있는 반면, 사용중인 버전(python)으로만 환경을 구축할 수 있다는 문제점을 가지고 있다. 사용중인 라이브러리가 파이썬 버전에도 영향을 받는지 유의하여 사용하도록 하자. 그 외에도 몇가지 단점을 가지고 있는데, 그 부분은 아래와 같다.

`virtualenv`보다 느리다.(`app-data` seed method가 없음)

확장성(extendable)이 떨어진다.

가상환경의 파이썬 버전이 수정이 불가하다.

모듈의 업그레이드가 불가능하다.

[가상환경 생성 및 활성화]


#사용예제
```
$ python -m venv {디렉토리}
$ python -m venv ./myvenv                #예제

#가상환경 활성화
$ source ./myvenv/bin/activate
(myvenv)$                                #정상적으로 가상환경 활성화

#필수 라이브러리 설치 with requirements.txt
(myvenv)$ pip install -r requirements.txt   #설치
(myvenv)$ pip freeze > requirements.txt     #저장
``` 
<br/><br/>
 
---
#### (2) `virtualenv`
앞서 짧게 소개된 바와 같이 확장성을 가진 가상환경 생성모듈이다. 다른 점은 `venv` 명령어 대신 `virtualenv` 명령어를 사용한다는 점과 원하는 파이썬 버전에 맞추어 생성할 수 있다는 점이다. 

 

[가상환경 생성 및 활성화]

```
$ pip install virtualenv
$ cd {디렉토리}
$ virtualenv {가상환경이름}
$ virtualenv myvenv --python=python3.7       #myvenv라는 가상환경에 파이썬3.7설치

#활성화
$ source ./myvenv/bin/activate
(myvenv)$                                    #정상적으로 가상환경 활성화

#필수 라이브러리 설치 with requirements.txt
(myvenv)$ pip install -r requirements.txt   #설치
(myvenv)$ pip freeze > requirements.txt     #저장
```
<br/><br/>
 
---
#### (3) `conda` 
마지막으로 conda 명령어를 통해서 가상환경을 생성하는 방법이다. conda에서 지속적인 관리를 해주고 conda 명령어과 같이 사용시에 매우 간편하게 설치하고 관리할 수 있다는 장점이 있는 반면, conda 위의 가상환경이 만들어지는 만큼  virtualenv 에 비해 무겁다는 단점이 있다.  


[가상환경 생성 및 활성화]

```
# 가상환경 리스트 확인
$ conda env list

#원하는 디렉토리 이동 및 설치
$ cd {디렉토리}
$ conda create -n {가상환경이름} python=3.7
$ conda create -n myvenv python=3.7

#활성화
$ conda activate myvenv
(myvenv) $ 
(myvenv) $ conda deactivate

# 필수 라이브러리 설치 with conda_requirements.txt
conda env create -f conda_requirements.txt  #설치
conda env export > conda_requirements.txt   #저장
```
<br/><br/>


이번 페이지를 통해서는 파이썬의 가상환경을 만드는 방법에 대해서 알아보았습니다. 사실 가상환경을 만드는 방법은 더욱 많겠죠..! 
각자가 편한 방법으로 환경을 생성하시되, 각 방법을 장단점을 이해하고 가장 편한 방법으로 생성하면 큰 무리없이 사용가능하지 않을까하는 생각을 해봅니다.

다음 튜토리얼 시리즈에서 뵈어요..!<br/><br/>

 
