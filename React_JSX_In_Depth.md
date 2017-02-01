JSX In Depth
============

JSX는 React.createElement(component, props, ..children)의 syntactic sugar 이다. 

```JSX
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

컴파일 하면, 

```Javascript
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```


Specifying The React Element Type
-
JSX tag의 첫번째 part는 React Element의 type을 결정한다. 대문자로 시작하는 type은 JSX tag가  React component를 참조한다는 것을 의미한다. 즉, <Foo /> 표현을 사용했다면, Foo element는 반드시 scope 내에 있어야 한다. 

### React Must be in Scope
```
import React from 'react';
import CustomButton from './CustomButton';

function WarningButton() {
  // return React.createElement(CustomButton, {color: 'red'}, null);
  return <CustomButton color="red" />;
}
```

Warning Button 내에서 사용되는 React와 CustomButton 을 import하여, scope안에 존재하도록 추가해야한다. 

###Using Dot Notation for JSX Type

```
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```
JSX 안에서도  dot-notation 사용이 가능하다. Dot-notation은 Single module내에 여러 React Component를 export하고 있는 경우에 사용하면 유용하다. 

###User-Defined Components Must Be Capitalized

소문자로 시작하는 type명의 경우에는 built-in component 라고 판단하여, React.createElement.**Type** 형태로 처리해 버린다. 따라서 사용자 정의 Component의 경우에는 대문자로 시작하도록 정의해야 React.createElement(Foo) 형태로 compile이 된다. 

###Choosing the Type at Runtime

React Element type으로 General expression(일반 표현식)을 사용할 수 없다. 만일 사용하고 싶다면, 대문자로 시작하는 변수에 해당 내용으로 assign하고 해당 변수를 JSX에 사용한다. 
이는 종종 prop 에 따라서 Rendering 할 component 를 선택할때 사용되기도 한다. 

```

// 아래와 같이 components[props.storyType]으로 사용하면 안된다. 
function Story(props) {
  // Wrong! JSX type can't be an expression.
  return <components[props.storyType] story={props.story} />;
}

// 대문자로 시작하는 변수에 할당하여 사용한다. 
function Story(props) {
  // Correct! JSX type can be a capitalized variable.
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

<br>

Props in JSX
-

###Javascript Expressions

prop 으로 javascript expression은 넘겨줄 수 있으며, 이때는 { }를 사용한다. *if* 나 *for* 는 javscript expression이 아니기 때문에, JSX에 바로 사용될 수는 없으나, JSX 밖에서는 아래와 같이 사용할 수 있다. 

```
function NumberDescriber(props) {
  let description;
  if (props.number % 2 == 0) {
    description = <strong>even</strong>;
  } else {
    description = <i>odd</i>;
  }
  return <div>{props.number} is an {description} number</div>;
}
```

###String Literals

리터럴 스트링을 JSX에서 prop으로 바로 사용할 수 있으며, { }로 묶어서도 사용가능 하다. 
리터럴 스트링이 HTML-unescaped 인 경우에도 JSX에서 사용이 가능하다. 

###Props Default to "True"

prop 에 value를 지정하지 않는다며, 기본적으로 “true”로 설정된다. 아래 두 표현은 동일한 결과를 보여준다. 

```
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

{true}와 같은 표현은 추천하지 않는다. ES6의 object shorthand 표현식과 유사하여 혼란을 가져올 수 있다. 
{foo : true} 와 같이 명시적으로 사용하는 방법을 추천. 


###Spread Attributes

prop을 미리 object형태로 선언한 상태라면, JSX에서 넘겨줄때 spread operator **...** 을 사용할 수 있다. 

```
function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

spread operator는 generic container 를 만들떄 유용하게 사용될 수 있다. 그러나 불필요한 props들도 함꼐 전달될 수 있기 때문에, 코드를 지저분하게 만들 수도 있다. 이러한 경우에는 이를 사용하는 것에 대해서 추천하지는 않는다. 

<br>
Children in JSX
-


###String Literals
JSX의 openning tag와 closing tag 사이의 content는 **props.children** 이라는 특별한 prop으로 전달된다. 

```
<MyComponent>Hello world!</MyComponent>
```
위 예제의 MyComponent의 props.children은 “Hello world!” 이다. 

JSX는 opening tag와 closing tag사이의 모든 whitespace, blank line을 제거한다. 

###JSX Children
Multiple JSX element를 children으로 선언하는 것도 가능하며, string literal과 함께 다른 type의 elements를 Mix하여 사용할 수 있다. 
React Component는 Multiple Elements를 리턴할 수 없기 때문에, 반드시 div 같은 tag로 wrapping하여 사용해야 한다. 

###JavaScript Expressions

Javascript Expression은 children으로 {}로 묶어서 사용할 수 있다. 이는 JSX로 임의의 길이의 list를 rendering할때 유용하다. 

```
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
```

###Functions as Children
일반적으로, JSX에 삽입된 Javascript Expression은 string, React element 등으로 판단된다. 그러나 props.children의 경우, React가 어떻게 render할 것있니 알고 있는 것들이 아닌, 일종의 data로 판단되어 전달된다는 점에 있어서, 다른 props과 동일한 것으로 판단된다. 예를 들면, props.children을 통해서, Custom component에 callback function을 전달할 수 있다. 

```
// Calls the children callback numTimes to produce a repeated component
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```
###Booleans, Null, and Undefined Are Ignored

false, null, undefined, true 모두 valid children이지만, render하지 않는다. 

React Element를 조건부 redering 하는 경우, 아래와 같은 예제가 유용할 수 있다. 

```
<div>
  // showHeader === true 인 경우에만, <Header />를 render 한다. 
  {showHeader && <Header />}
  <Content />
</div>
```

조건 식에 숫자 0 같은 “falsy” value에 대해서는 React는 그대로 render 해버린다. 

```
<div>
  {props.messages.length &&
    <MessageList messages={props.messages} />
  }
  // props.messages 가 empty array인 경우 length는 0 이므로, <MessageList />를 ‘0’과 함께 render 한다. 
</div>
```

수정하려면, 아래와 같이 변경해야 한다. 

```
<div>
  {props.messages.length > 0 &&
    <MessageList messages={props.messages} />
  }
</div>
```