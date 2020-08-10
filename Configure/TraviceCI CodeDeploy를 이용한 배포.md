# CI/CD
지속적 통합 `CI(Coutinuous Integration)`
지속적 배포 `CD(Continuous Deployment)`

## TravisCI
깃허브에서 제공하는 무료 CI 서비스
AWS에서도 TravisCI를 제공하지만 빌드 시간만큼 요금이 부과되는 구조임

설정방법은 TravisCI 홈페이지에 접속 하여 로그인 한 후 프로필 -> 설정 화면에서 배포 할 프로젝트를 설정한다.

``` yml
language: java
jdk:
  - openjdk8

branches:
  only: 
    - master

# Travis CI 서버의 홈
cache:
  directories:
    - '$HOME/.m2/repository'
    - '$HOME/.gradle'

script: "./gradlew clean build"

before_deploy:
  - zip -r springboot-on-aws-with-travisCI *
  - mkdir -p deploy
  - mv springboot-on-aws-with-travisCI.zip deploy/springboot-on-aws-with-travisCI.zip

deploy:
  - provider: s3
    access_key_id: $AWS_ACCESS_KEY
    secret_access_key: $AWS_SECRET_KEY
    bucket: springboot-on-aws-with-travisci
    region: ap-northeast-2
    skip_cleanup: true
    acl: private
    local_dir: deploy
    wait-until-deployed: true

# CI 실행 완료 시 메일로 알람
notifications:
  email:
    recipients:
      - llsb156@gmail.com
```
1. branches
	- TravisCI를 어느 브랜치가 부시될 때 수행할지 지정
2. cache
	- 그레이들을 통해 의존성을 받게 되면 이를 해당 디렉토리에 캐시하여 같은 의존성은 다음 배포 때부터 다시 받지 않도록 설정
3. script
	- master 브랜치에 푸시되었을 때 수행하는 명령어
4. notifications
	- TravisCI 실행 완료시 자동으로 알람이 가도록 설정
5. before-deploy
	- deploy 명령어가 실행되기 이전에 수행
	- CodeDeploy는 Jar 파일을 인식하지 못하므로 Jar+ 기타 설정 파일들을 모아 압축함
6. zip -r springboot-on-aws-with-travisCI *
	- 현재 폴더의 모든것을 압축
7. deploy
	- S3로 파일 업로드 혹은 Code Deploy로 배포 등 외부 서비스와 연동될 행위들을 선언
8. local_dir:deploy
	- 앞에서 생성한 deploy 디렉토리를 지정
	- 해당 위치의 파일들만 S3로 전송

위 설정을 성공적으로 등록하면 커밋시 자동으로 빌드가 진행된다.
빌드가 완료되면 상세화면으로 들어가서 `[More Option]` -> `Settings`에 진입하여 설정 변수를 저장하여준다
이때 AWS에서 `IAM`키를 받아다 `ACCESS_KEY`, `SECURE_KEY`를 등록한다.
(AmazonS3FullAccess, AWSCodeDeployFullAcess 두개의 권한이 필요)

커밋후에 자동화 하는 프로세서는 다음과 같다.
``` graphLR
GitHub --> TravisCI
TravisCI --1.jar upload--> AWS_S3
TravisCI --2.request deploy--> AWS_CodeDeploy
AWS_S3 --3.jar upload--> AWS_CodeDeploy
AWS_CodeDeploy --4.deploy--> AWS_EC2
AWS_EC2 --> SpringBoot
Browser(사용자) --> SpringBoot
```


## IAM
- 역할
	- AWS 서비스에만 할당할 수 있는 권한
	- EC2, CodeDeploy, SQS 등..
- 사용자
	- AWS 서비스 외에 사용할 수 있는 권한
	- 로컬 PC, IDC 서버 등..

## CodeDeploy
Code Deploy와 비슷한 AWS 서비스들
- Code Commit
	- 깃허브와 같은 코드 저장소의 역할
	- 프라이빗 저장소가 있지만 깃허브에서도 프라이빗 지원으로 사용하지 않음
- Code Build
	- Travis CI와 마찬가지로 빌드용 서버
	- 멀티 모듈을 배포하는 경우 사용하기 좋음
	- 규모가 있는 서비스에서는 대부분 젠킨스/팀시티 등을 이용하여 사용할일 없음
- Code Deploy
	- AWS의 배포서비스
	- 다른 대체제가 없음
	- 오토 스케일링 그룹 배포, 블루 그린 배포, 롤링 배포, EC2 단독 배포등 많은 기능 지원

``` yml
version: 0.0
os: linux
files:
  - source:  /
    destination: /home/ubuntu/app/step2/zip
    overwrite: yes

permissions:
  - object: /
    pattern: "**"
    owner: ubuntu
    group: ubuntu

hooks:
  AfterInstall:
    - location: stop.sh # 엔진엑스와 연결되어 있지 않은 스프링 부트 종료
      timeout: 60
      runas: ubuntu
  ApplicationStart:
    - location: start.sh # 엔진엑스와 연결되어 있지 않은 Port로 새 버전의 스프링 부트 시작
      timeout: 60
      runas: ubuntu
  ValidateService:
    - location: health.sh # 새 스프링 부트 정상적 실행 확인
      timeout: 60
      runas: ubuntu
```


