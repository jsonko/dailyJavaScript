Optimizing Performance
==

React application 성능을 향상 시킬수 있는 몇가지 방법 

Use The Production Build
--

Minified Production build 테스트 방법 

* Create React App 사용시에, npm run build 를 실행하고, 다음 과정을 따른다. 
* Single-file 빌드시, min.js version을 사용한다
* Browserify 사용시, NODE_ENV=production 옵션을 사용하여 빌드한다. 
* Webpack 사용시,  다음과 같은 Plugin을 production config으로 사용한다. 

```
new webpack.DefinePlugin({
  'process.env': {
    NODE_ENV: JSON.stringify('production')
  }
}),
new webpack.optimize.UglifyJsPlugin()
```

Profiling Components with Chrome Timeline
--
Development mode에서, App Load 시, ?react_perf query와 함께 실행하면, Component 별 Timeline을 볼수 있다. 

Avoid Reconciliation
--

Component의 prop이나 state가 변경되었을 때, React는 이전 render된 것과 새로운 element간 비교를 통해서 실제 DOM update가 필요한지 판단한다. 만일 둘이 같지 않다면, React는 DOM을 업데이트 한다. 
특정한 경우, re-rendering process가 시작하기 이전에 트리거 되는 lifecycle function인 shouldComponentUpdate를 Override해서 속도를 높일 수 있다. 
이 함수의 기본 구현은 아래와 같이 true를 리턴한다. 

```
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

만일 component를 업데이트 하기 원치 않은 상황이라면, false를 리턴하면 된다. 

### Examples

```
class CounterButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

이 code에서는 shouldComponentUpdate 내부에서 props.color또는 state.count의 변경이 있는지 체크하고 있다. 그러나 component가 더 복잡해질 수록 비교하는 코드는 어려워 진다. 이를 위하여, props과 state에 대해서 shallow comparison을 수행해주는 React.PureComponent helper를 React는 제공해 준다. 

```
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```
위 코드는 shouldComponentUpdate 대신에 React.PureCompoennt 를 사용한 예제이다. 그러나 PureComponent는 shallow Comparison을 수행하기 때문에, 아래 예제와 같이 Array 내부의 value가 변경되는 경우 같은 변경 사항에 대해서는 비교가 불가능 하다. 

```
class ListOfWords extends React.PureComponent {
  render() {
    return <div>{this.props.words.join(',')}</div>;
  }
}

class WordAdder extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      words: ['marklar']
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // This section is bad style and causes a bug
    const words = this.state.words;
    words.push('marklar');
    this.setState({words: words});
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick} />
        <ListOfWords words={this.state.words} />
      </div>
    );
  }
}
```

### The Power Of Not Mutating Data 

이 문제를 피하기 위한 가장 쉬운 방법은 Prop이나 state 의 value를 변경하는 것을 피하는 것이다. 
handleChick() 메소드를 아래와 같이 concat 을 사용하여 재작성하면 된다. 

```
handleClick() {
  this.setState(prevState => ({
    words: prevState.words.concat(['marklar'])
  }));
}
```

또한, ES6에서 지원하는 spread syntax를 사용하면, 더 쉽게 작성할 수 있다. 

```
handleClick() {
  this.setState(prevState => ({
    words: [...prevState.words, 'marklar'],
  }));
};
```

Object의 Mutation을 피하기 위한 예제는 아래와 같다. 
다음과 같은 object 내 property를 변경하는 함수의 경우, 

```
function updateColorMap(colormap) {
  colormap.right = 'blue';
}
```

Object.assign 메소드를 사용하면, Original object를 mutating을 변경할 수 있다. 

```
function updateColorMap(colormap) {
  return Object.assign({}, colormap, {right: 'blue'});
}
```

spread syntax를 사용하면, 더 쉽게 새로운 Object를 생성할 수 있다. 

```
function updateColorMap(colormap) {
  return {...colormap, right: 'blue'};
}
```

Using Immutable Data Structures
---

Immutable.js를 사용하는 것이 이 문제를 해결하는 또 다른 방법이 될 수 있다. Immutable.js는 immutable, persistent collection을 제공해 준다. 

* Immutable : 한번 생성된 collection은 다른 시점에 변경될 수 있다. 
* Persistent : 이전 collection과 mutation을 함께 포함하여, 새로운 collection을 생성할 수 있다. Original Collection은 새로운 collection이 생성된 이후에도 유지 된다 .
* Structural Sharing : 새로운 collection은 original collection과 가능한 동일한 구조를 사용하여 생성되며, 성능을 향상시키기 위하여 복사를 최소화 한다. 

Immutability는 tracking 비용을 낮춰준다. 변경은 항상 새로운 object에서 발생하기 떄문에, 오직 Object의 refernce가 변경되는지 체크하면 된다. 

```
const x = { foo: "bar" };
const y = x;
y.foo = "baz";
x === y; // true
```

위 코드에서 y는 x와 동일한 object의 reference와 동일하기 때문에, 비교 결과는 true이다. 이와 동일한 코드를 immutable.js로 다시 짤수 있다. 

```
const SomeRecord = Immutable.Record({ foo: null });
const x = new SomeRecord({ foo: 'bar'  });
const y = x.set('foo', 'baz');
x === y; // false
```
이 경우, x에 변경이 발생하였을때, 새로운 reference가 리턴되기 때문에, x가 변경되었다는 것을 안전하게 가정할 수 있다. 


