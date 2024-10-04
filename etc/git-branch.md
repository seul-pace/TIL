# Git 브랜치 전략에 관하여

**git에서 제공하는 브랜치 전략 (변경 이력 관리 전략)**

기술 블로그를 읽다가 브랜치 전략에 관한 글을 보았다.

> 당근페이) 매일 배포하는 팀이 되는 여정(1) - 브랜치 전략 개선하기
https://medium.com/daangn/%EB%A7%A4%EC%9D%BC-%EB%B0%B0%ED%8F%AC%ED%95%98%EB%8A%94-%ED%8C%80%EC%9D%B4-%EB%90%98%EB%8A%94-%EC%97%AC%EC%A0%95-1-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5-%EA%B0%9C%EC%84%A0%ED%95%98%EA%B8%B0-1a1df85b2cff

브랜치 전략에 대해서 이야기 하는데 git 전략에 대해 글이 잘 읽히지 않았다.   
단순한 개념이라고 생각했는데 이해가 되지 않으면 공부해야지...

Git Flow 라는 단어를 알고 있었지만 이게 브랜치 전략인지 몰랐다. 그냥 단어인 줄?   
깊게는 어렵고, 기본만 알아보고자 한다.

---

# 🤔 Git Flow
가장 유명한 전략.   
아래 5개의 브랜치를 이용한다.   
내가 아는 것과 사용법이 달라서 정리하게 됐다...   

### 브랜치의 종류는 5개
1. main (master)
   1. 서비스를 직접 배포하는 브랜치
2. feature 
   1. 각 기능 별 개발 브랜치 
3. develop 
   1. feature에서 개발된 내용이 저장되는 브랜치 
4. release 
   1. 배포를 하기 전 내용을 QA 하기 위한 브랜치 
5. hotfix 
   1. main 브랜치 배포 후 버그 발생 시 빠른 수정을 위한 브랜치

(다른 글에서는 support 라는 브랜치도 설명한다. support는 버전 호환성 문제를 처리하기 위한 브랜치)

### 순서
1. master -> develop 브랜치 생성
2. develop -> feature 특정 기능 개발을 위해 브랜치 생성 
   1. feature에서 개발 진행 
3. feature -> develop (push or pull request) -> merge 
   1. feature 삭제
4. develop -> release 생성 
   1. QA 진행
   2. 버그 확인 및 수정
5. release -> develop
6. release -> master
7. master deploy

### 내가 아는 것과 다르다고 느낀 점
1. release 브랜치가 작업이 저장된 develop에서 분리됨
2. 한번 생성된 develop은 쭉 가는데, release만 매번 새롭게 생성 됨
3. feature에서 작업 후 develop에 적용할 때도 PR을 날린다...!!

### 재밌다고 생각한 부분
release를 만들어서 QA 전에 개발 요청이 들어왔을 때,   
develop에는 이번 배포에 참여되어서는 안 되는 작업이 이미 merge 되었다면  
release에서 feature 브랜치를 딴다!  
-> 어차피 release 끝나면 develop에 반영됨

# 🤔 Github Flow
git flow가 github에서 사용하기 복잡하여 나온 전략   
(github에는 릴리즈라는 개념이 없는 듯 하다...)   
단일 브랜치 사용, 신속함   

### 특징
1. master(main) 브랜치와 작업 브랜치 뿐 
   1. develop, feature, release가 없다 (hotfix도) 그냥 하나의 작업 브랜치로 생각한다 -> 순서는 있을 듯...
2. master로 merge 되면 즉시 배포 
   1. hubot 등을 이용하여 자동으로 배포 되도록 설정 (안 되면 바로바로 수동으로 해야지)
3. 지속적인 배포 진행 필수

### 순서
1. Branch 생성
2. 작업 완료 후 pull request
3. 코드 리뷰
4. merge
5. deploy

브랜치의 명명 규칙이나, 특정 역할 브랜치의 존재 유무에 대해서는 글마다 다르다.
그러나 master(main) 브랜치, 작업 브랜치가 존재한다는 사실은 다르지 않다.

# 🤔 Gitlab Flow
github flow가 너무 간단해서 조금 복잡한 이슈를 보완하기 위해 나온 전략

### 특징
1. Production 브랜치 존재 
   1. Git Flow의 master와 역할이 같음
   2. master -> Production merge
   3. 배포만 담당
2. Pre-production 브랜치가 있는 전략도 있음
3. master 브랜치가 Git flow의 develop과 동일한 역할

### 순서
1. master -> feature 브랜치 생성 
   1. feature에서 개발 진행
2. feature -> master merge 
   1. master에서 테스트 진행
3. master -> production merge 
   1. staging이 필요하다면 pre-production 브랜치로 머지 
   2. pre-production에서 배포해서 통합 테스트 등 진행
4. production deploy

Git Flow와 Github Flow의 중간 정도의 복잡도를 갖고 있는 듯 하다.

# 왜 이렇게까지 분류하는 걸까..?
블로그들을 쭉 읽어 보았을 때

1. 1개의 레퍼지토리에서 충돌을 방지하기 위해 
2. 배포 시기가 다르기 때문에 
3. 위의 2개의 이유 등으로 효율적으로 사용하기 위해서

---

# 당근페이 제레미 님의 글을 다시 읽어보기

느낀 점

1. 당근페이는 Git Flow 전략을 사용하면서 master 브랜치를 Production으로 명명함 
   1. 더불어 항상 배포해도 상관 없는 Stable 브랜치를 따로 갖고 있음
2. Trunk-Based Development 
   1. 트렁크라는 브랜치에서 다같이 작업하고 빠른 시일 내에 완료해서 merge 하는 듯? 
   2. 추후 더 많은 공부가 필요하다
3. 페이인데 github flow를 채택하다니 너무 대단하다
4. Feature Flag가 정확히 무엇일지?

이제야 글이 이해 된다...

<hr/>

> 참고 URL

### Git Flow
https://puleugo.tistory.com/107   
https://velog.io/@seongwon97/Git-Git-Flow-%EA%B0%9C%EB%85%90-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0   
https://m.blog.naver.com/adamdoha/222712473510 (추천!)   
예시 있음   
https://uroa.tistory.com/106

### Github Flow
https://brownbears.tistory.com/603

### Gitlab Flow
https://brownbears.tistory.com/605

### 총 정리
https://velog.io/@gmlstjq123/Git-Flow-VS-Github-Flow (추천)
- Git Flow, Github Flow 장단점 분석
- Git 브랜치 전략 필요성   

https://wiki.yowu.dev/ko/dev/Git/about-git-github-gitlab-flow

