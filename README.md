# Turborepo
Turborepo는 대규모 모노레포 프로젝트 관리에서 오는 피로감과 부수적인 툴링에 대한 부담을 줄이면서, Google이나 Facebook과 같은 큰 기업에서 사용하는 수준의 개발 경험을 주는 데 포커싱한, Vercel에서 개발 및 운영하고 있는 JavaScript/TypeScript를 위한 모노레포 빌드 시스템이다.   
   
Turborepo는 증분 빌드(incremental build), 원격 캐싱, 병렬 처리 기법을 통해 빌드 성능을 끌어올리고, Pipeline의 쉬운 설정과 profiling, trace 등 다양한 시각화 기능을 제공해 관리 편의성을 높인 것이 특징이다.   
   
그리고 기존의 모노레포 프로젝트에 점진적으로 Turborepo를 적용할 수 있으며, 쉽게 Turborepo로 구성된 새 모노레포 프로젝트를 생성할 수 있도록 스캐폴딩 기능을 제공한다.   
또한 패키지 매니저로 yarn, npm, pnpm와 함께 잘 동작하므로 프로젝트 상황에 맞게 선택해 사용할 수 있다.   
   
# create-turbo 구성 확인하기
아래의 명령어를 통해 새 모노레포 프로젝트를 생성해 본다.
   
      npx create-turbo@latest <PACKAGE_NAME>   
   
