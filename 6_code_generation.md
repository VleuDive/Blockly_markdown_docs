Code generation 코드 작성하기
===========================================
지금까지 Blockly에서 custom block의 모양과 성질을 정의하였습니다. 지금까지의 단계를 모두 마쳤다면, 이제 사용자의 마음대로 정의된, 온전한 '형태'를 갖춘 블록이 만들어진 것입니다. 하지만, 이것만으로는 부족합니다. 지금까지 만든 블록은 아직 형태만 갖추어진 껍데기이기 때문에 실질적인 기능을 나타내지 못합니다. 블록이 실제 프로그래밍 언어로 된 코드와 연결될 수 있게 하기 위해서는 code geneartor 라는 특수한 코드를 같이 붙여 주어야 합니다. 보통 code generator는 block definition  부분과 별도의 .js파일로 분리하여 작성하게 됩니다.

Code generator는 Javascript로 정의합니다. 두 가지 방법으로 작성 가능한 블록 정의 구문과는 다르게, **code generator 정의 구문은 Javascript 스타일로만 정의가 가능합니다.** 또한, 블록을 변환할 언어(목표 언어)에 따라서 실제 매핑되는 구문이 달라져야 하기 때문에, 코드 작성 시 어떤 언어로 변환하는 것인지 명시해 주어야 합니다. 자세한 코드는 아래에서 살펴보겠습니다.

Code generator 정의 코드
---------------------------------------------------------------

여태까지 정의한 블록인, Short_math 블록을 가지고 계속 진행합니다.

![short_math](img/short_math_ex.png)

이 블록에 Python 코드를 연결시켜 보겠습니다. 이 블록과 연결해서 생성하려는 코드는 a+=b, a*=b, a/=b, a-=b 중 한 가지입니다.

전체 코드는 다음과 같이 작성합니다.

```javascript
Blockly.Python['short_math']=function(block)
{
    var num_1=Blockly.Python.valueToCode(block,'a',Blockly.Python.ORDER_ATOMIC);
    var dropdown_select = block.getFieldValue('select');
    var num_2=Blockly.Python.valueToCode(block,'b',Blockly.Python.ORDER_ATOMIC);
    var code=num_1+dropdown_select+"= "+num_2+"\n";
    return code;
}
```

코드를 자세히 분석해 보면 다음과 같습니다.

### Code generation 시작 함수

코드는 전반적으로 이렇게 구성되어 있습니다.

```javascript
Blockly.Python['short_math']=function(block){
    //함수 내용
}
```

**Blockly.Python['short_math']** 부분은 "Short_math 블록에 Python으로 된 코드를 붙일 것이다" 라고 해석할 수 있습니다. 여기서 short_math는 이미 정의해 놓은 Short_math 블록의 type을 가리킵니다. 이 구조 자체는 Javascript 스타일로 블록을 정의할 때와 매우 유사하지만, 한 가지 다른 점이 있다면 시작 부분이 **Blockly.Blocks 가 아니라 Blockly.Python**이라는 것입니다. 바로 이 .Python 부분에서 블록을 어떤 언어로 변환한 것인지를 지정합니다. Python 이외의 다른 언어로의 변환을 하고 싶다면, 시작 부분을 다음과 같이 써 주시면 됩니다.

```javascript
Blockly.Lua // Lua로 변환
Blockly.Javascript // Javascript로 변환
Blockly.Dart // Dart로 변환
Blockly.PHP // PHP로 변환 
```

이후 **function(block)** 부분에서 block을 파라미터로 받는 함수를 선언하고, 그 함수 안에 code generation의 단계가 기술됩니다.

### Block으로부터 값 읽어 오기

Short_math 블록에는 읽어 와야 할 값이 총 3가지 있습니다.

1. 첫 번째 value input(첫 번째 피연산자)의 값
2. Dropdown field에서 선택된 option값 (연산의 종류)
3. 두 번째 value input(두 번째 피연산자)의 값

이 값들을 Blockly에서는 이런 식으로 읽어 사용합니다.

```javascript
var num_1=Blockly.Python.valueToCode(block,'a',Blockly.Python.ORDER_ATOMIC);
var dropdown_select = block.getFieldValue('select');
var num_2=Blockly.Python.valueToCode(block,'b',Blockly.Python.ORDER_ATOMIC);
```

첫 번째 줄은 첫 번째 value input을 읽어 오는 코드입니다. **Blockly.Python.valueToCode()**함수를 사용해 값을 읽어 오고 있습니다. (Python 이외의 다른 언어를 사용하려면 .Python부분을 다른 언어로 바꾸어 주시면 됩니다.) Blockly에서 블록에 할당된 input이나 field의 값을 읽어 올 때는 이처럼 미리 지정된 함수를 사용하는데, 값 읽기에 사용되는 함수 몇 가지를 간단히 정리하면 다음과 같습니다.

* valueToCode() : Value input의 값을 읽습니다. 파라미터는 3개이며, 순서대로 해당 input이 위치하는 블록, input의 이름, 그리고 그 값이 속하는 타입의 연산자 우선순위입니다. 연산자 우선순위에 대해서는 뒤에 다시 정리하겠습니다.

* statementToCode() : Statement input의 값을 읽습니다. 파라미터는 2개이며, 순서대로 해당 input이 위치하는 블록과 input의 이름을 의미합니다.

