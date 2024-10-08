# 세그멘테이션 [^Jo-Sehyun]

[^Jo-Sehyun]: [조세현](https://github.com/Jo-Sehyun)

베이스와 바운드 레지스터를 사용하면 운영체제는 프로세스를 물리 메모리의 다른 부분으로 쉽게 재배치할 수 있습니다. 그러나 이러한 형태의 주소 공간에서 재미있는 사실은 스택과 힙 사이에 사용되지 않는 큰 공간이 존재한다는 것입니다. 이 공간은 주소 공간을 물리 메모리에 재배치할 때 물리 메모리를 차지하게 됩니다. 베이스와 바운드 레지스터 방식은 이러한 메모리 낭비가 심할 수 있으며, 또한, 주소 공간이 물리 메모리보다 큰 경우 실행이 어려워질 수 있습니다. 이러한 측면에서 볼 때, 베이스와 바운드 방식은 유연성이 부족한 것으로 여겨집니다.

> **핵심 질문: 대용량 주소 공간을 어떻게 지원하는가**

> 스택과 힙 사이에 잠재적으로 큰 빈 영역이 존재하는 주소 공간을 어떻게 지원할까요?

> 작은 주소 공간에서는 낭비는 그렇게 심각하지 않지만 크기가 4 GB인 32비트 주소 공간을 생각해보면 통상 프로그램은 단지 수 메가바이트만 사용함에도 불구하고 주소 공간 전체가 메모리에 탑재되어야 합니다.

## 세그멘테이션: 베이스/바운드(base/bound)의 일반화

위 문제를 해결하기 위한 아이디어 중 하나가 바로 세그멘테이션(segmentation)입니다.

세그멘테이션은 오랜 역사를 가진 메모리 관리 기법 중 하나로, 최소 1960년대 초까지 거슬러 올라갈 수 있습니다. 이 방법은 메모리의 효율적인 사용과 프로그램의 보다 유연한 배치를 가능하게 합니다.

세그멘테이션은 주소 공간을 논리적으로 분할하여 각각의 세그멘트에 대해 별도의 베이스(base)와 바운드(bound) 쌍을 할당하여 메모리 관리 장치(MMU)에 저장하는 방식입니다. 이러한 세그멘트는 특정 길이를 가지는 연속적인 주소 공간을 나타내며, 일반적으로 코드, 스택, 및 힙 등과 같이 프로그램이나 데이터의 논리적인 부분을 나타냅니다.

- 세그멘트(segment): 프로그램의 논리적인 구성 요소로, 연속적인 주소 공간을 차지하는 코드, 데이터, 스택 등을 나타냅니다. 각 세그멘트는 고유한 이름, 크기, 보호 속성 등을 가집니다.

세그멘테이션을 사용하면 운영체제는 각 세그멘트를 메모리에 별도로 배치함으로써 프로그램이나 데이터를 물리 메모리의 다양한 위치에 할당할 수 있습니다. 또한 사용되지 않는 가상 주소 공간이 물리 메모리를 차지하는 것을 방지할 수 있습니다.

![19 1](https://github.com/entelecheia/os-2024/assets/162019986/987719c3-0cd3-44a2-ad6a-9432e4a3dbb7)

예를 들어, 그림 19.1의 주소 공간을 물리 메모리에 배치하려고 합니다. 각 세그멘트의 베이스와 바운드 쌍을 이용하여 세그멘트들을 독립적으로 물리 메모리에 배치할 수 있습니다.

![19 2](https://github.com/entelecheia/os-2024/assets/162019986/e80f1558-e888-42a5-bb46-7d0fa6baf8c7)

그림 19.2를 보면 64 KB의 물리 메모리에 3개의 세그멘트와 운영체제용으로 예약된 16 KB 영역이 존재합니다. 그림에서 볼 수 있듯이, 사용 중인 메모리에만 물리 공간이 할당됩니다. 이러한 구조는 사용되지 않은 영역이 많은 대형 주소 공간(드문드문 사용되는 주소 공간(sparse address space)이라고도 부름)을 수용할 수 있습니다.

- 드문드문 사용되는 주소 공간(sparse address space): 프로그램의 주소 공간 중 실제로 사용되는 부분이 드문드문 흩어져 있는 상태를 말합니다. 이는 주소 공간의 상당 부분이 사용되지 않고 비어 있음을 의미합니다.

세그멘트 지원을 위한 MMU 하드웨어 구조는 예상한 것과 같습니다. 이 예의 경우 3쌍의 베이스와 바운드 레지스터 집합이 필요합니다.

![19 3](https://github.com/entelecheia/os-2024/assets/162019986/c7c90c74-681b-47bb-9f41-0b317f7b32ef)

그림 19.3은 앞의 예에 해당하는 각 레지스터의 값을 보여줍니다. 각 바운드 레지스터는 세그멘트의 크기를 저장합니다. 그림에서 코드 세그멘트가 물리 주소 32 KB에 배치되고 크기는 2 KB이며, 힙 세그멘트가 34 KB에 배치되고 역시 크기는 2 KB라는 것을 알 수 있습니다.

> **세그멘트 폴트의 여담**

> 용어 Segment Fault는 세그멘트 사용 시스템에서 불법적인 주소 접근 시 발생합니다.

> 이 용어는 세그멘트에 대한 지원이 전혀 없는 컴퓨터에서도 여전히 사용되는데, 이 경우에는 코드의 오류 원인을 알 수 없습니다.

위 그림 19.1의 주소 공간을 사용하여 주소 변환을 살펴봅시다. 먼저, 가상 주소 100번지가 참조된다고 가정해봅시다. 이 주소는 코드 세그멘트에 속합니다. 참조가 발생하면, 하드웨어는 해당 세그멘트의 베이스 값에 가상 주소의 오프셋을 더합니다. 이 경우, 100을 더하여 물리 주소는 100 + 32 KB로 계산되어 32868이 됩니다. 그 후, 하드웨어는 이 주소가 범위 내에 있는지 확인하고 (100은 2 KB보다 작으므로), 범위 내에 있다면 물리 메모리 주소 32868을 읽습니다.

- 오프셋(offset): 세그멘트의 시작 주소로부터 특정 주소까지의 거리를 나타내는 값입니다. 오프셋은 세그멘트 내에서의 상대적인 위치를 나타냅니다.

다음으로 가상 주소 4200번지를 힙에서 살펴보면, 힙의 베이스인 34 KB에 이를 더하면 물리 주소 39016을 얻게 됩니다. 그러나 이 주소는 올바른 물리 주소가 아닙니다. 먼저 힙 내에서의 오프셋, 즉 주소가 세그멘트의 시작으로부터 몇 번째 바이트인지를 확인해야 합니다. 힙은 가상 주소 4 KB(4096)에서 시작하기 때문에 오프셋은 4200 - 4096으로 계산되어 104가 됩니다. 이 오프셋(104)을 힙의 베이스 레지스터의 물리 주소(34 KB)에 더하면 원하는 결과인 34920을 얻을 수 있습니다.

그러나 만약 잘못된 주소인 힙의 마지막을 벗어난 7 KB와 같은 주소에 접근하려고 한다면 어떻게 될까요? 하드웨어는 이 주소가 범위를 벗어났다는 것을 감지하고 운영체제에 트랩을 발생시킵니다. 운영체제는 문제의 프로세스를 종료시킬 가능성이 큽니다. 이렇게 잘못된 주소 접근으로 인해 발생하는 문제를 C 프로그래머들이 많이 겪는 유명한 용어의 기원을 알 수 있습니다: 세그멘트 위반(segment violation) 또는 세그멘트 폴트(segment fault).

```{admonition} 세그멘트 위반(segment violation) 또는 세그멘트 폴트(segment fault)
세그멘트 위반(segment violation) 또는 세그멘트 폴트(segment fault)는 프로그래밍에서 주소 접근 오류가 발생했을 때 나타나는 용어로 기원적으로는 주소 접근 오류가 발생했을 때, 해당 프로세스가 접근한 메모리 주소가 메모리 세그멘트의 범위를 벗어났을 때 발생한 것입니다. 이러한 오류는 프로그램이 메모리를 잘못 사용하거나 액세스하려고 할 때 발생합니다.

세그멘트 위반 또는 세그멘트 폴트가 나타나면, 보통 운영체제는 프로그램이 메모리를 잘못 사용하는 오류를 신속하게 감지하여 프로세스나 시스템의 안정성을 유지하기 위해 해당 프로세스를 중단하거나 종료시킵니다.
```

- 트랩(trap): 프로세스가 잘못된 메모리 접근이나 불법적인 명령어 실행 등의 예외 상황을 발생시켰을 때, CPU가 현재 실행 중인 프로세스를 중단하고 운영체제에게 제어권을 넘기는 것을 말합니다. 트랩이 발생하면 운영체제는 적절한 예외 처리기를 실행하여 상황을 처리합니다.

세그멘테이션은 프로그램의 논리적 구조에 기반하여 메모리를 관리하므로, 프로그래머에게 직관적이고 유연한 메모리 모델을 제공합니다. 또한 세그멘트 단위로 메모리 보호와 공유가 가능하므로 시스템의 안정성과 보안을 향상시킬 수 있습니다. 그러나 세그멘테이션은 외부 단편화(external fragmentation) 문제를 야기할 수 있으며, 세그멘트 테이블 관리에 오버헤드가 발생할 수 있습니다.

- 외부 단편화(external fragmentation): 메모리에 할당되지 않은 빈 공간들이 여러 곳에 산재해 있어, 실제로는 충분한 메모리가 있음에도 불구하고 연속적인 메모리 공간을 할당하지 못하는 현상을 말합니다. 세그멘테이션에서는 가변 크기의 세그멘트를 할당하므로 외부 단편화가 발생할 수 있습니다.

현대의 대부분의 운영체제는 세그멘테이션보다는 페이징(paging) 기법을 사용하여 메모리를 관리합니다. 페이징은 고정된 크기의 페이지 단위로 메모리를 나누어 관리하므로, 외부 단편화 문제를 해결할 수 있습니다. 그러나 일부 운영체제에서는 세그멘테이션과 페이징을 혼합하여 사용하기도 합니다.

세그멘테이션은 초기의 메모리 관리 기법으로서 중요한 역할을 했으며, 메모리 보호와 공유의 기본 개념을 제공했습니다. 비록 현대의 운영체제에서는 주로 페이징 기법이 사용되고 있지만, 세그멘테

베이스와 바운드 레지스터를 사용하면 운영체제는 프로세스를 물리 메모리의 다른 부분으로 쉽게 재배치할 수 있습니다. 그러나 이러한 형태의 주소 공간에서 재미있는 사실은 스택과 힙 사이에 사용되지 않는 큰 공간이 존재한다는 것입니다. 이 공간은 주소 공간을 물리 메모리에 재배치할 때 물리 메모리를 차지하게 됩니다. 베이스와 바운드 레지스터 방식은 이러한 메모리 낭비가 심할 수 있으며, 또한, 주소 공간이 물리 메모리보다 큰 경우 실행이 어려워질 수 있습니다. 이러한 측면에서 볼 때, 베이스와 바운드 방식은 유연성이 부족한 것으로 여겨집니다.

## 세그멘트 종류의 파악

하드웨어는 주소 변환을 위해 세그멘트 레지스터를 사용합니다. 하드웨어는 가상 주소가 어느 세그멘트를 참조하는지, 그리고 그 세그멘트 안에서 오프셋은 얼마인지를 어떻게 알 수 있을까요?

한 가지 일반적인 접근 방법은 가상 주소의 최상위 몇 비트를 사용하여 주소 공간을 여러 세그멘트로 나누는 것입니다. 이 방법은 예를 들어 VAX/VMS 시스템에서 사용되었습니다. 위의 예에서는 3개의 세그멘트가 있으므로 주소 공간을 세그멘트로 나누기 위해서는 2비트가 필요합니다. 따라서 세그멘트를 표시하기 위해 가상 주소의 최상위 2비트를 사용하는 경우, 가상 주소의 형태는 다음과 같을 것입니다.

![segment(1)](https://github.com/entelecheia/os-2024/assets/162019986/aea2548e-a05e-4029-84bf-9002bf308b0d)

예를 들어, 최상위 2비트가 00이면 하드웨어는 가상 주소가 코드 세그멘트를 가리킨다는 것을 인식하고, 이에 따라 코드 세그멘트의 베이스와 바운드 쌍을 활용하여 주소를 정확한 물리 메모리 위치로 재배치합니다. 최상위 2비트가 01이면, 하드웨어는 주소가 힙 세그멘트를 가리킨다는 것을 인지하고, 힙의 베이스와 바운드를 사용하여 주소를 변환합니다. 이해를 돕기 위해 이전에 언급한 힙에 해당하는 가상 주소인 4200을 변환해 보면 가상 주소 4200에 대한 이진 표현은 다음과 같습니다.

![segment(2)](https://github.com/entelecheia/os-2024/assets/162019986/aad96d7c-48f4-4b56-b6e1-4011ea330a2a)

그림에서 볼 수 있듯이, 최상위 2비트 (01)는 하드웨어에게 참조하는 세그멘트의 종류를 알려줍니다. 그리고 하위 12비트는 해당 세그멘트 내의 오프셋을 나타냅니다. 예를 들어, 이진 형식으로 표현된 주소 0000 0110 1000은 16진수로는 0x068 또는 10진수로는 104입니다. 하드웨어는 세그멘트 레지스터를 이해하기 위해 처음 2비트를 사용하고, 그 다음 12비트를 세그멘트 오프셋으로 취합니다. 이 오프셋에 베이스 레지스터 값을 더하여 하드웨어는 최종적인 물리 주소를 계산합니다. 또한, 오프셋을 사용하면 바운드 검사도 쉽게 수행할 수 있습니다. 바운드를 넘어선 오프셋인지를 검사하기만 하면 됩니다. 그렇지 않으면 주소가 잘못된 것입니다. 만약 베이스와 바운드 쌍을 배열 형태로 저장한다면 (세그멘트당 하나의 항목), 원하는 물리 주소를 얻기 위해 다음과 같은 과정을 수행하게 됩니다.

![segment(3)](https://github.com/entelecheia/os-2024/assets/162019986/a4a11b0a-acd1-46b6-b71d-6199a2347543)

- SEG_MASK: 가상 주소에서 세그멘트 번호를 추출하기 위한 비트 마스크입니다. 이 마스크를 가상 주소와 AND 연산하면 세그멘트 번호만 남게 됩니다.

- SEG_SHIFT: 세그멘트 번호를 오른쪽으로 시프트하여 배열 인덱스로 사용하기 위한 시프트 양입니다. 이 값은 세그멘트 번호를 나타내는 비트 수와 같습니다.

- OFFSET_MASK: 가상 주소에서 세그멘트 내 오프셋을 추출하기 위한 비트 마스크입니다. 이 마스크를 가상 주소와 AND 연산하면 오프셋만 남게 됩니다.

우리는 현재 예를 기준으로 위 코드에서 사용된 상수 값들을 설정할 수 있습니다. SEG_MASK는 0x3000, SEG_SHIFT는 12, 그리고 OFFSET_MASK는 0xFFF로 지정됩니다. 세그멘트 종류를 나타내는 데 최상위 2비트를 사용하고, 주소 공간에는 코드, 힙, 스택 세그멘트만 존재하기 때문에 지정 가능한 세그멘트 하나가 미사용으로 남게 됩니다. 즉, 전체 주소 공간의 1/4은 사용이 불가능합니다. 이 문제를 해결하기 위해 일부 시스템은 코드와 힙을 하나의 세그멘트에 저장하고 세그멘트 선택을 위해 1비트만 사용합니다.

특정 주소의 세그멘트를 하드웨어적으로 파악하는 다른 방법들도 있습니다. 묵시적(implicit) 접근 방식에서는 주소가 어떻게 형성되는지를 관찰하여 세그멘트를 결정합니다. 예를 들어, 주소가 프로그램 카운터에서 생성된다면 해당 주소는 코드 세그멘트 내에 있을 것입니다. 주소가 스택 또는 베이스 포인터에 의해 생성된다면 주소는 스택 세그멘트 내에 있을 것입니다. 다른 주소는 모두 힙에 위치하고 있어야 합니다.

## 스택

지금까지 주소 공간의 중요한 구성 요소 중 하나인 스택을 다루지 않았습니다. 앞의 그림 19.3에서는 스택이 물리 메모리의 28 KB에 배치되어 있지만 다른 세그멘트들과는 다르게 확장 방향이 반대라는 중요한 차이가 있습니다. 스택은 물리 메모리의 28 KB에서 시작하여 26 KB를 차지하며, 가상 주소에서는 16 KB에서 14 KB에 해당합니다. 이에 따라 다른 방식의 주소 변환이 필요합니다.

첫 번째, 간단한 하드웨어가 추가로 필요합니다. 베이스와 바운드 값뿐만 아니라 하드웨어는 세그멘트가 어느 방향으로 확장하는지도 알아야 합니다. 예를 들어, 하나의 비트를 사용하여 주소가 양의 방향으로 확장되는 경우에는 1로 설정하고, 음의 방향으로 확장되는 경우에는 0으로 설정할 수 있습니다. 이러한 사실을 반영하여 아래 그림 19.4는 하드웨어가 관리해야 하는 정보를 나타냅니다.

- 방향 비트(Direction Bit): 세그멘트가 양의 방향으로 확장되는지 음의 방향으로 확장되는지를 나타내는 비트입니다. 이 비트를 통해 스택과 같이 음의 방향으로 확장되는 세그멘트를 지원할 수 있습니다.

하드웨어는 세그멘트가 반대 방향으로 확장될 수 있다는 것을 알기 때문에, 이러한 가상 주소에 대해서는 다른 방식으로 변환합니다. 스택에 해당하는 가상 주소를 예로 들어 변환해 보겠습니다. 이 예에서 가상 주소 15 KB에 접근하려고 한다고 가정할 때, 이 주소는 물리 주소 27 KB에 매핑되어야 합니다. 이 가상 주소를 이진 형태로 변환하면 11 1100 0000 0000 (16진수 0x3C00)이 됩니다. 하드웨어는 상위 2비트 (11)를 사용하여 세그먼트를 지정합니다. 이를 고려하면 3 KB의 오프셋이 남습니다. 올바른 음수 오프셋을 얻기 위해 3 KB에서 세그멘트 최대 크기를 빼야 합니다. 이 예에서는 세그멘트의 최대 크기가 4 KB이므로 올바른 오프셋은 3 KB에서 4 KB를 뺀 -1 KB입니다. 이 음수 오프셋 (-1 KB)을 베이스 (28 KB)에 더하면 올바른 물리 주소 27 KB를 얻게 됩니다. 바운드 검사는 음수 오프셋의 절댓값이 세그멘트의 크기보다 작다는 것을 확인하여 계산할 수 있습니다.

![19,4](https://github.com/entelecheia/os-2024/assets/162019986/8bee0453-44e9-488d-a552-4f113a37a520)

## 공유 지원

세그멘테이션 기법이 발전함에 따라 시스템 설계자들은 간단한 하드웨어 지원으로 새로운 종류의 효율성을 성취할 수 있다는 것을 깨달았습니다. 구체적으로, 메모리를 절약하기 위해 때로는 주소 공간들 간에 특정 메모리 세그멘트를 공유하는 것이 유용합니다. 특히, 코드 공유가 일반적이며, 현재 시스템에서도 널리 사용되고 있습니다.

이러한 공유를 지원하기 위해, 하드웨어에 보호 비트(protection bit)의 추가가 필요합니다. 각 세그멘트에 보호 비트를 추가하여 세그멘트를 읽거나 쓸 수 있는지, 혹은 세그멘트의 코드를 실행할 수 있는지를 나타냅니다. 코드 세그멘트를 읽기 전용으로 설정하면 주소 공간의 독립성을 유지하면서도, 여러 프로세스가 주소 공간의 일부를 공유할 수 있습니다. 각 프로세스는 여전히 자신의 전용 메모리를 사용한다고 생각하지만, 운영체제는 이러한 변경이 불가능하도록 설정된 메모리 영역을 비밀리에 공유하여 그러한 환상을 유지합니다.

- 보호 비트(Protection Bit): 각 세그멘트에 할당된 비트로, 세그멘트에 대한 접근 권한을 나타냅니다. 일반적으로 읽기, 쓰기, 실행 권한을 나타내는 비트들로 구성됩니다. 보호 비트를 통해 세그멘트 단위로 메모리 보호를 구현할 수 있습니다.

![19 5](https://github.com/entelecheia/os-2024/assets/162019986/35a246c0-422d-457f-b4e0-8a6cf2d7efa5)

하드웨어 (및 운영체제)가 유지하는 부가 정보의 예시가 그림 19.5에 나와 있습니다. 코드 세그멘트는 읽기 및 실행으로 설정되어, 같은 물리 세그멘트가 여러 가상 주소 공간에 매핑될 수 있습니다.

보호 비트를 사용하면 앞서 언급한 하드웨어 알고리즘이 수정되어야 합니다. 가상 주소가 범위 내에 있는지 확인하는 것뿐만 아니라 특정 액세스가 허용되는지를 확인해야 합니다. 사용자 프로세스가 읽기 전용 페이지에 쓰기를 시도하거나 실행 불가능한 페이지에서 실행하려고 할 때 하드웨어는 예외를 발생시켜서 운영체제가 위반 프로세스를 처리할 수 있도록 해야 합니다.

## 소단위 대 대단위 세그멘테이션

우리 예제의 대부분은 지금까지 소수의 세그멘트 (예: 코드, 스택, 힙)만을 지원하는 시스템에 주로 초점을 맞추고 있습니다. 이러한 세그멘테이션은 대단위(coarse-grained)로 간주할 수 있습니다. 이는 주소 공간을 비교적 큰 단위의 공간으로 분할하기 때문입니다. 그러나 일부 초기 시스템 (예: Multics)은 주소 공간을 작은 크기의 공간으로 잘게 나누는 것이 허용되어 소단위(fine-grained) 세그멘테이션이라고도 합니다.

```{admonition} 소단위, 대단위 세그멘테이션
1. 소단위 세그멘테이션 (Fine-Grained Segmentation)
- 소단위 세그멘테이션은 주소 공간을 작은 크기의 세그멘트로 분할하는 방법입니다.
- 이 방법은 메모리를 작은 조각으로 나누어 세밀하게 관리할 수 있습니다.
- 예를 들어, 각 프로세스가 자신의 코드, 데이터, 스택 등을 각각의 작은 세그멘트로 분할하여 메모리를 효율적으로 활용할 수 있습니다.
- 소단위 세그멘테이션은 보다 세밀한 메모리 관리와 보안 제어를 제공할 수 있지만, 세그먼트 테이블과 같은 자원을 더 많이 소비하고 관리하기 어려울 수 있습니다.

2. 대단위 세그멘테이션 (Coarse-Grained Segmentation)
- 대단위 세그멘테이션은 주소 공간을 상대적으로 큰 단위의 세그멘트로 분할하는 방법입니다.
- 이 방법은 메모리를 큰 단위로 관리하여 세그먼트 테이블과 같은 자원을 절약할 수 있습니다.
- 주로 코드, 데이터, 스택과 같이 큰 단위의 세그멘트로 분할하여 관리합니다.
- 대단위 세그멘테이션은 자원을 덜 소비하고 관리하기 쉽지만, 세밀한 메모리 관리와 보안 제어에는 적합하지 않을 수 있습니다.

소단위 세그멘테이션은 세밀한 메모리 관리와 보안이 필요한 경우에 유용하며, 대단위 세그멘테이션은 자원을 절약하고 단순한 메모리 관리에 적합합니다. 선택은 시스템 요구사항과 우선순위에 따라 다를 수 있습니다.
```

많은 수의 세그멘트를 지원하기 위해서는 여러 세그멘트의 정보를 메모리에 저장할 수 있는 세그멘트 테이블과 같은 하드웨어가 필요합니다. 세그멘트 테이블을 이용하면 매우 많은 세그멘트를 손쉽게 생성하고 융통성 있게 세그멘트를 사용할 수 있습니다. 예를 들어, Burroughs B5000과 같은 초창기 시스템은 수천 개의 세그멘트를 지원했고, 컴파일러가 코드나 데이터를 여러 세그멘트로 분할할 경우 운영체제와 하드웨어가 이를 지원했습니다. 당시의 생각은 소단위 세그멘트로 관리하는 것이 운영체제가 사용 중인 세그멘트와 미사용인 세그멘트를 구분하여 메인 메모리를 더 효율적으로 활용할 수 있다는 것이었습니다.

- 세그멘트 테이블(Segment Table): 각 세그멘트의 정보를 저장하는 자료구조입니다. 세그멘트 테이블은 세그멘트의 베이스 주소, 크기, 보호 비트 등을 포함합니다. 세그멘트 테이블을 통해 많은 수의 세그멘트를 효율적으로 관리할 수 있습니다.

## 운영체제의 지원

이제 세그멘트의 기본 동작은 이해하고 있어야 합니다. 시스템이 각 주소 공간 구성 요소를 별도로 물리 메모리에 재배치하기 때문에 전체 주소 공간이 하나의 베이스-바운드 쌍을 가지는 간단한 방식에 비해 물리 메모리를 효율적으로 절약할 수 있습니다. 특히, 스택과 힙 사이의 사용하지 않는 공간에 물리 메모리를 할당할 필요가 없어져서 같은 크기의 물리 메모리에 더 많은 주소 공간을 탑재할 수 있습니다.

그러나 세그멘테이션은 새로운 많은 문제를 제기합니다.

첫 번째 문제는 오래된 문제입니다. 문맥 교환 시 운영체제가 수행해야 하는 작업은 무엇일까요? 바로 세그멘트 레지스터의 저장과 복원입니다. 각 프로세스는 자신의 가상 주소 공간을 가지며, 운영체제는 프로세스가 다시 실행하기 전에 레지스터들을 올바르게 설정해야 합니다.

두 번째로, 더욱 중요한 문제는 미사용 중인 물리 메모리 공간의 관리입니다. 새로운 주소 공간이 생성되면 운영체제는 이 공간의 세그멘트를 위한 비어있는 물리 메모리 영역을 찾을 수 있어야 합니다. 이전에는 각 주소 공간의 크기가 동일하다고 가정했지만, 이제는 프로세스가 많은 세그멘트를 가질 수 있고, 각 세그멘트의 크기도 다를 수 있습니다.

일반적으로 발생할 수 있는 문제는 물리 메모리가 빠르게 작은 크기의 빈 공간들로 채워진다는 것입니다. 이 작은 빈 공간들은 새로이 생겨나는 세그멘트에 할당하기도 힘들 뿐만 아니라 기존 세그멘트를 확장하는 데에도 도움이 되지 않습니다. 이러한 문제를 외부 단편화(external fragmentation)라고 부릅니다.

- 외부 단편화(External Fragmentation): 메모리에 할당되지 않은 작은 빈 공간들이 여러 곳에 산재해 있는 현상을 말합니다. 이러한 작은 빈 공간들은 새로운 메모리 할당 요청을 만족시키기에는 충분히 크지 않아서 메모리 낭비를 초래합니다. 세그멘테이션에서는 가변 크기의 세그멘트를 할당하므로 외부 단편화가 발생할 수 있습니다.

![19 6](https://github.com/entelecheia/os-2024/assets/162019986/838abb6b-50c8-4139-9e92-42429f125f97)

이 예에서 새로운 프로세스가 생성되어 20 KB를 할당하려고 합니다. 위 예에서 24 KB의 빈 공간이 존재하지만 하나의 연속된 공간이 아닌 세 개의 청크(chunk)로 나누어져 있습니다. 운영체제는 20 KB의 요청을 충족시킬 수 없습니다.

이 문제의 해결책 중 하나는 기존의 세그멘트를 정리하여 물리 메모리를 압축(compact)하는 것입니다. 예를 들어, 운영체제는 현재 실행 중인 프로세스를 중단하고, 그들의 데이터를 하나의 연속된 공간에 복사한 후, 세그멘트 레지스터가 새로운 물리 메모리 위치를 가리키게 하여, 자신이 작업할 큰 빈 공간을 확보합니다. 이렇게 함으로써 운영체제는 새로운 할당 요청을 충족시킬 수 있습니다. 그러나 세그멘트 복사는 메모리에 부하가 큰 연산이고 일반적으로 상당량의 프로세서 시간을 사용하기 때문에 압축은 비용이 많이 듭니다. 압축 작업 후의 물리 메모리는 그림 19.6(오른쪽)에 나와 있습니다.

- 압축(Compaction): 메모리에 할당된 세그멘트들을 한 쪽으로 모아서 연속적인 빈 공간을 만드는 작업입니다. 압축을 통해 외부 단편화를 해소할 수 있지만, 모든 세그멘트를 이동시켜야 하므로 오버헤드가 발생합니다.

간단한 방법은 빈 공간 리스트를 관리하는 알고리즘을 사용하는 것입니다. 빈 공간 관리 알고리즘은 할당 가능한 메모리 영역을 리스트 형태로 유지합니다. 최적 적합(best-fit), 최악 적합(worst-fit), 최초 적합(first-fit), 버디 알고리즘(buddy algorithm) 등 여러 가지 방식이 있습니다. 이 중 최적 적합은 빈 공간 리스트에서 요청된 크기와 가장 비슷한 크기의 공간을 할당합니다. 알고리즘이 아무리 정교하게 동작한다 해도 외부 단편화는 여전히 존재하며, 좋은 알고리즘은 외부 단편화를 최소화하는 것이 목표입니다.

> **팁: 1000개의 해결책이 존재한다면, 최선의 해결책은 존재하지 않는다**
>
> 외부 단편화를 최소화하기 위한 매우 다양한 알고리즘이 존재한다는 사실은 중요한 의미를 가집니다 : 외부 단편화를 해결하는 유일한 "최선"책은 존재하지 않습니다. 우리는 합리적인 선택을 하고 그 해결책이 충분히 좋기를 희망할 뿐입니다. 유일한 진정한 해결책은 가변 길이 할당을 허용하지 않음으로써 아예 문제가 생기지 않도록 하는 것입니다. 이 해결책에 대해서는 추후에 살펴볼 예정입니다.

## 요약

세그멘테이션은 많은 문제를 해결하며, 메모리 가상화를 효과적으로 실현할 수 있습니다. 단순한 동적 재배치를 넘어 세그멘테이션은 주소 공간 상의 논리 세그멘트 사이의 큰 공간에 대한 낭비를 피함으로써 드문드문 사용되는 주소 공간(sparse address space)을 지원할 수 있습니다. 세그멘테이션에 필요한 산술 연산은 쉽고 하드웨어 구현에 적합하기 때문에 속도가 빠르며 변환 오버헤드도 최소입니다. 또한, 코드 공유의 장점도 부가적으로 발생합니다. 코드가 별도의 세그멘트에 존재한다면 그러한 코드는 실행 중인 여러 프로그램 사이에서 공유될 수 있습니다.

- 드문드문 사용되는 주소 공간(Sparse Address Space): 프로그램의 주소 공간 중 실제로 사용되는 부분이 드문드문 흩어져 있는 상태를 말합니다. 주소 공간의 상당 부분이 사용되지 않고 비어 있음을 의미합니다. 세그멘테이션은 이러한 드문드문 사용되는 주소 공간을 효과적으로 지원할 수 있습니다.

그러나 세그멘트의 크기가 일정하지 않기 때문에 몇 가지 문제가 발생합니다. 첫째는 외부 단편화입니다. 세그멘트는 가변 크기이기 때문에 빈 공간들의 크기 역시 모두 다르며 메모리 할당 요청을 충족시키는 것이 어려울 수 있습니다. 비교적 정교한 알고리즘을 사용하거나 주기적으로 메모리를 압축할 수는 있지만, 외부 단편화 문제는 가변 길이 할당의 태생적인 문제이기 때문에 회피하기 어렵습니다.

두 번째로, 더욱 중요한 문제는 세그멘테이션이 아직 일반적인 드문드문 사용되는 주소 공간을 지원할 만큼 충분히 유연하지 못하다는 것입니다. 예를 들어, 크기가 크지만 드문드문 사용되는 힙이 하나의 논리적인 세그멘트에 배정되어 있다고 할 때 이 힙에 접근하기 위해서는 힙 전체가 여전히 물리 메모리에 존재해야 합니다. 다시 말해서, 주소 공간이 사용되는 모델과 이를 지원하기 위한 세그멘테이션의 설계 방법이 정확히 일치하지 않는다면, 세그멘테이션은 제대로 동작하지 않을 수 있습니다. 이러한 문제들에 대한 새로운 해결책을 찾아야 할 필요가 있습니다.

이상으로 세그멘테이션에 대해 자세히 살펴보았습니다. 세그멘테이션은 초기의 메모리 가상화 기법으로서 중요한 역할을 했으며, 메모리 보호와 공유의 기본 개념을 제공했습니다. 또한, 드문드문 사용되는 주소 공간을 효과적으로 지원할 수 있다는 장점이 있습니다. 그러나 외부 단편화와 같은 문제점도 있으며, 현대의 시스템에서는 페이징과 같은 더 발전된 기법이 주로 사용되고 있습니다. 하지만 세그멘테이션의 기본 개념과 원리를 이해하는 것은 메모리 관리 기법의 발전 과정을 파악하는 데 도움이 될 것입니다.
