# Git 자주 사용하는 명령어

- ### Git Project Download

  ```bash
  # git init
  # git clone <저장소 URL>
  ```

  

- ### Git Project Upload

  ```bash
  # git init
  # git add .
  # git commit -m "커밋 메세지 입력"
  # git remote add origin <저장소 URL>
  # git push origin master 
  ```

  

- ### 주요 명령어

  ```git
  - git 저장소 생성
  # git init
  
  - git 저장소 복제
  # git clone <저장소 URL>
  
  - git 저장소 추가
  # git remote add origin <저장소 URL>
  
  - git 저장소 삭제
  # git remote rm <저장소 URL>
  
  - git 새로운 파일을 추가
  # git add <파일>
  
  - git 모든 원격 브랜치 목록 보기
  # git branch -r
  
  - git 브랜치 생성
  # git branch <브랜치 이름>
  
  - git 현재 파일들 상태 보기
  # git status
  
  - git 로그 보기
  # git log
  ```
  
### GitHub Error
###remote: Repository not found. <br>
fatal: repository 'https://github.com/userName/repository.git/' not found <br>
아래 명령어 사용 <br>git remote set-url origin https://userName@github.com/userName/repository.git
