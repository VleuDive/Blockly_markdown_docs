Javascript로 블록 정의하기
================================
앞서 Blockly의 custom block을 정의하는 방법에는 Javascript만 이용하는 것과 JSON array를 같이 이용하는 것, 두 가지가 있다고 했습니다.
이번 문서에서는 그 중 첫 번째 방법인 Javascript만 이용하는 방법으로 블록을 정의해 보겠습니다.

정의할 블록
----------------------------------
이번과 다음 문서에서 정의할 블록의 이름은 Short_math이며, 다음과 같은 모양을 하고 있습니다.

![short_math_ex](img/short_math_ex.png)

이 블록은 Top+Bottom connection을 가지고 있으며, 2개의 Value Input을 가집니다. 또한 중간에 dropdown field가 있어 연산의 종류를 선택할 수 있습니다.
이 블록은 '짧은 표현식'을 구현하고 있습니다. C++ 이나 Java, Python 등의 문법으로 보자면 **a+=b** 식의 연산을 나타내는 블록입니다. 단, 중간에 dropdown field를 두어 연산의 종류를 선택할 수 있게 했기 때문에 결과적으로는 한 블록에서 a+=b, a-=b, a/=b, a*=b 의 연산이 모두 가능합니다.

블록 정의 코드
----------------------------------------
Javascript만 사용하여 Short_math를 정의하는 코드는 다음과 같습니다.

```javascript
Blockly.Blocks['short_math'] = {
  init: function() {
    this.appendValueInput("a")
        .setCheck("Number");
    this.appendDummyInput()
        .appendField(new Blockly.FieldDropdown([["+","+"], ["*","*"], ["-","-"], ["/","/"]]), "select")
        .appendField("=");
    this.appendValueInput("b")
        .setCheck("Number");
    this.setInputsInline(true);
    this.setPreviousStatement(true);
    this.setNextStatement(true);
    this.setColour(260);
 this.setTooltip("Short version of arithmetic calculations");
 this.setHelpUrl("");
  }
};
``` 

코드를 하나하나 분석해 보겠습니다.

### Block 정의 함수

Block 정의 함수는 다음과 같은 형태를 하고 있습니다.

```javascript
Blockly.Blocks['블록 이름']={
    init:function(){
        // 블록의 특성 정의
    }
}
```

이 구문은 한 .js 파일 내에 여러 번 호출될 수 있으며, 해당 파일 안에 정의하고 싶은 블록의 개수만큼 호출되어야 합니다. JSON array를 활용하는 방식에서는 한 번의 함수 호출로 한꺼번에 여러 개의 블록을 정의할 수 있지만, Javascript 방식의 Blockly.Blocks[]의 경우 한 번에 한 블록밖에 정의하지 못합니다. 

지금 정의하는 Short_math와 같은 간단한 블록을 정의할 때는 이 구문을 살짝 응용해서, init 메소드를 삭제하고 다음과 같이 사용해도 큰 문제는 없지만, mutator나 extension 고급 기능을 추가하게 되면 init 외에도 다양한 메소드를 추가하게 되기 때문에, 복잡한 구조의 블록의 경우 init을 생략하지 않는 위의 방법이 더 바람직합니다.

```javascript
Blockly.Blocks['블록 이름']={
        // 블록의 특성 정의
}
```

### 첫 번째 Value Input 정의

Short_math 블록은 두 개의 value input을 가집니다. 그 중 첫 번째 Value Input을 추가하는 부분은 다음과 같습니다.

```javascript
  this.appendValueInput("a")
        .setCheck("Number");
```

**this**는 지금 정의하고 있는 바로 이 블록, Short_math를 의미합니다.

**this.appendValueInput("a")** 는 이 블록에 'a'라는 이름을 가지고 있는 value input을 추가하겠다는 뜻입니다. 여기서 정의해 준 이름 'a'는 블록에는 표시되지 않지만, code generation시에 block의 구성 요소를 지정하는 데 사용됩니다.

