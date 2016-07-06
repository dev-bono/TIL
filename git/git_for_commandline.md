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
* 여러개를 삭제하기 위해서는 file-glob 패턴을 사용한다. 쉘에서 파일 삭제후 처리해도 되지만, git rm 명령어를 쓰면 아마도 삭제후 바로 stage에 올라가는듯하다.
```
git rm log/\*.log // log 폴더안에 확장자가 log인 파일 모두 삭제
git rm \*~ // ~로 끝나는 모든 파일 삭제

```
### 커밋하기
```
git commit 
git commit --amend
```
### rebase 과정
일반적으로 git을 사용할때 원본브랜치에서 새로운 브랜치를 만들어 작업한 후에 원본 브랜치에 머지하는 과정을 거친다.
그런데, 작업 중 원본브랜치가 수정되었다면, 작업한 브랜치에다 원본 브랜치를 머지한 후 다시 작업한 브랜치를 원본 브랜치에 머지하는 과정을 거쳐야 한다.
사실 이렇게 git을 관리해도 문제가 있는건 아니지만, git이 지저분해지는 점이 보기 싫을 수 있다.
특히 소스트리와 같이 git GUI 프로그램을 사용하지 않고 커맨드라인에서 git을 관리하는 경우에는 복잡해진 git 그래프를 알아보기란 여간 어려운 일이 아니다.
이런 문제를 해결하기 위해서는 rebase라는 명령과 개념을 이해하고 적용하면 된다.
```
git rebase -i HEAD~4 
```
* -i 옵션은 interactive, HEAD~4는 4개 커밋 내용을 정리하겠다는 의미
* rebase 후에는 로컬에 같은 이름으로 브랜치 생성 (origin에 푸시되어 있는 경우에만 해당)
* 다음 명령으로 원본 브랜치 삭제함
```
git push origin :rebase-branch
```

### local tag 오리진 서버에 push 하기
```
git push origin 태그명
```
