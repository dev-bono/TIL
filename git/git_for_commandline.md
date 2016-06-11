# git for commandline
소스트리에만 너무 익숙해진 나머지, git에 대한 기본적인 원리를 제대로 이해하지 못하는 문제가 있음
so, 터미널에서 git 명령어를 쓰는 연습을 하기 위함

## 변경사항 수정 

### 변경사항 staging
* 변경 사항을 만들어 인덱스(stage)에 등록하기
```
git add *
git add README // README 파일 추적, 특정 파일만 staging
```
### 삭제된 파일 staging
```
git rm README // 삭제된 README 파일 스테이징
```
* 여러개를 삭제하기 위해서는 file-glob 패턴을 사용한다.
```
ex) git rm log/\*.log
```
### 커밋하기
```
git commit 
git commit --amend
```