Web Components
==

React와 Web Component는 다른 문제를 해결하기 위하여 만들어진 것이다. Web Component는 컴포넌트 재사용성을 위한 강력한 캡슐화를 제공하는 반면에, React는 데이터와 동기화 되는 DOM을 유지하기 위한 Library를 제공한다. 이 2가지 목적은 서로 보완적인 면이 있다. 개발자는 편하게 Web Component를 위하여 React를 사용할 수 있고, Web Component 내에 React를 사용할 수 있다. 

React를 사용하는 대부분의 사람들은 Web Component를 사용하지 않지만, Web Component를 사용하는 third-party UI component를 사용하는 사람들은 필요할 수 있다. 

Using Web Components in React
--

```
class HelloMessage extends React.Component {
  render() {
    return <div>Hello <x-search>{this.props.name}</x-search>!</div>;
  }
}
```

####Note: 
Web Components 는 imperative API를 제공하는 경우가 있다. 예를 들면 video Web Component의 경우, play(), pause() 함수를 제공한다. Web Component의 imperative API를 접근 하기 위해서, DOM node에 직접 접근해서 사용하는 ref 를 사용할 수 있을 것이다. 만일 Third-party Web Component를 사용한다면, 가장 좋은 방법은 그 Component를 React Component 형태로 Wrapping하는 것이다. 

헷갈릴 수 있는 부분 중 하나는 Web Components는 “className” 대신에 class를 사용한다는 점이다. 

```
function BrickFlipbox() {
  return (
    <brick-flipbox class="demo">
      <div>front</div>
      <div>back</div>
    </brick-flipbox>
  );
}
```

Using React in your Web Components
---

```
const proto = Object.create(HTMLElement.prototype, {
  attachedCallback: {
    value: function() {
      const mountPoint = document.createElement('span');
      this.createShadowRoot().appendChild(mountPoint);

      const name = this.getAttribute('name');
      const url = 'https://www.google.com/search?q=' + encodeURIComponent(name);
      ReactDOM.render(<a href={url}>{name}</a>, mountPoint);
    }
  }
});
document.registerElement('x-search', {prototype: proto});
```