Uncontrolled Components
==

Controlled Components에서는 form 구현시에, form data처리를 React Component를 통해서 처리한다. 
그러나 Uncontrolled Component에서는 DOM 자체적으로 form data를 처리할 수 있다. 

Uncontrolled Component를 구현하기 위해서, 모든 state 업데이트에 대한 event handler를 작성하는 것 대신에, **ref**를 사용해서 DOM으로 부터 form value를 가져올 수 있다. 

```javascript
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
  }
  
  // input tag의 value를 ref를 통해서 직접 가져온다. 
  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={(input) => this.input = input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

Uncontrolled component는 DOM source 정보를 유지하기 때문에, React와 non-React code 를 통합하는데 용이함을 제공한다. 이 방식은 빠르나 지저분한 code가 될 수 있으므로, Controlled Component를 사용할 것을 추천한다. 


###Default Value

React rendering lifecycle에서, form element의 value attribute은 DOM 안의 value를 Override하게된다.. 그리고Uncontrolled component에서 value의 초기값을 지정하기 원하는 경우에도, 이어진 uncontrolled update에 대해서는 그대로 나둬야 한다. 이러한 케이스를 지원하기 위해서, value대신에 defaultValue attribute을 지정할 수 있다. 

```javascript
render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <label>
        Name:
        <input
          defaultValue="Bob"
          type="text"
          ref={(input) => this.input = input} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}
```

<input type=“checkbox”>와 <input type=“radio”>는 defaultChecked를 지원하고, <select>는 defaultValue를 지원한다. 