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