**setCheck("Number")** 는 이 input이 받을 수 있는 자료형을 'Number'로 고정합니다. 이렇게 설정해 놓으면, 'Number' 이외의 자료형(ex: String) 을 가진 value input은 이 블록에 적용되지 못합니다. 이 함수의 이름이 setCheck인 것은, input 내에서 타입을 지정해 주는 프로퍼티의 이름이 'check'이기 때문입니다. Javascript 스타일로 블록을 정의할 때는 이런 식으로, setXXX(); 와 같은 하위 메소드를 호출해 블록 또는 input의 세부 프로퍼티를 조작합니다.

### Dropdown Field 추가

두 번째로 정의해야 하는 부분은 dropdown field입니다. dropdown field는 다음과 같이 정의합니다.

```javascript
 this.appendDummyInput()
        .appendField(new Blockly.FieldDropdown([["+","+"], ["*","*"], ["-","-"], ["/","/"]]), "select")
        .appendField("=");
```

이 부분에서 **"Field 사용 시에는 Dummy input을 같이 사용하는 경우가 많다"** 는 특징이 잘 드러납니다. 여기서도 this는 지금 정의하고 있는 Short_math 블록을 나타냅니다.

**appendDummyInput()** 은 이 블록에 dummy input을 추가하는 함수입니다. 이 dummy input의 field로 dropdown field를 추가하게 될 것입니다.
이 코드에서 총 두 번 등장하는 **apendField()** 구문이 바로 input에 field를 추가하는 구문입니다. 위에서 등장한 setXXX() 류의 함수가 블록에 이미 있던 기본 property를 조작하는 것에 가깝다면, appendField() 함수는 해당 블록에 아예 새로운 field를 추가해 블록의 구조 자체를 바꾸는 함수입니다.

첫 번째로 등장하는 appendField()에는 **new Blockly.FieldDropdown([["+","+"],["*","*"],["-","-"],["/","/"]]),"select")** 라는 구문이 파라미터로 전달되는데, 이 파라미터는 바로 새로운 dropdown field를 생성하는 생성자 구문입니다. 

그 중 **[["+","+"],["*","*"],["-","-"],["/","/"]]** 부분은 dropdown field 의 option을 지정하는 리스트입니다. 리스트 내 항목의 개수는 얼마든지 늘릴 수 있으며, 이 리스트는 한 쌍의 string이 들어 있는 더 작은 리스트의 집합으로 이루어져 있습니다. 이 string 쌍에서, 앞의 string은 블록 사용자가 직접 보고 선택하게 될, 즉 블록에 직접 표시될 option 이름이고 뒤의 string은 프로그램 내에서 사용되는 option이름입니다. 이처럼 두 이름을 같게 해도 무방합니다. 또한 **"select"** 는 이 dropdown field 자체의 이름을 지정한 것입니다. 후에 code generation 등을 할 때 지정자로서 요긴히 쓰입니다.

두 번쨰로 등장하는 appendField()에는 "="이라는 string 하나만 전달되었습니다. 이는 =라는 문자를 표시하는 label로써, label은 이런 식으로 string 하나만 전달해서 추가할 수도 있습니다.


### 두 번째 Value Input 정의 및 블록 세부 속성 정의 

두 번째 Value Input을 정의하고 남아 있는 블록의 세부 속성을 정의하는 코드는 다음과 같습니다. 두 번째 value input을 정의하는 구문은 첫 번째 value input을 정의하는 구문과 거의 유사하므로 넘어갑니다.

```javascript
    this.appendValueInput("b")
        .setCheck("Number");
    this.setInputsInline(true);
    this.setPreviousStatement(true);
    this.setNextStatement(true);
    this.setColour(260);
    this.setTooltip("Short version of arithmetic calculations");
    this.setHelpUrl("");
```