* block.getFieldValue() : 블록에 단일 Field가 있는 경우, 이 field의 값을 읽어 오는 함수입니다. 특이하게도 이 함수는 위의 두 함수와 다르게 독립적으로 호출되지 않고 block의 하위 메소드로 호출됩니다. 그렇기 때문에 어떤 블록의 field를 읽어 오는 것인지 명시해 줄 필요가 없고, 읽어 오려는 field의 이름만 파라미터로 넘겨 주면 됩니다.

이번 예시에서는 2개의 input과 1개의 dropdown field가 존재하기 때문에 valueToCode()와 block.getFieldValue() 함수만 사용되었습니다. 첫 번째 줄에서는 block의 "a" input 값을 읽어 오고, 두 번째 줄에서는 block의 select field의 선택 값을 가져오며 마지막 줄에서는 block의 "b" input의 값을 읽어 옵니다.

### 연산자 우선순위

valueToCode() 함수는 세 번째 파라미터로 **연산자 우선순위**를 받습니다. 연산자 우선순위는 코드를 생성하고 조합하는 과정에서 꽤 중요한 역할을 하는데요, Blockly/generators/(언어명).js 파일 안에 각 언어별 연산자 우선순위가 지정되어 있습니다. 미리 지정되어 있는 값이기 때문에 사용할 때는 그 값을 고민할 필요 없이, 미리 지정되어 있는 해당 연산자의 이름만 호출해서 사용 가능합니다.

하지만 조금 이상한 것은, valueToCode()가 가져오는 것은 보통 Number나 String 등의 값일 텐데, 이런 단일 값을 가져올 때에도 우선순위를 명시해 주어야 하는가 하는 점입니다. 결론부터 말하면 "예"입니다. 이 연산자 우선순위는 나중에 코드를 생성하고 조합하는 과정에서 필요한 부분을 괄호로 묶어 주는 등의 작업을 하는 데 필요합니다. 지금 다루고 있는 블록에 어떤 input이 결합하느냐에 따라 최종 코드가 어떤 모양이 될 지가 결정되며, 어떤 부분이 괄호로 묶일지 하는 점도 달라집니다. 이런 점을 사람이 하나하나 하드코딩해 줄 수 없기 때문에, Blockly는 연산자 우선순위를 이용해 코드의 형태를 결정하는 방식을 취하고 있습니다. 그렇게 하기 위해서는 숫자 하나, 문자열 하나에도 모두 우선순위를 부여해서 Blockly가 다루는 모든 타입의 결합 형태를 파악할 필요가 있습니다.  

Python 기준으로 자주 쓰이는 연산자 우선순위에는 다음과 같은 것들이 있습니다.

* ORDER_ATOMIC : 숫자나 String 등 하나의 독립적 값을 의미합니다.

* ORDER_COLLECTION : 리스트나 튜플 등 여러 개의 요소가 하나의 큰 덩어리를 이루는 형태의 자료구조를 의미합니다.

* ORDER_NONE : 정의되어 있는 어떤 연산자에도 해당되지 않는 것을 리턴하는 경우나 리턴값이 없는 경우 등에 사용합니다.

### 코드 생성

예시 코드의 마지막 두 줄은 이렇게 끝납니다. 이 부분이 실제 Python code를 생성해서 string으로 리턴해 주는 부분이죠.

```javascript
var code=num_1+dropdown_select+"= "+num_2+"\n";
return code;
``` 

첫 번째 줄을 리턴한 code를 생성하는 부분입니다. 앞에서 읽어 온 다양한 값들과 파이썬 구문의 일부를 조합하여 하나의 완전한 코드를 만들어 내고 있습니다. 만약 num_1에 "a", dropdown_select에 "+", num_2에 "5"가 들어왔다면 완성되는 코드는 다음과 같습니다.

```python
a+=5
```

그 다음 줄에서는 만들어진 code를 리턴하고 있습니다. 이때 주의하실 점은, **output이 있는 블록과 없는 블록은 code를 리턴하는 방식이 서로 다르다**는 것입니다. Short_math는 output이 없는 블록이므로, 단순히 code 한 줄만 리턴합니다. 이런 식으로 output이 없는 블록은 코드만 리턴해야 합니다. 

하지만 output이 있는 블록의 경우, 다음과 같이 code와 연산자 우선순위를 같이 리턴해 주어야 합니다.

```javascript
return [code, Blockly.Python.ORDER_ATOMIC];
```

output이 있는 블록의 경우 실행 결과 리턴되는 값이 있다는 뜻이기 때문에, 그 리턴 값에 대한 우선순위를 같이 명시해 주어야 합니다. 만약 리턴하는 값이 이미 정의된 연산자 범주 안에 속하지 않거나, 블록 모양상으로만 output이 존재하고 실제로는 리턴값이 없는 경우라면 ORDER_ATOMIC 대신 ORDER_NONE을 리턴할 수 있습니다.

이 과정까지 모두 끝내면 모양과 기능을 동시에 갖춘 온전한 블록이 만들어집니다. 이제 남은 것은 만든 블록을 Blockly editor에 등록하는 것입니다. 그 점에 대해서는 다음 문서에서 다루겠습니다.

