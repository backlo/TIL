# Gradle

## Gradle 이란?

그래들이란 어플리케이션을 유연하게 실행 시켜줄 빌드 자동화 도구입니다. 그래들은 다음과 같은 특징을 지니고 있다고 합니다.

> 1. 고성능 : 그래들은 I/O 작업만 실행하여 불필요한 작업을 방지합니다. 
> 2. JVM : 그래들은 JVM에서 실행되며 JDK가 필요로 합니다. 이는 자바 개발자들에게 익숙하고 좋은 도구라고 생각합니다. 그렇다고 JVM에만 국한되지 않으며 다른 네이티브 프로젝트에도 빌드를 지원합니다.
> 3. 규약 : 메이븐과 다르게 일반적인 유형의 프로젝트를 쉽게 빌드를 할 수 있습니다. 규칙 재정의, Task 추가, 규칙 기반 빌드를 통해 사용자가 원하는대로 수행할 수 있습니다.
> 4. IDE 지원 : 인텔리제이, 안드로이드 스튜디오, 이클립스 등 다양한 IDE를 지원합니다.
> 5. 통찰력 : 빌드 스캔을 통해 빌드 된 정보를 좀 더 정확하게 볼 수 있습니다. 그리고 다른사람과 공유하여 문제를 빠르게 해결 할 수 있습니다.

그래들은 사실 메이븐의 장점만 모아 좀 더 업그레이드 된 빌드 도구라 생각하면 됩니다. 그래들 공식 문서에서 메이븐과 그래들의 성능 비교를 한 글이 있습니다.

다음은 그래들을 사용하려면 다음 핵심 원칙 5가지입니다.

1. Gradle은 범용 빌드 도구 입니다.
   * 그래들은 빌드하려는 작업이나 수행 방법을 가정하지 않기 때문에 어떤 소프트웨어든 빌드를 할 수 있습니다.
   * 플러그인을 통해 규칙 계층과 미리 빌드된 기능을 사용하여 프로젝트를 쉽게 빌드 할 수 있습니다.
   * 사용자 지정 플러그인을 만들고 게시하여 고유 한 규칙을 캡슐화하고 빌드 시킬 수 있습니다.
2. 핵심 모델 작업을 기반으로 합니다.
   * 빌드 작업 집합을 구성하고 종속성을 기반으로 함께 연결하는 DAG 작업으로 모델링합니다. 
   * 작업 그래프가 생성되면 Gradle은 어떤 작업을 어떤 순서로 실행해야하는지 결정한 다음 실행합니다.
   * 서로 연결된 잡엄과 함께 플러그인과 자체 빌드 스크립트로 정의 할 수 있습니다.
   * 작업은 다음과 같이 구성됩니다. (선택 사항)
     1. 작업 - 파일 복사, 소스 컴파일과 같은 작업을 수행
     2. 입력 - 작업이 실행되는 파일 혹은 디렉토리
     3. 출력 - 작업이 생성, 수정하는 파일 혹은 디렉토리
3. Gradle를 빌드 할시 다음과 같이 3가지의 고정 단계가 있습니다.
   * 초기화 - 빌드 환경을 설정
   * 구성 - 빌드에 대한 작업 그래프를 구성후 실행하려는 작업에 따라 실행해야하는 작업과 순서를 결정
   * 실행 - 선택한 작업 실행
   * 이 과정들을 통해 그래들의 생명주기를 알 수 있습니다. 
4. Gradle은 여러 방면으로 확장이 가능합니다.
   * 대부분 빌드에는 사용자가 몇가지 옵션을 추가해 주어야 합니다.
   * 사용자 지정 작업 유형 - 작업을 만들어 자신만의 빌드 작업을 수행 시키면 됩니다.
   * 사용자 지정 작업 동작 - ``Task.doFirst()``, ``Task.doLast()``의 메소드를 통해 작업 전후를 관리 할 수 있습니다.
   * 프로젝트 및 작업에 대한 추가 속성 - 추가 속성으로 프로젝트를 설정 할 수 있습니다. (ex - ``ext`` )
   * 사용자 지정 규칙 - 규칙을 통해 빌드의 단순화를 만들 수 있습니다.
   * 커스텀 모델 - 작업, 파일 및 종속성 구성뿐만 아니라 빌드의 새로운 개념을 도입 할 수 있습니다. 이로써 빌드 프로세스의 적절한 모델링을 통해 효율성, 용이성을 향상 시킬 수 있습니다.
5. 빌드 스크립트는 API에 잘 작동합니다.