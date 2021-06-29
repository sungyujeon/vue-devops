# Vue CLI 프로젝트 기반 DevOps 개발환경 실습
> 목표
>
> - Vue 프로젝트 생성 및 로컬 실행 확인
> - GitHub에 코드 Push 및 Pages에 수동 배포
> - GitHub Actions workflow로 배포 자동화
> - 코드 수정 및 테스트 실패로 인한 자동 배포 실패 확인



## Vue 프로젝트 생성 및 로컬 실행 확인

##### Yarn으로 설치하기

[Yarn 설치 경로](https://classic.yarnpkg.com/en/docs/install)

- Vue CLI 설치

  ```bash
  $ yarn global add @vue/cli
  $ vue -V
  @vue/cli 4.5.13
  ```

> yarn vs npm
>
> - Package 병렬 설치
>   - npm은 여러 package를 설치할 때 각각의 package가 설치된 후 다음이 설치되지만, yarn은 병렬로 처리하여 performance와 speed 증가
> - 자동 Lock file 생성
>   - npm, yarn 모두 package.json에 버전을 명시하고 의존성을 추적 관리함
>   - yarn은 자동으로 yarn.lock 파일 생성하지만, npm은 `npm shrinkwrap` 커맨드로 생성
> - 보안
>   - npm은 다른 package를 즉시 포함시킬 수 있는 코드를 자동으로 시작하여 보안에 취약하지만, yarn은 yarn.lock 또는 package.json에 있는 파일만 설치해 더 안전하다고 여겨짐



##### Vue 프로젝트 생성

- vue 프로젝트 생성(manually, Unit testing 추가 > 3.x > ESLink + Prettier > Lint on save > Jest > in dedicated config files)

  ```bash
  $ vue create vue-devops
  ```



## GitHub에 코드 Push 및 Pages에 수동 배포

- Create a GitHub repo

- Push a code to GitHub repo

- Github pages로 배포하기 위한 라이브러리 추가 및 배포에 필요한 정보 등록

  ```bash
  $ yarn add gh-pages -D
  ```

  ```json
  // /pacakge.json
  // predeploy, deploy, clean 추가
  {
    "scripts": {
      // "serve": "vue-cli-service serve",
      // "build": "vue-cli-service build",
      "predeploy": "vue-cli-service build",
      "deploy": "gh-pages -d dist",
      "clean": "gh-pages-clean",
      // "test:unit": "vue-cli-service test:unit",
      // "lint": "vue-cli-service lint"
    },
  }
  ```

- 배포용 publicPath 설정

  - github.io 페이지로 만들 경우에는 publicPath가 루트 URL로 잡혀서 별도 설정이 필요 없지만, 해당 페이지가 있는 경우에 대비해 추가 path를 설정해줌

    ```js
    // vue.config.js 파일 생성
    module.exports = }
    	publicPath: '/vue-devops/',
      outputDir: 'dist'
    }
    ```

- build

  - 빌드 시 정적 파일을 원격저장소의 gh-pages 브랜치를 생성해 푸시

    ```bash
    $ yarn deploy
    ```

  - github pages가 hosting 하고 있는 주소로 접속 가능

    `https://<github_id>.github.io/<repository>/`

    

## GitHub Actions workflow로 배포 자동화

> 위 과정처럼 수동으로 deploy하여 Github Pages에 수동으로 빌드된 정적파일을 github에서 호스팅 해줄 수도 있지만, github Actions를 통해 개발자가 새 소스 코드를 push하거나 pull request 같은 이벤트에 반응하여 트리거하도록 구성할 수 있음

##### Simple workflow

- Github Actions 메뉴의 simple workflow 파일을 커밋

- /.github/worflows/deploy.yml

  - jobs.deploy.steps: 실행할 sequence 명시
  - Actions를 통해 triggered 시 해당 step의 작업 순차 시행

  ```yaml
  # /.github/workflows/deploy.yml
  
  # A workflow run is made up of one or more jobs that can run sequentially or in parallel
  jobs:
    deploy:
      runs-on: ubuntu-latest
  
      steps:
        - name: Checkout source code
          uses: actions/checkout@master
        
        - name: Set up Node.js
          uses: actions/setup-node@master
          with:
            node-version: 14.x
        
        - name: Install dependencies
          run: yarn install
  
        - name: Test unit
          run: yarn test:unit
  
        - name: Build page
          run: yarn build
          env:
            NODE_ENV: production
  
        - name: Deploy to gh-pages
          uses: peaceiris/actions-gh-pages@v3
          with:
            github_token: ${{ secrets.GITHUB_TOKEN }}
            publish_dir: ./dist
  ```

  

## 코드 수정 및 테스트 실패로 인한 자동 배포 실패 확인

> 빌드 자동화 시 테스트를 통과하지 못한 코드가 반영되지 않도록 테스트 유닛 script를 추가

```yaml
jobs:
	deploy:
		# ...
		steps:
			# ...
			- name: Test unit
			  run: yarn test:unit
```
