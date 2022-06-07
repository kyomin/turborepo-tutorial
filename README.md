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