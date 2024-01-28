# config
- config 이름과 이메일은 협업을 위해 설정하는 것임
(프로젝트마다 다르게 설정할 수 있음)
- git config --global user.name
git config --global user.name "(본인 이름)"

- 디폴트 브랜치명
git config --global init.defaultBranch main


- git diff는 변경사항을 좀 더 자세하게 볼 수 있음

# git reset
 시간을 과거로 되돌림
 이후 행적은 히스토리에서 지워버림
 
 git reset --hard 0bf487 (해쉬코드 앞 6글자 정도만 복사해도됨)
 git reset --hard e659da
 
 reset전으로 가려면 백업해놓은 .git을 넣으면 됨
 
 git reset으로 첫 커밋으로 돌아간 상태인데 history는 복구가 되었다.
 그래서 First Commit이 아닌 Last Commit으로 깃은 인식하고 있는데 막상 파일은 First Commit이므로 깃은 Last Commit에 상태에서 First Commit 파일 상태로 바뀌었다고 인식하고 있는걸 git status를 보면 알 수 있다.(몇몇 파일이 그냥 없는 상태에서 history가 돌아왔는데 그저 파일이 없단 이유로 삭제된 줄 아는것임)
 그 때는 git reset --hard를 하면 가장 최근 커밋 상태로 파일을 되돌림
 
# git revert
 했던 거를 거꾸로 함
 이걸 사용하면 수치스러운 과거에 했던 행동만 콕 집어서 되돌릴 수 있음
 
 git revert hashcode
 
 근데 만약 내가 2번째 커밋을 없던 일로 하려고 한다
 거기에는 A라는 파일을 더했다
 그래서 2번째 커밋을 되돌려서 A라는 파일이 삭제된다.
 그렇게 6번째 커밋이 "revert 2번째 커밋"이 되어야 하는데
 4번째 커밋중 A라는 파일을 수정한 커밋이 있다.
 그래서 A를 수정했던 것 버전과 충돌하게 된다.
 그래서 git rm leopards.yaml로 이걸 삭제하는것이 중요하다.
 
 >git rebert --no-commit을 하면 revert 한 상태에서 커밋이 안되었기 때문에 다른 작업을 하고 커밋을 할 수도 있다.
 
 # branch
 
 - 프로젝트를 하나 이상의 모습으로 관리해야 할 때
 	- 실배포용, 테스트 서버용, 새로운 시도용
 - 여러 작업들이 각각 독립되어 진행될 때
   - 예) 신기능 1, 신기능 2, 코드개선, 긴급수정 ...
   - 각각의 차원에서 작업한 뒤 확정된 것을 메인 차원에 통합
   
이 모든 것을 하나의 프로젝트 폴더에서 진행할 수 있게 해줌

git branch add-coach

git branch

git switch add-coach
(checkout 명령어가 Git 2.23 버전부터 switch, restore로 분라)

git switch -c 2 
만들면서 이동하기

git branch -d (브랜치명)
삭제

git branch -m (기존) (새 거)
이름 바꾸기

git log --all --decorate --oneline --graph
깃 트리 보기
   
   
    
 
 # git merge && rebase
 
(main) git merge 2 (main에서 흡수될 브랜치를 대상으로)
 
(2) git rebase main(반대로)
 
 > 근데 여기서 2브랜치와 main 브랜치의 위치가 다를 것이다(실제로 main으로 가보면 2의 내용이 없을 것이다) -> 여기서 가장 최신은 2일 것이다
 이제 main에 완전히 병합하기 위해 merge를 사용한다
 그럼 이제 다시 main으로 가서 git merge 2를 한다
 (main) git merge 2
 
>rebase할 대상 브랜치로 가서 main 브랜치에 가지를 이어 붙이고 main의 새싹을 merge해서 이동시키면 된다.

### 충돌 시 merge

