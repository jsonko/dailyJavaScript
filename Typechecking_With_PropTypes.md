Typechecking With PropTypes
=====

React에서는 built-in typechecking 기능인 PropTypes property를 제공한다. 

```
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: React.PropTypes.string
};
```

React.propTypes에서 정의한 것과 다른 type의 props가 제공되었다면, console을 통해서 warning이 출력된다. 
Performance를 위해서 propType은 오작 development mode에서만 사용한다. 

###React.PropTypes

```
MyComponent.propTypes = {

  // Primitive Type 들 
  optionalArray: React.PropTypes.array,
  optionalBool: React.PropTypes.bool,
  optionalFunc: React.PropTypes.func,
  optionalNumber: React.PropTypes.number,
  optionalObject: React.PropTypes.object,
  optionalString: React.PropTypes.string,
  optionalSymbol: React.PropTypes.symbol,

  // redering 될 수 있는 모든 type 을 지정할때 사용 
  optionalNode: React.PropTypes.node,

  // A React element.
  optionalElement: React.PropTypes.element,

  // 특정 객체의 인스턴스 임을 지정할때 사용 
  optionalMessage: React.PropTypes.instanceOf(Message),

  // You can ensure that your prop is limited to specific values by treating
  // it as an enum.
  // 특정 value로 한정하는 경우(enum 처럼 동작함)
  optionalEnum: React.PropTypes.oneOf(['News', 'Photos']),

  // An object that could be one of many types
  // 정의한 여러 타입중 하나 인 경우에 사용 
  optionalUnion: React.PropTypes.oneOfType([
    React.PropTypes.string,
    React.PropTypes.number,
    React.PropTypes.instanceOf(Message)
  ]),

  // An array of a certain type
  // 특정 타입의 Array인 경우 
  optionalArrayOf: React.PropTypes.arrayOf(React.PropTypes.number),

  // An object with property values of a certain type
  // 특정 타입의 property value 를 갖는 객체 임을 제한하는 경우 . 
  // 아래의 경우 number object를 갖는 객체여야 함 
  optionalObjectOf: React.PropTypes.objectOf(React.PropTypes.number),


  // An object taking on a particular shape
  // 특정한 형태의 Object로 제한하는 경우 
  optionalObjectWithShape: React.PropTypes.shape({
    color: React.PropTypes.string,
    fontSize: React.PropTypes.number
  }),

  // You can chain any of the above with `isRequired` to make sure a warning
  // is shown if the prop isn't provided.
  // isRequire가 추가된 prop의 경우 해당 prop이 제공되지 않는 경우 warning을 출력한다. 
  requiredFunc: React.PropTypes.func.isRequired,

  // A value of any data type
  requiredAny: React.PropTypes.any.isRequired,

  // You can also specify a custom validator. It should return an Error
  // object if the validation fails. Don't `console.warn` or throw, as this
  // won't work inside `oneOfType`.
  // Validator 함수를 직접 구현하는 경우. 
  // 직접 console.warn() 같은 함수를 사용하지 않으며, 단지 Error객체를 리턴하면 된다. 
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // You can also supply a custom validator to `arrayOf` and `objectOf`.
  // It should return an Error object if the validation fails. The validator
  // will be called for each key in the array or object. The first two
  // arguments of the validator are the array or object itself, and the
  // current item's key.
  customArrayProp: React.PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```


###Requiring Single Child

React.PropTypes.element를 사용하면, children으로 오직 single child 만 넘겨받는 것으로 지정할 수 있다. 

```
class MyComponent extends React.Component {
  render() {
    // This must be exactly one element or it will warn.
    // this.props.children은 오직 하나의 element 형태여야 한다. 
    // 그렇지 않으면 warn 발생. 
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children: React.PropTypes.element.isRequired
};
```

###Default Prop Values

defaultProps property를 사용하면, props의 default값을 설정할 수 있다. 


```
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// Specifies the default values for props:
Greeting.defaultProps = {
  name: 'Stranger'
};

// Renders "Hello, Stranger":
ReactDOM.render(
  <Greeting />,
  document.getElementById('example')
);
```
defaultProps 는 Parent Element로 부터 props 값을 넘겨 받지 못하는 경우, 값을 지정하기 위해서 사용된다. 
PropsType에 의한 type checking은 이 defaultProps가 처리된 이후에 적용된다. 