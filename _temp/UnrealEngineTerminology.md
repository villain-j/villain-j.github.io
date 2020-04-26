## Unreal Engine 개념 정리



### <u>Projects</u>

하나의 게임에 필요한 내용과 코드를 모두 포함하고 있는 단위.

[Working with Unreal Projects](https://docs.unrealengine.com/en-US/Engine/Basics/Projects/index.html)에서 새 프로젝트를 생성하고, 기존 프로젝트를 열고, 프로젝트를 패키징하거나 기존의 프로젝트를 템플릿으로 만드는 등의 작업에 대한 가이드를 볼 수 있다.



### <u>Objects</u>

언리얼 엔진의 기본 구성 단위를 Object라고 한다. UE4의 모든 것은 Object에서 상속된 것이다. C++에서는 모든 오브젝트들의 기본 클래스가 UObject이다. 이 UObject는 GC 기능, 언리얼 에디터를 위해 metadata 노출 기능, 게임 저장과 로딩을 위한 serialization을 지원한다.



### <u>Classes</u>

Class는 언리얼 엔진 게임 창작에 필요한 특정 Actor나 Object의 행동과 특성을 정의한다. Class는 위계적인 구조를 가지고 있으며, C++이나 블루프린트로 생성할 수 있다.

#### 	Blueprint Class

블루프린트라고 짧게 줄여서 부른다. 언리얼 에디터 안에서 코드를 사용하지 않고도 시각적으로 만들 수 있고, 콘텐츠 패키지 안에 어셋으로 저장된다. 블루프린트는 새로운 타입의 액터를 정의하는데 쓰인다.

블루프린트를 만들기 전에 부모 클래스를 지정해야 한다. 주로 Actor, Pawn, Character, PlayerController, GameMode 등을 부모 클래스로 지정한다. 부모 클래스를 지정하면 부모로부터 필요한 특징을 상속받을 수 있다.

Data-Only 블루프린트는 부모로부터 상속받은 노드 그래프 형태의 코드, 변수, 컴포넌트만을 포함하는 블루프린트 클래스이다. 이런 블루프린트는 상속받은 특징을 수정할 수 있지만, 새로운 원소를 추가할 수는 없다. 

[블루프린트 클래스 생성하기](https://docs.unrealengine.com/en-US/Engine/Blueprints/UserGuide/Types/ClassBlueprint/Creation/index.html)

[블루프린트 클래스 열기](https://docs.unrealengine.com/en-US/Engine/Blueprints/UserGuide/Types/ClassBlueprint/Opening/index.html)

[블루프린트 클래스 수정하기](https://docs.unrealengine.com/en-US/Engine/Blueprints/Editor/UIBreakdowns/ClassBPUI/index.html)

[블루프린트 에디터 소개](https://docs.unrealengine.com/en-US/Engine/Blueprints/Editor/index.html)



#### 	Gameplay Class

언리얼 엔진의 게임플레이 클래스는 헤더 파일(`.h`)과 소스 파일(`.cpp`)로 구성되어있다. 클래스 헤더 파일은 클래스와 그 멤버들을 정의하고, 소스 파일은 클래스 내 함수들을 구현해 클래스의 기능을 정의한다.

클래스 이름은 각각 무슨 클래스를 가리키는지를 표시하기 위해 첫번째 글자에 prefix를 붙이는 것이 언리얼 엔진의 작명 규칙이다. 게임플레이 클래스의 경우 스폰할 수 있는 게임플레이 객체의 경우에는 A(Actor)를 붙인다. 기타 다른 모든 게임플레이 객체의 부모는 U(UObject)로, 주로 컴포넌트에 해당한다.

###### Class 추가하기

C++ Class Wizard를 통해 새로운 클래스의 헤더 파일과 소스 파일을 설정하고, 그에 따라 게임 모듈도 업데이트 할 수 있다. 헤더 파일과 소스 파일은 자동으로 클래스 정의, 클래스 생성자, 언리얼 엔진 코드(ex. `UCLASS()` 매크로) 등을 포함한다.

###### Class Headers

언리얼 엔진 안의 게임플레이 클래스는 일반적으로 각기 고유한 클래스 헤더 파일을 가진다. 이 헤더 파일은 주로 클래스 이름에서 prefix를 뺀 이름과 `.h` 파일 확장자를 가진다. 각 헤더 파일의 제일 위에는 자동으로 생성되는 generated header 파일을 포함해야 한다. 예를 들어, `ClassName.h` 파일 제일 위에는 `#include "ClassName.generated.h"`가 있어야 한다.

###### Class Declaration

클래스 선언은 클래스 이름, 클래스의 부모 클래스, 클래스에 속한 함수와 변수를 정의한다.

```c++
UCLASS([specifier, specifier, ...], [meta(key=value, key=value, ...)])
class ClassName : public ParentName
{
    GENERATED_BODY()
}
```

여러 가지 Class Specifier와 Metadata Specifier를 통해 각 클래스의 특성을 정의할 수 있다.

###### Class Implementation

모든 게임플레이 클래스는 제대로 구현하기 위해 헤더 파일에서 `GENERATED_BODY()` 매크로를 포함해야 한다. 또한 클래스의 소스 파일은 해당 클래스의 헤더 파일을 포함해야 한다.

###### Class Constructor

```c++
UMyObject::UMyObject()
{
    // Initialize Class Default Object properties here.
}
```

가장 기본적인 형태의 UObject 생성자이다. 이 생성자는 해당 클래스의 기본값을 가진 객체(CDO, Class Default Object)를 만든다.

```c++
UMyObject::UMyObject(const FObjectInitializer& ObjectInitializer)
: Super(ObjectInitializer)
{
    // Initialize CDO properties here.
}
```

위의 생성자는 클래스의 특성을 설정할 수 있는 생성자이다. 엔진이 이미 자동으로 모든 필드를 `0`, `NULL` 등의 변수의 기본 값으로 초기화하지만, 위의 생성자들을 이용하면 엔진 내에서 `CreateNewObject`, `SpawnActor` 등으로 생성하는 객체의 기본값을 정할 수 있다.

클래스 참조값, 이름, 어셋 참조값과 같은 더 복잡한 값 설정을 하기 위해서는 `ConstructorStatics` 구조체를 정의하고 생성해 필요한 값들을 가지고 있게 해야 한다. `ConstructorStatics`는 생성자가 처음 실행될 때만 생성되고, 그 이후에는 포인터만 복사해 사용함으로써 속도를 높인다. `ConstructorHelpers`는 `ObjectBase.h`에 정의된 특별한 네임스페이스로 생성자에 공통적으로 필요한 helper template을 포함한다.



#### 	Class Creation Basics

[블루프린트로 클래스 만들기](https://docs.unrealengine.com/en-US/Gameplay/ClassCreation/BlueprintOnly/index.html)

[C++로 클래스 만들기](https://docs.unrealengine.com/en-US/Gameplay/ClassCreation/CodeOnly/index.html)

[블루프린트와 C++로 클래스 만들기](https://docs.unrealengine.com/en-US/Gameplay/ClassCreation/CodeAndBlueprints/index.html)



### <u>Actors</u>

<u>Actor는 Level 안</u>에 배치될 수 있는 모든 object를 가리킨다. Actor는 3D 구조를 지원하는 제네릭 클래스로, 게임 플레이 코드(C++/블루프린트)에 따라 생성되고(스폰되고) 삭제될 수 있다. C++에서 모든 Actor의 베이스 클래스는 AActor이다.

#### Mesh & Geometry Actor Types

###### 	StaticMeshActor

화면에서 mesh를 나타내는 기본 타입의 Actor. Geometry가 변하지 않는다는 점에서 static이라는 이름이 붙었지만 플레이 중 움직이거나 바뀔 수 있다. 주로 월드 지오메트리나 장식용 mesh로 레벨을 꾸미는데에 사용된다.

###### [Brush](https://docs.unrealengine.com/en-US/Engine/Actors/Brushes/index.html)

간단한 3D 지오메트리를 나타내는 Actor. 메모리 상 비효율적이지만 처음 게임의 간단한 프로토타이핑을 할 때 사용한다. Additive / Subtractive Brush가 있으며, Blocking, pain-causing 등의 효과를 넣을 수 있다. 

###### SkeletalMeshActor

애니메이션을 보일 수 있는 Actor, 혹은 지오메트리가 바뀔 수 있는 Actor를 가리킨다. 주로 캐릭터나 생물, 복잡한 기계를 나타낼 때 쓰인다.



#### Gameplay Actor Types

###### PlayerStart

레벨 안에 배치되는 Actor로 플레이어가 게임이 시작할 때 있을 위치를 나타낸다. 

###### Triggers

다른 객체들과의 상호작용으로 이벤트를 발생시키기 위해 사용하는 actor.

[Trigger Actors](https://docs.unrealengine.com/en-US/Engine/Actors/Triggers/index.html)

###### MatineeActor

Matinee 애니메이션 도구를 이용해 액터들의 특성을 애니메이션으로 보여줄 수 있게 해주는 actor.

[Matinee and Cinematics](https://docs.unrealengine.com/en-US/Engine/Matinee/index.html)



#### Light Actor Types

###### PointLight

###### SpotLight

###### DirectionalLight



#### Effects Actor Types

###### ParticleEmitter

연기, 불, 번쩍임 등의 이펙트를 발생시키기 위해 쓰이는 액터.

[Particle Systems](https://docs.unrealengine.com/en-US/Engine/Rendering/ParticleSystems/index.html)

[Cascade Particle Editor](https://docs.unrealengine.com/en-US/Engine/Rendering/ParticleSystems/Cascade/index.html)



#### Sound Actor Types

###### AmbientSound



#### Camera Actors

[Camera Actor Guide](https://docs.unrealengine.com/en-US/Engine/Actors/CameraActors/index.html)



### <u>Components</u>

Actor에 추가될 수 있는 기능. 독자적으로는 존재할 수 없으나 액터에 더해졌을 때 액터는 component의 기능을 사용할 수 있다.



#### Component Instancing

특정 액터 클래스에 속한 액터는 그 액터 클래스를 구성하는 모든 컴포넌트 클래스의 인스턴스를 포함한다.



#### Component Types

[AI Components](https://docs.unrealengine.com/en-US/Engine/Components/AI/index.html)

[Audio Components](https://docs.unrealengine.com/en-US/Engine/Components/Audio/index.html)

[Camera Components](https://docs.unrealengine.com/en-US/Engine/Components/Camera/index.html)

[Light Components](https://docs.unrealengine.com/en-US/Engine/Components/Lights/index.html)

[Movement Components](https://docs.unrealengine.com/en-US/Engine/Components/Movement/index.html)

[Navigation Components](https://docs.unrealengine.com/en-US/Engine/Components/Navigation/index.html)

[Paper 2D Components](https://docs.unrealengine.com/en-US/Engine/Components/Paper2D/index.html)

[Physics Components](https://docs.unrealengine.com/en-US/Engine/Components/Physics/index.html)

[Rendering Components](https://docs.unrealengine.com/en-US/Engine/Components/Rendering/index.html)

[Shape Components](https://docs.unrealengine.com/en-US/Engine/Components/Shapes/index.html)

[Skeletal Mesh Components](https://docs.unrealengine.com/en-US/Engine/Components/SkeletalMesh/index.html)

[Static Mesh Components](https://docs.unrealengine.com/en-US/Engine/Components/StaticMesh/index.html)

[Utility Components](https://docs.unrealengine.com/en-US/Engine/Components/Utility/index.html)

[Widget Components](https://docs.unrealengine.com/en-US/Engine/Components/Widget/index.html)



#### Components in Code

###### Actor Components

`UActorComponent`: 움직임, 인벤토리, 특성 관리 등 non-physical 개념과 관련된 추상적인 행동에 쓰인다. 월드에서 물리적인 위치나 방향을 갖지 않는다.

`USceneComponent`: 지오메트리로 나타나지 않는 위치 기반 행동을 지원한다. eg. spring arms, cameras, physical forces, constraints, audio

`UPrimitiveComponent`: 지오메트리로 나타나는 USceneComponent

###### Registering Components

매 프레임마다 액터 컴포넌트를 갱신하고 그에 맞게 화면을 바꾸기 위해서 엔진은 각 액터 컴포넌트를 '등록'해야 한다. 이는 액터가 스폰되면서 액터의 부속 컴포넌트들이 생성될 때 자동으로 일어난다. 그러나 플레이 도중 생성된 컴포넌트를 등록하려면 `RegisterComponent` 함수를 이용해 직접 등록해야 한다. 컴포넌트를 새로 등록하면서 `OnRegister, CreateRenderState, OnCreatePhysicsState` 등의 `UActorComponent` 함수도 실행한다.

이와 반대로 플레이 도중 컴포넌트를 삭제할 때에는 `UnregisterComponent` 함수를 이용할 수 있고, 이 또한 `OnUnregister, DestroyRenderState, OnDestroyPhysicsState` 함수가 실행된다.

###### Updating

액터 컴포넌트는 액터처럼 매 프레임을 업데이트 할 수 있다. `TickComponent` 함수를 이용하면 컴포넌트가 각 프레임마다 코드를 실행할 수 있다. 기본값은 프레임마다 업데이트하지 않는 것이지만, 컴포넌트의 생성자에서 `PrimaryComponentTick.bCanEverTick`을 `true`로 설정하면 프레임마다 업데이트하게 된다.

###### Render State

렌더링을 위해 액터 컴포넌트는 반드시 render state를 생성해야 한다. 렌더링 데이터의 변경이 필요할 경우, render state는 이를 엔진에 알린다. 이런 경우 render state가 dirty하다고 말한다. 매 프레임의 마지막에는 모든 dirty component의 렌더링을 업데이트한다. Scene Component는 기본적으로 render state를 생성하지만, Actor Component는 필요할 경우 따로 설정해주어야 한다.

###### Physics State

Render state와 마찬가지로 물리적 모델링이 필요할 경우 생성해야 하는 것. Primitive Component는 기본적으로 생성하나 나머지는 필요할 경우 따로 설정해 주어야 한다.

###### Visualization Components

에디터 내에서 작업할 때에만 시각적으로 보이는 컴포넌트.



### <u>Pawns</u>

Actor의 subclass로 인게임 아바타나 페르소나로서 기능한다. 플레이어나 게임의 AI에 의해 조종될 수 있다. Pawn이 사람이나 AI 플레이어에 의해 컨트롤되고 있을 경우 'possessed' 되었다고 말하고, 반대의 경우 'unpossessed' 되었다고 말한다.



### <u>Characters</u>

플레이어의 캐릭터로 쓰이도록 설계된 Pawn Actor이다. 플레이어의 인풋과 이족 보행의 움직임(걷기, 뛰기, 점프하기 등)의 바인딩과 collision 세팅, 플레이어의 조종을 위한 추가적인 코드를 지원한다. 



### <u>PlayerController</u>

플레이어의 입력을 받아 게임 내의 상호작용으로 바뀌게 한다. 모든 게임에는 최소한 하나의 플레이어 컨트롤러가 존재하고, 플레이어 컨트롤러는 보통 Pawn이나 Character 하나에 빙의(possess)할 수 있다. 멀티 플레이어 게임의 경우 각 클라이언트는 자신의 플레이어에 해당하는 플레이어 컨트롤러가 있고, 이를 이용해 서버와 통신한다.



### <u>AIController</u>

플레이어 컨트롤러가 Pawn에 빙의(possess)하는 것처럼 AIController는 NPC에 해당하는 Pawn에 빙의(possess)할 수 있다.



### <u>Levels</u>

게임 내에서의 하나의 공간. 슈퍼마리오 게임에서 각 스테이지마다 하나의 레벨이 있다고 생각하면 된다. Level Editor에서 여러 액터와 지오메트리를 이용해 게임 내에서의 공간을 구성할 수 있다. 언리얼 에디터에서 각 레벨은 각각의 .umap 파일로 저장되고, 이 때문에 종종 map이라고 불린다.



### <u>World</u>

로딩된 레벨의 리스트를 포함하고 있다. 레벨 스트리밍과 동적 액터 스폰을 담당한다.



### <u>GameMode</u>

서버에서 가지고 있는 데이터로 게임의 규칙을 정의한다. 게임의 규칙은 자주 바뀌지 않는 속성(게임 승리 조건, 최대/최소 플레이어의 수 등)을 포함한다. 프로젝트 세팅에서 디폴트 게임 모드를 설정할 수 있으나, 레벨 별로 덮어쓸 수 있다. 하나의 레벨에는 항상 하나의 게임 모드가 존재한다. 멀티 플레이어 게임의 경우 게임 모드는 서버에서 보관하고 각 클라이언트가 서버의 게임 모드를 복제해 사용한다.



### <u>GameState</u>

게임 스테이트는 게임 모드와 달리 게임의 현재 상태에 관한 것들을 포함한다. 매치가 시작되었는지, 게임에 참여중인 플레이어가 몇 명인지 등의 정보이다. 멀티 플레이어 게임에서 각 플레이어의 컴퓨터에는 하나의 게임 스테이트가 있고, 서버에서 모든 클라이언트의 게임 스테이트를 업데이트한다.

[게임 모드와 게임 스테이트](https://docs.unrealengine.com/ko/Gameplay/Framework/GameMode/index.html)



### <u>PlayerState</u>

플레이어 스테이트는 현재 게임에 참여 중인 참여자(PC, NPC 모두 해당)의 상태를 나타낸다. 각 참여자의 점수와 체력 상태와 같은 정보이다. 멀티 플레이어 게임에서 각 플레이어의 정보는 모든 클라이언트와 서버에 존재하고, 서버에서 정보를 복제해 와서 동기화된 상태를 유지한다.