![](https://velog.velcdn.com/images/jckim22/post/995f69d3-2896-498d-bf1a-90b1d9a099bb/image.png)

 원하는 걸 하면되고 만약 돌아가고 싶으면
 git merge --abort 
 
 다 했으면 다시 커밋
 
 ### 충돌 시 rebase
 
 rebase는 브랜치의 히스토리 자체를 옮기기 때문에 커밋마다 충돌을 다 해결해줘야한다(역사도 합칠것이기 때문)
 
 gir rebase main 하고
 충돌 해결하고
 git add 하고
 git rebase --continue 하고 커밋을 한다
 그 다음 커밋을 해결하고
 git add 하고
 git rebase --continue하면
 ![](https://velog.velcdn.com/images/jckim22/post/fe390937-ad61-40bc-a03e-f85f2794b343/image.png)

해결 완료

그리고 마지막으로 main으로 가서 merge 2를 해주면 된다.
> 다 쓴 1, 2 브랜치는 삭제하자 (그 때마다 삭제하는 것이 좋다)

git branch -D 1
git branch -D 2

![](https://velog.velcdn.com/images/jckim22/post/65b96eda-859b-464c-a9b7-f8deec07bedf/image.png)

깔끔해진 것을 볼 수 있다.

# Github

Personal access token 만들기(프로젝트에 접근할 수 있는 토큰)

git remote add origin (원격 저장소 주소)

git branch -M main
기본 브랜치명을 main으로

git push -u origin main
로컬 저장소의 커밋 내역을 원격으로 push
(-u로 현재 브랜치와 명시된 으ㅓㄴ격 브랜치 기본 연결)

origin은 현재 연동된 remote repo를 뜻함

Git clone으로 원격에 저장된 커밋들을 가져옴

## 실습

오리진에 푸쉬하고 로컬에서 작업을 한 다음 트리를 보자

![](https://velog.velcdn.com/images/jckim22/post/a9556b3d-b315-4dd4-9ccc-57278e49f801/image.png)

위처럼 로컬의 main이 origin의 main보다 한 단게 커밋이 앞서 있다.
이제 push를 하자

git push

![](https://velog.velcdn.com/images/jckim22/post/773d1578-8094-4899-a274-fcd39652a080/image.png)

이제 원격에서 하나 작업을 완료하고
pull을 해보자

git pull

![](https://velog.velcdn.com/images/jckim22/post/a705c723-1579-4838-ab70-cee22bc6c6c1/image.png)

위처럼 된다


#### 같은 커밋 단계에서 다른 팀원은 다음 커밋을 해서 원격에 올렸는데 나는 아직 로컬에 커밋사항이 있는 경우는 어떻게 할까?

git push를 해보자

![](https://velog.velcdn.com/images/jckim22/post/34c62892-4521-4976-bb87-5def943de873/image.png)

내 로컬 커밋 상태가 원격의 최신을 따라가지 못하고 있기 때문에 오류가 난다.

그럼 git pull을 하자

방법이 2가지가 있다

git pull --no-rebase
merge를 하는 것이다.


![](https://velog.velcdn.com/images/jckim22/post/33242231-2ff6-4294-8621-517dff1ec069/image.png)

origin의 브랜치를 다른 브랜치로 생각하고 로컬 main과 merge가 된 것을 볼 수 있다.
로컬 main의 커밋과 origin의 커밋 단계는 같기 때문에 merge가 된 것이다.

이제 다른 방법인 rebase를 해보기 위해 reset을 해주자

git reset --hard hashcode

gir pull --rebase 를 해보자
그럼 내 로컬의 시간선을 origin 위에 딱 붙인다

![](https://velog.velcdn.com/images/jckim22/post/4680cd41-9dcb-41b0-b090-0a2552e578e2/image.png)

자 이제 여기서 push를 하면 된다
git push

![](https://velog.velcdn.com/images/jckim22/post/269000a9-8c3c-4fcf-9ed1-76e80ad59627/image.png)

## 충돌이 난다면 ?

일단 위와 같은 상황이다
둘 다 A라는 커밋에서
나는 로컬에서 작업하고 커밋을 했고
동료는 작업하고 커밋하고 푸쉬까지 완료햇다

나는 푸쉬를 하려고 하는데 원격에 이미 동료가 푸쉬를 해서 맞지 않아 rebase를 하려고 한다.
gir pull --rebase

근데 아래처럼 충돌이 생겼다.

![](https://velog.velcdn.com/images/jckim22/post/a256862f-ab2b-449c-98f9-da518db4e6be/image.png)

충돌을 해결해보자

그리고 git add
git rebase --continue를 해주면?
![](https://velog.velcdn.com/images/jckim22/post/e686c9ca-01fe-4b89-8416-03e224b2b77f/image.png)

>rebase는 어떤 선택을 했냐에 따라서 커밋 수가 달라질 수 있다.

> 협업 시 local에서 작업할 때 뭔가 공유된 것들을 rebase해서 올리지 마라
>> 다만 pull할 때 rebase는 상관 없다.

## 강제 푸쉬

로컬에 있는 내용이 원격보다 뒤쳐져 있을 때는 푸쉬할 수 없지만
원격에 잘못된 것이 올라갔을 경우에는 git push --force로 로컬에 있는 걸로 원격을 덮어씌울 수 있다.

# 원격 브랜치 다루기
로컬에서 브랜치를 새로 만들고 푸쉬를 하면 어디로 푸쉬해야할지 몰라서 오류가 뜬다
그래서 git push -u origin from-local (-u눈 upstream의 약자이다)
으로 원격에 from-local 브랜치룰 생성해서 거기에 푸쉬한다는 것을 알려주어야 한다ㅣ.

git branch -a
git branch --all
원격 브랜치들까지 다 볼 수 있다.

### 팀원이 브랜치를 올려서 그걸 받을 때는 어떡할까?
원격에서 브랜치를 하나 생성하자

아직 로컬에서 원격의 브랜치가 보이지 않는다
로컬에 깃이 원격의 변화를 업데이트 하지 않았기 때문이다.

git fetch로 업데이트 해주자

그럼 확인이 된다.


git switch -t origin/from-remote (원래는 squash 였을 것임)를 하자
이것은 원격에서 있는 브랜치를 로컬에 생성해 연결해서 쓰겠다는 것이다.

![](https://velog.velcdn.com/images/jckim22/post/01b8fb77-c43d-4383-9d6a-be7758628acf/image.png)


### 원격 브랜치 삭제

git push (원격 이름) --delete (원격의 브랜치명)


# Git의 장점

Git은 스냅샷 방식으로 그냥 파일이 풀로 저장되어있어서 커밋이 아무리 많아도 불러오지 않아도 되기 때문에 느리지 않다.
그리고 Git은 원격에 의존하지 않고 로컬에서도 각각 진행할 수 있어서 좋다

# Git의 3가지 공간
커밋이 된 상태 -> Repository
수정 사항이 생기면 -> Working directory
git add로 준비 상태 -> Staging area (파일들을 드래그 해서 선택한 것과 똑같음)
커밋이 된 상태 -> Repository

## 파일의 삭제와 이동

git rm
삭제와 add까지 한번에 해줌

git mv
파일 이름 수정과 add까지 한번에 해줌

## 스테이지에서 빼기

git restore --staged "파일명"

## 아예 수정 사항을 없애고 싶다

git restore "파일명"



# Git reset의 옵션

hard -> 그 내역 자체를 지워버림 Worging directory에서도 날려버림
mixed -> working directory에는 남기지마 Staging area에서는 없애버림
soft -> commit만 안된 상태로 만듬(staging)

# Git HEAD

![](https://velog.velcdn.com/images/jckim22/post/6ddac37d-d7e7-4cb6-bde2-e961e86b982e/image.png)

HEAD: 각 브랜치의 최신 커밋


#### git checkout으로 앞뒤 이동해보기

시간선을 바꾸지 않고 파일들의 상태만 옮길 수 있다.

예로 delta-branch에서 전 단계의 파일 상태를 가져오겠다고 해보자
git checkout HEAD^
이렇게 하면 된다
HEAD로 부터 ^개수만큼 전에 커밋의 파일 상태를 가져오는 것이다
>시간선은 그대로 둔다

그럼 HEAD가 하나 뒤로 간다.

git checkout - 해주면 돌아온다

>근데 왜 HEAD가 움직이는가?
HEAD는 각 브랜치의 최신 커밋이라고 하였는데?
익명의 다른 브랜치를 만들어서 움직이는 것이기 때문이다.(커밋의 해쉬코드로 네이밍이 된)

다시 git switch delta-branch를 하면 원래대로 돌아온다.

**checkout으로 왔다갔다 하면서 어떤 특정 부분에서 새로운 브랜치를 만들어서 그 부분을 또 분기점으로 만들 수 있는 것이다!**

### HEAD로 reset 하기

git reset --hard HEAD^2

로 하면 2단계 전으로 리셋된다.

# Fetch와 Pull

fetch: 원격 저장소의 최신 커밋을 로컬로 가져오기만 함 (보이지 않는 브랜치에 가져와서 확인만 함)
pull: 그 이후에 merge또는 rebase까지 진행함

fetch는 아까 HEAD를 이동할 때 처럼 익명의 브랜치에 정보를 담아오기 때문에 checkout으로 확인할 수 있다.

**시나리오**
원격에 다른 동료가 올린 커밋이 있는데 나는 업데이트가 되지 않았다.
그런데 바로 merge하기에는 부담스러워서 한번 테스트를 해보려고 한다.

git fetch로 origin/main의 최신 커밋을 받아오고  git switch가 아닌(익명 브랜치이기 때문에) git checkout origin/main으로 그 최신 커밋을 받아온 익명 브랜치로 이동해서 거기서 맘껏 테스트를 한다.

이 정도면 괜찮다고 생각이 들면 git switch main하고 git pull을 진행한다.

원격에 새로운 브랜치가 있다면
git fetch origin/new-branch
하고 git checkout origin/new-branch로 확인을 해주고
ok 사인이 되면 git switch -t origin/new-branch를 해서 로컬에도 브랜치를 받아오자

# Git help

git help
git help -a

또는 git 문서로 가자

# Git의 각종 설정

git config

### global 설정
컴퓨터에서 설정
git config global user.name -----
### local 설정
git config user.name -----


### 설정값 확인

git config --list

git config global --list

### 유용한 설정들

git config --global core.autocrlf (window: true / mac: input)
윈도우와 협업할 때 문제가 없게 하기 위함

# 프로답게 커밋하기

### 1. 작업을 커밋할 때 권장사항
#### 1. 하나의 커밋에는 한 단위의 작업을 넣도록 합니다.
- 한 작업을 여러 버전에 걸쳐 커밋하지 않습니다.
- 여러 작업을 한 버전에 커밋하지 않습니다.
#### 2. 커밋 메시지는 어떤 작업이 이뤄졌는지 알아볼 수 있도록 작성합니다.

## 2. 커밋 메시지 컨벤션
널리 사용되는 커밋 메시지 작성방식

```
type: subject

body (optional)
...
...
...

footer (optional)
```

**예시**

```
feat: 압축파일 미리보기 기능 추가

사용자의 편의를 위해 압축을 풀기 전에
다음과 같이 압축파일 미리보기를 할 수 있도록 함
 - 마우스 오른쪽 클릭
 - 윈도우 탐색기 또는 맥 파인더의 미리보기 창

Closes #125
```

- Type

![](https://velog.velcdn.com/images/jckim22/post/6dd97221-79b5-408e-b715-c49149e71af4/image.png)

- Subject
커밋의 작업 내용 간략히 설명


- Body
길게 설명할 필요가 있을 시 작성

- Footer
Breaking Point 가 있을 때
특정 이슈에 대한 해결 작업일 때

## Git emoji
https://gitmoji.dev/

## 보다 세심하게 스테이징하고 커밋하기

평소에는 git add
git commit 했다


이제는 

git add -p로 부분(헝크)별로 add를 할 수 있다.

이렇게 부분만 add를 하면 git status를 볼 때 같은 파일이 add와 modify 2개에 다 있는 것을 볼 수 있다.

그냥 git commit -v를 하면 커밋 메시지 작성을 하는 곳이 뜬다
그리고 원하는 부분만 확인하면서 커밋이 된다.

## git stash(커밋하기 애매한 변화 치워두기)

한참 작업을 진행하고 있었는데 급하게 작업할 일이 생긴다

그 때 잠시 다른 공간에 치워두자

staging 된 상태에서
git stash 해보자

그럼 사라진다

이건 스택에 보관되는 것이다

그래서 이제 작업을 하다가 stash를 하고 다른 작업을 하다가 원래 브랜치나 다른 브랜치에 stash에 있는 내용을 적용할 수 있는 것이다.

git stash pop
스태쉬한 것을 가져온다 (충돌이 일어날 수도 있음)

### 원하는 것만 stash 하기

git stash -p

git stash -m 'Add Stash3'
메시지로 치울 수 있다.

git stash list로 스태쉬한 리스트를 확인할 수 있다.

>리스트상의 번호로 apply, drop, pop 가능
ex) git stash apply stash@{1}
ex) git stash drop stash@{1}
위 2개를 합하면 pop
git stash branch new-branch (새로운 브랜치에 pop)
git stash clear (비우기)


# 커밋 수정하기

git commit --amend

### 2. 커밋에 변화 추가
애매한 작업이 있다

작업을 수정하고 스테이징한다
git commit --amend로 마지막 커밋에 포함
커밋 메시지 아무렇게나 변경

git commit --amend -m 'Add members to Panthers and Pumas'
위처럼 한줄로 변경도 가능하다.



# 과거의 커밋을 수정, 삭제, 병합, 분할하기

git rebase -i (대상 바로 이전 커밋 해시코드)
- 과거 커밋 내역을 다양한 방법으로 수정 가능

![](https://velog.velcdn.com/images/jckim22/post/9d9ecdc2-321e-4fe8-a999-a6768e97eddd/image.png)

![](https://velog.velcdn.com/images/jckim22/post/38be41b8-213d-43e7-8ac0-5bc150b79a19/image.png)

한번에 특정 커밋을 삭제하고 또 어떤 커밋들은 합치는 작업을 진행한다

합치는 작업은 squash로 진행되기 때문에 합칠 커밋을 s로 해준다
그럼 삭제되고 이제 두 커밋을 합칠 것이다
그럼 잠깐 두 브랜치로 분기 되고 리베이스가 되면서 합쳐진다.
그리고 커밋끼리는 스쿼시로 합쳐진다

2개의 작업이 한 커밋에 있어서 분할할 것이다
e를 사용하면 된다.
git reset HEAD^를 하면 커밋 전으로 돌아간다.
그리고 각자 커밋을 해주고 git add 하고 git rebase continue를 하면 된다.

결국 왜 rebase인가

git은 모든 커밋 내역들이 순차적으로 저장이된다.
그런데 과거에 어떤 내역이 변경되면 이전과 다른 상태가 된다

그래서 과거의 어떤 내용을 수정하기 위해서는 거기만 수정하는 것이 아니라 아예 거기서 분기를 빼서 끝까지 모든 커밋을 수정하고 붙여버리는 rebase를 사용하게 되는 것이다.


# 관리되지 않는 파일들 삭제하기

git clean
Git에서 추적하지 않는 파일들 삭제


![](https://velog.velcdn.com/images/jckim22/post/2207b805-e47f-40a8-b6de-7ea067162d95/image.png)

파일들 추가한 뒤 옵션 조합과 함께 clean 명령어 사용해보기
toClean1.txt
toClean2.txt
dir/toClean3.txt

💡 흔히 쓰이는 조합: git clean -df

# 커밋하지 않은 변경사항 되돌리기

git restore (파일명)
파일 하나 되돌리기

git restore --staged (파일명)
스테이지에서 빼고 싶다

git restore --source=(헤드 또는 커밋 해시) 파일명
특정 파일을 특정 커밋의 상태로 되돌리기


# reset했어도 희망이 있다.

git reflog


reflog는 프로젝트가 위치한 커밋이 바뀔 때마다 기록되는 내역을 보여주고
이를 사용하여 reset하기 이전 시점으로 프로젝트를 복구할 수 있습니다

그래서 내가 원하는 log의 커밋 해시코드를 찾아서
git reset --hard hascode


# Git Tag


- 특정 시점을 키워드로 저장하고 싶을 때
- 커밋에 버전 정보를 붙이고자 할 때


https://semver.org/lang/ko/ - version 관리 참고

![](https://velog.velcdn.com/images/jckim22/post/1b3a8370-f134-4cbe-996d-0f0ca2d510cc/image.png)


git tag v2.0.0
마지막 커밋에 태그 달기 (lightweight)

git tag
현존하는 태그 확인

git show v2.0.0
원하는 태그의 내용 확인

git tag -d v2.0.0
태그 삭제

git tag -a v2.0.0
마지막 커밋에 태그 달기 (annotated)
입력 후 메시지 작성 또는

git tag v2.0.0 -m '자진모리 버전'
-m 태그가 -a 태그 암시
git show v2.0.0으로 확인

git tag (태그명) (커밋 해시) -m (메시지)
원하는 커밋에 태그 달기

git tag -l 'v1.*'
원하는 패턴으로 필터링하기

git checkout v1.2.1 (익명의 브랜치로 가는 것이다 HEAD)
원하는 버전으로 체크아웃

# 원격의 태그 관리

git push (원격명) (태그명)
특정 태그 원격에 올리기

git push --delete (원격명) (태그명)
삭제

git push --tags
전부 올리기


git release은 원하는 버전을 다운로드 받을 수 있게 되어있는 github의 기능이다.
tag에서 create release 하면 된다.

# Fastforward vs 3-way merge

### Fastforward
한쪽만 커밋이 앞서 나가고 있을 때
굳이 커밋을 하나 더 만들지 않고 병합할 커밋에 합쳐버리고 썻던 브랜치를 삭제한는 방식
하지만 히스토리가 남지 않는 단점이 있음
그래서 
git merge --no-ff 병합할 브랜치명 
위처럼 하면 된다

### 3-Way-merge
양 쪽다 커밋이 앞서나가고 있을 때
어쩔 수 없이 커밋을 만들 수 밖에 없다

왜 3-wat이냐면 이 두 커밋의 뿌리인 분기점 커밋을(1way) A커밋(2way)과 B커밋(3way)에 각각 대조해서 살펴보고 뭐가 바뀐지 인식한 뒤 어떤게 충돌이 나는지 확인하기 때문이다.

# Cherry pick(다른 브랜치의 원하는 커밋만 가져오기)

![](https://velog.velcdn.com/images/jckim22/post/58eb8397-8edc-4e36-b3e7-1561919da9c9/image.png)

>소스트리로 보면 다른 것들이 root 브랜치에 달려있는 것처럼 보일 수 있는데 털실 처럼 root 히스토리를 꽉 잡아 당기면 main을 포함한 다른 커밋들이 옆에 달려있는 것처럼 되기 때문에 같은 그림이다.

git cherry-pick (체리의 해시)
(특정 커밋만 복제해서 가져오는 것이라 원본은 그대로 있다)

# 다른 가지의 잔가지만 가져오기
main에서 어떤 보조기능 브랜치를 2개를 만들었는데 어떤 기능만 쓰게 되어서 한 잔가지만 가져오게 되는 상황이다.

git rebase --onto (도착 브랜치) (출발 브랜치) (이동할 브랜치)
git rebase --onto main fruit citrus
이렇게 하면 citrus가 main branch로 옮겨 붙는다
이제 main HEAD를 옮겨 주자
git switch main
git merge citrus


# 다른 가지의 마디들을 묶어서 가져오기
root 브랜치의 마디들을 하나로 묶어 main 브랜치로 가져오기 (한 커밋으로)

git merge --squash (대상 브랜치)
변경사항들 스테이지 되어 있음
git commit 후 메시지 입력

# 협업을 위한 브랜치 활용법
### Git flow

https://nvie.com/posts/a-successful-git-branching-model/

사용되는 브랜치들
![](https://velog.velcdn.com/images/jckim22/post/082535ae-e2ed-4e45-a7ab-5743272be45c/image.png)

# Git log 활용
각 커밋마다의 변경사항 함께 보기
```
git log -p
```

최근 n개 커밋만 보기


```
git log -(갯수)
```
통계와 함께 보기

```
git log --stat
```
한 줄로 보기

```
git log --oneline
```
변경사항 내 단어 검색

```
git log -S (검색어)
```
커밋 메시지로 검색


```
git log --grep (검색어)
```
자주 사용되는 그래프 로그 보기

```
git log --all --decorate --oneline --graph
```
포맷된 로그 보기

```
git log --graph --all --pretty=format:'%C(yellow) %h  %C(reset)%C(blue)%ad%C(reset) : %C(white)%s %C(bold green)-- %an%C(reset) %C(bold red)%d%C(reset)' --date=short
```
[포메팅 옵션들 살펴보기](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%EC%BB%A4%EB%B0%8B-%ED%9E%88%EC%8A%A4%ED%86%A0%EB%A6%AC-%EC%A1%B0%ED%9A%8C%ED%95%98%EA%B8%B0#pretty_format)

# git diff(차이 살펴보기)

워키 디렉토리 변경사항 확인
git diff

스테이지 확인
git diff --staged

커밋간의 차이 확인
git diff (커밋1) (커밋2)

브랜치간의 차이 확인
git diff (브랜치1) (브랜치2)

# git blame(누가 코딩했는지 알아내기)
파일의 부분별로 작성자 확인하기
git blame (파일명)

특정 부분 지정해서 작성자 확인하기
git blame -L (시작줄) (끝줄, 또는 +줄수) (파일명)


# git bisect(오류가 발생한 시점 찾아내기)
이진 탐색 시작
git bisect start

오류발생 지점임을 표시
git bisect bad

의심 지점으로 이동
git checkout (해당 커밋 해시)

오류 발생 않을 시 양호함 표시
git bisect good


♻️ 원인을 찾을 때까지 반복
git bisect good/bad

이진 탐색 종료
git bisect reset


# Git Hooks(자동화)


📁 Git Hooks 폴더 보기
프로젝트 폴더 내 .git > hooks 폴더 확인

파일 끝에 .sample을 없애면 훅 실행파일이 됨

### gitmoji-cli로 활용예 보기

https://github.com/carloscuesta/gitmoji-cli
### 2. 프로젝트의 훅에 적용
프로젝트 폴더에서 아래 명령어 실행
gitmoji -i
- hooks 폴더에 추가된 파일 확인하기
- 프로젝트에 수정 뒤 git add ., git commit하여 진행
- 커밋 추가 뒤 push하여 GitHub에서 확인

![](https://velog.velcdn.com/images/jckim22/post/695a8941-e557-4dc1-b7bb-6b9e099d3fb3/image.png)
![](https://velog.velcdn.com/images/jckim22/post/0925e7f7-d989-482d-b5ea-d5c8d462c497/image.png)
![](https://velog.velcdn.com/images/jckim22/post/b4872e0b-322a-4a68-9e41-692f4002b6c5/image.png)

# Git Submodules

### 서브모듈
프로젝트 폴더 안에 또 다른 프로젝트가 포함될 때 사용
여러 프로젝트에 사용되는 공통모듈일 때 유용

- main-project에 서브모듈로 submodule 프로젝트 추가
main-project 디렉토리상 터미널에서 아래 명령어 실행
```
git submodule add (submodule의 GitHub 레포지토리 주소) (하위폴더명, 없을 시 생략)
```
서브모듈은 수정되어도 메인에서 관리되지 않음

- 서브모듈 업데이트
1. main-project 새로운 곳에 clone하기
2. 아래 명령어들로 서브모듈 init 후 클론
```
git submodule init (특정 서브모듈 지정시 해당 이름)
```
```
git submodule update
```
3. GitHub에서 submodule에 수정사항 커밋
main-project에서 아래 명령어로 업데이트
```
git submodule update --remote
```
서브모듈 안에 또 서브모듈이 있을 시: --recursive 추가

# GitHUb 자세히 알아보기

## Pull Request와 Issue

>체계적으로 일하기 위해(코드리뷰) 


- 1. Pull request
변경사항을 merge하기 전 리뷰를 거치기 위함

팀원들의 동의를 거친 뒤 대상 브랜치에 적용

- 풀 리퀘스트 사용해보기
새로운 브랜치 생성 후 변경사항 커밋하여 푸시
GitHub 레포지토리 페이지에서 Compare & pull request 버튼 클릭

또는 ~ branches에서 New pull request 클릭

메시지 작성 후 Create pull request 클릭

- 풀 리퀘스트 검토 후 처리하기
GitHub 레포지토리 페이지에서 Pull requests 탭 클릭
대상 풀 리퀘스트 클릭하여 내용 검토

의견이 있을 시 코멘트 달기
반려해야 할 시 Close pull request
승인할 시 Merge pull request


- Issue
버그나 문제 제보, 추가할 기능 등의 이슈 소통
라벨
마일스톤(큰 기능)
이슈번호는 브랜치나 커밋의 푸터에 달아놓는다

# 오픈소스 참여하기
Fork하고 Clone하고 작업하기
그리고 PR 보내기

# SSH로 접속하기

공개키 암호화 방식 활용
username과 토큰 사용할 필요 없음
컴퓨터 자체에 키 저장


## SSH 키 등록하기
계정의 Settings - SSH and GPG keys
해당 페이지의 가이드 참조

# GPG로 커밋에 사인하기

#### GitHub 커밋 내역 살펴보기
로컬에서 푸시한 커밋과 GitHub에서 작성한 커밋 비교
Verified : 신뢰할 만한 출처에서 커밋되었다는 인증

# Github Actions

CI/CD 
- 동종: GitLab CI/CD, BitBucket Pipelines

- 1. GitHub Actions 살펴보기
**a. github.io 페이지의 액션**
해당 레포지토리 페이지에서 Actions 탭 살펴보기
새로운 커밋 푸시한 직후 다시 살펴보기

**b. 다른 프로젝트에서 액션 추가해보기**
Actions 탭에서 액션들 살펴보고 적용해보기
Marketplace 살펴보고 적용해보기
커밋 후 pull하여 로컬에서 확인

# Tips

- 1. OctoTree
파일 트리
크롬 익스텐션

- 2. GitHub CLI
GitHub 작업 전용 CLI 툴

https://cli.github.com/ (사이트)

https://cli.github.com/manual/ (주요 명령어)


