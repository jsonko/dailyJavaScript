Refs and the DOM
======

일반적인 React dataflow에서, parent element와 children이 interaction하는 유일한 방법은 prop을 이용하는 방법이다. 
허나, Child를 수정하기 위해서, 가끔 일반적인 dataflow를 벗어나 명시적으로 child 수정이 필요한 경우가 발생한다. 
이를 위하여, React는 escape hatch를 제공한다. 

The ref Callback Attribute
----
**ref** 속성에 callback function을 할당 할 수 있으며, 이 callback은 component가 mount되거나 unmount되는 직후에 실행된다. ref이 HTML Element에서 사용되는 경우에, ref callback은 arguments로 해당 DOM element를 전달 받게 된다.
Mounting되는 시점에는 해당 DOM element가 전달되고, Unmounting 시점에서는 *null* 이 전달된다. 

```
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.focus = this.focus.bind(this);
  }

  focus() {
    // Explicitly focus the text input using the raw DOM API
    // this.textInput 은 아래 ref callback에 의해서 input DOM Node를 가리킴. 
    this.textInput.focus();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in this.textInput.
    // input tag에 정의 된 ref callback 함수 
    // argument로 input tag의 DOM Node를 전달 받게 된다. 
    return (
      <div>
        <input
          type="text"
          ref={(input) => { this.textInput = input; }} />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focus}
        />
      </div>
    );
  }
}
```

ref callback을 사용하여, DOM element를 class property로 지정하는 것은 common한 pattern 이다. 위 예제를 다음과 같이 짧게 줄일 수도 있다. ref={input => this.textInput = input}.

ref attirbute이 custom component에서 사용되는 경우, ref callback은 argument로 해당 component의 mount된 instance를 받게 된다. 예를 들면, 위의 input 예제를 CustomTextInput이라는 element로 wraping 하였고, mount 직후에 click한 것처럼 처리하려면 아래와 같이 하면 된다. 

```
class AutoFocusTextInput extends React.Component {
  componentDidMount() {
    this.textInput.focus();
  }

  render() {
    return (
      <CustomTextInput
        ref={(input) => { this.textInput = input; }} />
    );
  }
}
```

functional components는 자체 instance가 없기 때문에, ref attribute을 사용하지 않을 수 있으나, functional component에서도 아래와 같이 ref를 사용할 수 있다. 

```
function CustomTextInput(props) {
  // textInput must be declared here so the ref callback can refer to it
  let textInput = null;

  function handleClick() {
    textInput.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={(input) => { textInput = input; }} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );  
}
```

<br>

Don't Overuse Refs
---

ref에 대한 첫인상은 ref를 갖고 app 안에서 어떤 기능이 동작하도록 만들 수 있게 보일 것이다. 그러나 Component 계층 안에서 state를 어디서 소유되어야 하는지에 대해서 다시한번 생각해 볼 필요가 있다. 그리고, state는 hierachy에서 higher level에 위치하는 것이 적절 할 것이라고 생가할 수 있을 것이다. 

Caveats
---
ref callback이 만일 inline 함수 처럼 정의되어 있다면, 이 함수는 update시 2번 호출될 것이다. 첫번째 null 과 함께 호출되고, 두번째는 DOM element와 함께 호출될 것이다. 이는 각각의 render와 함께 새로운 함수의 instance가 생성되기 때문에, React는 old ref에 대해서 clear할 필요가 있고, 새로운 ref를 설정할 필요가 있다. 이를 피하려면, ref callback은 반드시 class 내부에 bound method로 정의해야 한다. (대부분의 경우에는 문제가 되지 않지만.) 