![터보레포 폴더 구조](https://user-images.githubusercontent.com/46395776/172334584-4c234556-ae72-4f34-b9d9-b298a9922cb5.png)   
   
위 그림과 같이 /app 하위에 docs, web 앱과   
공통 패키지인 /packages 하위의 ui, config 패키지가 있는 프로젝트가 생성된다.   
이 구조는 일반적인 모노레포 프로젝트 구조와 유사해 쉽게 Turborepo를 사용할 수 있도록 한다.   
   
Vercel사에서 운영하는 프로젝트답게 앱은 기본적으로 Next.js(react)로 설정된다.   
그 외에 Express, Remix, pnpm 등 다양한 기술 스택으로 이루어 구성할 수도 있다.   
   
# 캐싱
이미 계산한 것들은 다시는 계산하지 않는다는 핵심 컨셉 아래 캐싱이 이루어지고 있다.   
이전에 실행한 파일 및 로그를 캐싱해, 만약 현재 실행 태스크에 이미 완료한 작업이 있다면 이것을 건너뛰기 때문에 작업 실행 시간을 효과적으로 줄일 수 있다.   
   
현 프로젝트의 루트 경로에서 빌드를 진행해 보자.
   
      yarn run build   
   
![캐시 전](https://user-images.githubusercontent.com/46395776/172342296-4157ba5a-8849-4c63-bdcc-478f7da2eaf0.png)   
   
그 다음 위 명령어로 다시 빌드를 하면 다음과 같이 시간이 대폭 줄어든 것을 확인할 수 있다.   
   
![캐시 후](https://user-images.githubusercontent.com/46395776/172342450-668dd464-b782-4dc0-9a79-5818104ca5a1.png)   
   
그렇다면 이 캐시들은 어떻게 저장되고 있을까?   
   
빌드를 하면 /app 하위 패키지들의 .turbo 디렉터리에 각각 로그 파일이 생기고 이 로그에는 해당 빌드에 대한 hash가 저장된다.   
(Turborepo는 빌드해야 할 항목을 파악하기 위해 타임스탬프를 확인하고 활용하는 대신 파일의 내용에 따른 hash 정책을 사용한다.)   
이 hash 값들은 아래 이미지와 같이 node_modules/.cache/turbo 하위에 hash로 구분된 스냅샷 폴더와 매칭된다.   
   
![빌드 로그](https://user-images.githubusercontent.com/46395776/172344531-3db521f3-e982-46d7-80f3-d9be9ece2396.png)
![캐시 파일](https://user-images.githubusercontent.com/46395776/172344580-510bf547-fc40-48c0-9226-fd08d69ddd98.png)   
   
이렇게 모든 작업에 대한 캐싱을 진행하기 때문에 변화된 부분만 재빌드하고 나머지는 캐싱한 것을 사용하면서 Turborepo는 작업 속도를 높일 수 있다.   
   
# 원격 캐싱
위에서 설명한 캐시는 로컬 환경에서만 유효하다는 한계가 있다.   
그러나 Turborepo는 이 한계를 극복하는 `원격 캐싱`이라는 강력한 기능을 제공한다.   
   
더 빠른 빌드를 위해 캐시를 클라우드에 올려서 팀원 및 CI/CD 시스템이 공유해 사용할 수 있어 대규모 프로젝트에서 다수의 팀이 협업하는 환경에서 개발 효율성을 높일 수 있다.   
   
실제로 원격 캐싱을 한 상태에서 빌드를 실행하면 "Remote computation caching enable" 문구와 함께 빌드 시간이 9초에서 3초로 비약적으로 감소하는 것을 확인해볼 수 있다.   
다만 이 기능은 현재 experimental(beta) 상태이며 Vercel 클라우드를 활용하기 때문에 Vercel의 계정이 필요하다.   
   
# Pipeline Package Task
Pipeline은 각 패키지의 package.json 스크립트(태스크) 간 작업 관계를 정의한다.   
이를 통해 새로 들어온 개발자도 작업 관계를 쉽게 이해할 수 있다.   
   
Pipeline 설정은 Turborepo turbo.json에서 확인할 수 있다.   
(package.json에도 설정 가능하나 turbo.json에 설정하도록 권장하고 있다.)   
   
<img width="1120" alt="터보 설정" src="https://user-images.githubusercontent.com/46395776/172350609-119741fe-a8dc-4025-bf6a-972ccc4779c2.png">   
   
위 설정에서 dependOn은 pipeline key에 해당하는 작업이 의존하는 작업을 의미한다.   
이 부분에 접두사 ^가 붙으면 각 패키지에 있는 작업을 의미한다.   
   
lint는 의존 관계와 상관없이 언제나 수행될 수 있으며,   
deploy는 Pipeline에서 build, test, lint가 선행되어야 수행됨을 의미한다.   
   
Pipeline으로 인해 Turborepo의 스케줄러는 기존의 모노레포에 비해 성능이 강력하다.   
한 번에 한 태스크씩 수행되는 기존 방식과 다르게 의존성이 없는 작업은 유휴 CPU에서 실행되기 때문에 빌드 시간이 긴 경우에 작업 성능이 비약적으로 높아진다.   
   
다음 그림을 살펴보자.   
   
![터보레포 파이프라인](https://user-images.githubusercontent.com/46395776/172351744-fb59e6b2-75a2-401f-a7b1-1488b972a1e5.png)   
   
A, B, C 세 개의 패키지로 구성된 프로젝트가 있다.   
A와 C는 B에 의존한다.   
   
lint, build, test, deploy 작업을 순차적으로 수행한다고 했을 때 Lerna의 경우에 A, C 패키지는 B의 build가 끝날 때까지 build를 수행할 수 없다.   
   
이에 반해 Turborepo는 build에 의존 관계가 없는 test 작업이 병렬적으로 수행된다.   
   
# 프로젝트 실행
프로젝트 루트 경로에서
   
      yarn install
      yarn run build
      yarn run dev   
   
를 순차적으로 실행하여 세 작업이 모두 정상적으로 이뤄지면 된다.   
localhost:3000으로 들어가면 client 페이지가,   
localhost:3001로 들어가면 admin 페이지가 실행됨을 확인할 수 있다.   
   
# 결론
모노레포 사용의 목적이 `단순히 공통 요소를 공유`하는 것이라면 Yarn으로 workspace를 구성해서 개발을 진행하는 것이 좋다.   
   
하지만 단순한 코드의 공유 외에 `패키지 간 의존성 관리 및 테스트, 배포` 등의 작업에 대한 더 나은 무언가를 필요로 한다면 Lerna를 함께 사용해보는 것도 좋은 선택지가 될 것이다.   
   
그리고 프로젝트가 성장하면서 개발, 관리에 유용한 더 많은 기능이 필요하다면 Nx와 Turborepo를 고려해보면 좋을 것 같다.   
Nx와 Turborepo는 모두 `캐싱` 기능을 지원해서 빌드 측면에서 우수한 속도를 자랑하고, `의존성 그래프 시각화` 기능을 통해 프로젝트의 구성 요소들이 서로 어떤 의존관계를 갖고있는 지 한눈에 파악이 가능하도록 해준다.