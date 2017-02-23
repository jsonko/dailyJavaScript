Highier-Order Components
==

Higher-order Component는 Component 재사용을 위한 React advanced technique 이다. 
HOC는 React API의 일부는 아니다. 그러나 React Compositional 특성으로 부터 떠오르는 패턴이다. 

구체적으로, higher-order component는 component를 입력 받아서, 새로운 component를 리턴하는 함수이다. 

```
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

component가 prop을 UI로 변환하는 것에 비해서, higher-order component는 component를 또 다른 component로 변환한다. 

HOC는 Redux의 connect, Relay의 createContainer같은 third-party React 라이브러리에서 일반적으로 사용된다. 
앞으로 왜 HOC가 유용하고 어떻게 작성하는지 알아보자. 

Use HOCs For Crossing-Cutting Concerns
--

#### Note

*이전에 cross-cutting concern을 처리 할 수있는 방법으로 mixin을 추천하였으나, mixin이 생각보다 문제를 많이 발생한다는 것을 알게되었다. 앞으로 mixin 을 사용하지 않게 되었는지와, 기존의 component를 어떻게 변환하는지에 대해서 보여주겠다.*

Components는 React에서 재 사용되는 주요 code 단위이다. 그러나 어떤 패턴들은 기존 component에 정확하게 맞지 않는 경우가 있을 것이다. 예를 들면, 외부 테이터 소스를 사용해서 comments list 를 그리는 *CommentList*라는 Component가 아래와 같이 있다. 

```
class CommentList extends React.Component {
  constructor() {
    super();
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // "DataSource" is some global data source
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // Subscribe to changes
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // Clean up listener
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // Update component state whenever the data source changes
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}

```
이 후에, 당신이 single blog post를 구독하는 component를 작성하였고, 그 component가 유사한 패턴을 따르고 있다면, 

```
class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id)
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id)
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
}
```
CommentList와 BlogPost는 동일하지 않다. DataSorce의 다른 Method를 호출하고 있고, 다른 output을 rendering한다. 그러나 다음 구현은 동일하다. 

* 마운트시, DataSource에 listener를 등록한다. 
* listener 내부에서, dataSource가 변경될 떄마다, setState를 호출한다. 
* Unmount시, change listener를 제거한다. 

Large app 에서 봤을때, DataSource를 subscribing하고, 변할때마다 setState를 호출하는 같은 패턴이 계속 반복해서 일어날 것이다. 우리는 이 로직이 한군데서 발생할 수 있도록 추상화할 필요가 있고, 다른 component들에게 공유하고자 한다. 이 것에 대해서 HOC가 탁월한 선택이 된다.

우리는 CommnetList와 BlogPost와 같이 DataSource를 subscribe하는 components를 생성하는 함수를 만들수 있다. 이 함수는 argument 중 하나로 child component를 받게 되며, 이 child component는 prop으로 subscribed data를 받게 된다. *withSubscription* 함수를 호출해 보자. 

```
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
});
```

첫번째 파라메터는 wrapped component이고, 두번째 파라메터는 Data Source와 current props내에서 우리가 관심있는 data를 찾는다. 

*CommentListWithSubscription* 와 *BlogPostWithSubscription* 함수가 렌더될때, CommentList와 BlogPost는 DataSource로 부터 가져온 current data 와 함께 data prop으로 전달되게 될 것 이다. 

```
// This function takes a component...
function withSubscription(WrappedComponent, selectData) {
  // ...and returns another component...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ... that takes care of the subscription...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

HOC는 input component자체를 수정하지 않는다는 점을 주목해야 하며, input component의 동작을 복사하기 위해서 상속을 사용하지도 않는다. HOC는 container component 안에 original component를 wrappping하여 작성한다. HOC는 side-effect가 없는 순수 함수이다. 

그리고 그게 다야! 래핑 된 component는 container의 모든 props와 함께 output을 렌더링하는 데 사용되는 새로운 prop, data를 받는다. HOC는 데이터가 사용되는 방법 또는 이유와 관련이 없으며 랩핑 된 구성 요소는 데이터가 어디서 왔는지와 관련이 없습니다.

withSubscription은 일반 함수이기 때문에 원하는만큼 인수를 추가 할 수 있습니다. 예를 들어 랩핑 된 component에서 HOC를 더 분리시키기 위해 데이터 component의 이름을 구성 가능하게 만들 수 있습니다. 또는 shouldComponentUpdate를 구성하는 인수 또는 데이터 소스를 구성하는 인수를 허용 할 수 있습니다. HOC는 구성 요소 정의 방법을 완전히 제어 할 수 있기 때문에 이러한 모든 작업이 가능합니다.

구성 요소와 마찬가지로 withSubscription과 래핑 된 component 간의 연결은 완전히 props 기반입니다. 이는 하나의 HOC를 다른 걸로 변경이 쉽게 만들어 줍니다. 이렇게하면 래핑 된 Component에 동일한 소품을 제공하는 한 다른 HOC를 쉽게 교체 할 수 있습니다. 예를 들어 data-fetching하는 라이브러리를 변경하는 경우 유용 할 수 있습니다.


Don't Mutate the Original Component. Use Composition.
--

HOC내의 component의 prototype을 수정하고 싶은 유혹에 대해서 저항하라, 

```
function logProps(InputComponent) {
  InputComponent.prototype.componentWillReceiveProps(nextProps) {
    console.log('Current props: ', this.props);
    console.log('Next props: ', nextProps);
  }
  // The fact that we're returning the original input is a hint that it has
  // been mutated.
  return InputComponent;
}

// EnhancedComponent will log whenever props are received
const EnhancedComponent = logProps(InputComponent);
```

여기는 몇가지 문제가 있다. 하나는 input component는 enhanced component로 부터 따로 재 사용될 수 있다는 것이다. 더 중요한 것은, 만일 네가 다른 HOC에 EnhancedComponent를 적용한다면(그 HOC도 componentWillReceiveProps를 수정한다면), 첫번째 HOC의 기능은 override되버릴 것이다. 이 HOC는 제대로 동작하지 않을것이다.  

HOC를 변경하는 것은 좋지 않은 추상화 방법이다. 사용자는 다른 HOC간에 충돌을 피하기위해서 그것들이 어떻게 구현되어 있는지 반드시 알아야 된다. 

변경 대신에, HOC는 composition을 사용해야 하며, container component안에 input component를 wrapping해야 한다. 

```
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it. Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

이 HOC는 mutating 버전과 동일한 기능을 갖으며, 잠재적인 clash 피할 수 있다. class와 functional components와 함께 동일하게 동작할 수 있다. 그리고 Pure function이기 때문에, 다른 HOC와 함께 또는 개별적으로 조합되는데 잘 사용될 수 있다. 

HOC와 container components와 비슷한 점을 알게 되었을 것이다. Container compoennt는 High-level과 Low-level의 concern 사이의 책임을 분산 시키기위한 전략 중일부이다. Container는 subscription과 state 같은 것들을 관리하고, UI를 rendering하기 위한 props을 전달한다. HOC는 그 구현의 일부로서 container를 사용한다. HOC를 Parameterized container component로 정의할 수도 있겠다. 

Convention: Pass Unrelated Props Through to the Wrapped Component
---

HOC는 component에 기능을 추가한다. 그들은 component의 본래 기능을 심하게 바꾸지 않는다. HOC에는 wrap 된 component와 유사한 interface가 존재해야 한다. 

HOC는 특정 관심사와 관련이 없는 props을 전달해야한다. 대부분의 HOC에는 다음과 같은 render 메소드가 있다. 

```
render() {
  // Filter out extra props that are specific to this HOC and shouldn't be
  // passed through
  const { extraProp, ...passThroughProps } = this.props;

  // Inject props into the wrapped component. These are usually state values or
  // instance methods.
  const injectedProp = someStateOrInstanceMethod;

  // Pass props to wrapped component
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```

Convention: Maximizing Composability
---

모든 HOC가 동일하지는 않다. 떄때로,HOC는 wrapped component 하나만 argument로 받는다. 

```
const NavbarWithRouter = withRouter(Navbar);
```

보통, HOC는 추가 argument를 받는다, 이어지는 예제를 보면, config object는 component의 data dependency를 정의하기 위하여 사용되었다. 

```
const CommentWithRelay = Relay.createContainer(Comment, config);
```

대부분의 HOC의 일반적인 시그니쳐는 아래와 같다. 

```
// React Redux's `connect`
const ConnectedComment = connect(commentSelector, commentActions)(Comment);
```

조금 분해해서 보면 이해할 수 있을 것이다. 

```
// connect is a function that returns another function
// connect는 다른 함수를 리턴하는 함수이다. 
const enhance = connect(commentListSelector, commentListActions);

// The returned function is an HOC, which returns a component that is connected
// to the Redux store
// 리턴된 함수는 HOC이며, Redux Store에 연결되는 component를 리턴한다. 
const ConnectedComment = enhance(CommentList);
```

다시 말하자면, connect는 HOC를 리턴하는 higher-order function이다. 

이 형태는 헷갈리고 불필요하게 보이지만, 유용한 property 를 갖고 있다. connect 함수에 의해서 리턴되는 Single-argument HOC 는 Component => Component 라는 signature를 갖고 있다. 출력 타입이 입력타입과 동일한 함수는 서로 compose 하기가 쉽다. 

```
// Instead of doing this...
const EnhancedComponent = connect(commentSelector)(withRouter(WrappedComponent))

// ... you can use a function composition utility
// compose(f, g, h) is the same as (...args) => f(g(h(...args)))
const enhance = compose(
  // These are both single-argument HOCs
  connect(commentSelector),
  withRouter
)
const EnhancedComponent = enhance(WrappedComponent)
```

이 compose utilty는 여러 third party 에서 제공합니다. 


CEVEATS
--

고차원 구성 요소에는 React를 처음 접하는 사람이라면 즉시 알 수없는 몇 가지주의 사항이 있습니다.

Don't Use HOCs Inside the render Method
---

```
render() {
  // A new version of EnhancedComponent is created on every render
  매번 render시마다 새로운 버전의 EnhancedComponent를 생성함. 
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // That causes the entire subtree to unmount/remount each time!
  return <EnhancedComponent />;
}
```


Static Methods Must Be Copied Over
---

React 구성 요소에 정적 메서드를 정의하는 것이 유용 할 때가 있습니다. 예를 들어, Relay 컨테이너는 정적 메서드 getFragment를 노출하여 GraphQL 조각의 구성을 용이하게합니다.

그러나 구성 요소에 HOC를 적용하면 원래 구성 요소가 컨테이너 구성 요소로 랩핑됩니다. 즉, 새 구성 요소에는 원래 구성 요소의 정적 메서드가 없습니다.

```
// Define a static method
WrappedComponent.staticMethod = function() {/*...*/}
// Now apply an HOC
const EnhancedComponent = enhance(WrappedComponent);

// The enhanced component has no static method
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

이를 해결하기 위해, return하기 전에 container안에서 method를 복사해주면 된다. 

```
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // Must know exactly which method(s) to copy :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```

그러나 이렇게하려면 어떤 method를 복사해야하는지 정확히 알아야합니다. ‘hoist-non-react-statics’를 사용하여 모든 non-react static method를 자동으로 복사 할 수 있습니다 :

```
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

또다른 방법은 static method를 component에서 분리해서 따로 export하는 방법입니다. 

```
// Instead of...
MyComponent.someFunction = someFunction;
export default MyComponent;

// ...export the method separately...
export { someFunction };

// ...and in the consuming module, import both
import MyComponent, { someFunction } from './MyComponent.js';
```

Refs Aren't Passed Through
---

HOC의 convensiont이 모든 props은 wrapped component로 전달한다라는 것에 반면에, refs를 넘길수는 없다. 이는 ref가 key와 같은 prop이 아니고, React에서 특별히 다루는 것이기 때문ㅇ이다. HOC에서 리턴하는 component의 내 element의 ref을 추가하고 싶다면, ref는 wrapped component가 아닌 외부 container component의 instatnce의 ref를 참조하게 된다. 

이상적인 해결책은 ref를 사용하지 않는 것이다. 

그렇다면 refs가 필요한 escape hatch 일 때가 있습니다 - React는 그렇지 않은 경우 지원하지 않을 것입니다. 입력 필드에 초점을 맞추는 것은 구성 요소의 필수 제어가 필요한 경우를 예로들 수 있습니다. 이 경우 한 가지 해결책은 다른 이름을 지정하여 일반 소품으로 ref 콜백을 전달하는 것입니다.

```
function Field({ inputRef, ...rest }) {
  return <input ref={inputRef} {...rest} />;
}

// Wrap Field in a higher-order component
const EnhancedField = enhance(Field);

// Inside a class component's render method...
<EnhancedField
  inputRef={(inputEl) => {
    // This callback gets passed through as a regular prop
    this.inputEl = inputEl
  }}
/>

// Now you can call imperative methods
this.inputEl.focus();
```
