
프로토타입 체이닝 
=============

두가지 의미의 프로토타입
------------------

자바스크립트에서 모든 객체는 자신을 생성한 생성자 함수의 prototype프로퍼티가 가리키는 **프로토타입 객체** 를
자신의 부모 객체로 설정하는 **[[Prototype]]** 링크 로 연결한다. 

```javascript
function Person(name) {
	this.name = name;
}

var foo = new Person(‘foo’);
```

위의 경우, Person() 생성자 함수는 prototype 프로퍼티로 자신과 링크된 프로토타입 객체(Person.prototype)를 가리킨다. 
Person() 생성자 함수로 생성된 foo 객체는 Person() 함수의 프로토타입 객체를 [[Prototype]] 링크로 연결한다. 

**자바스크립트에서 객체를 생성하는 건 생성자 함수의 역할이지만, 생성된 객체의 실제 부모 역할을 하는 건 생성자 자신이 아닌 생성자의 prototype 프로퍼티가 가리키는 프로토타입 객체이다.**

<br>

프로토타입 체이닝
---------------------------------------

자신의 부모 역할을 하는 프로토타입 객체의 프로퍼티 또한 마치 자신의 것처럼 접근이 가능하다. 이를 가능하게 하는 것이 **프로토타입 체이닝** 이다. 

객체 리터럴로 생성한 객체는 **Object()**라는 내장 생성자 함수로 생성된 것이므로, Object가 갖고 있는 prototype 프로퍼티 속성이 가르키는 프로토타입 객체, **Object.prototype** 객체를 자신의 프로토타입 객체로 연결한다. 

**프로토타입 체이닝**이란, 특정 객체의 프로퍼티나 메서드에 접근하려고 할때, 해당 객체에 해당 프로퍼티나 메서드가 없으면,
**[[Prototype]] 링크**를 따라 자신의 부모 역할을 하는 프로토타입 객체의 프로퍼티를 차례대로 검색하는 것을 말한다. 

일반 생성자 함수로 생성된 객체의 경우에는 생성자 함수의 prototype 프로퍼티가 가리키는 객체를 자신의 프로토타입 객체로 취급한다. 

```javascript
function Person(name, age, hobby){
	this.name = name;
	this.age = age;
	this.hobby = hobby;
}

var foo = new Person(‘foo’, 30, ‘tennis’);
```

위 예제의 경우, foo 객체의 생성자는 Person()이 되고, foo 객체의 프로토타입 객체는 Person.prototype이 된다. 

자바스크립트에서 Object.prototype 객체는 프로토타입 체이닝의 종점으로, 모든 자바스크립트 객체는 프로토타입 체이닝으로 Object.prototype 객체가 가진 프로퍼티와 메서드에 접근하고, 공유가 가능하다. 

Object.prototype에는 자바스크립트의 표준 메소드들이 정의 되어 있다. 

Number.prototype, String.prototype, Array.prototype 등에도 숫자, 문자열, 배열에서 사용되는 표준 메소드들이 정의되어 있다. 

이와 같은 표준 빌트인 프로토타입 객체에도 사용자가 직접 정의한 메서드들을 추가하는 것을 허용한다. 

함수가 생성될 때, 자신의 prototype 프로퍼티에 연결되는 프로토타입 객체는 디폴트로 constructor 프로퍼티만 갖는 객체가 되며, 당연히 이 프로토타입 객체에도 동적으로 프로퍼티 추가/삭제하는 것이 가능한다. 

```javascript
function Person(name, age, hobby){
	this.name = name;
	this.age = age;
	this.hobby = hobby;
}

var foo = new Person(‘foo’, 30, ‘tennis’);

Person.prototype.sayHello = function() {
	console.log(‘Hello’);
}

foo.sayHello();
```

프로토타입 메서드에서 this를 사용하는 경우, 해당 this는 그 메서드를 호출한 객체에 바인딩된다. 

Default Prototype 객체도 다른 일반 일반 객체로 변경하는 것이 가능하다. 

```javascript
function Person(name){
	this.name = name;
}
console.log(Person.prototype.constructor); // Person(name)
// 디폴트 프로토타입 객체는 construtor만을 가진다. 

var foo = new Person(‘foo’);
console.log(foo.country); 
// foo는 기본 Person 생성자에 의해서 생성된 객체이고, 
// foo 객체와 foo 의 프로토타입 객체에는 country가 존재하지 않으니, undefined가 출력됨 

Person.prototype = {
	country : ‘korea’
};
console.log(Person.prototype.constructor);
// Person의 prototype 객체를 country 프로퍼티만 있는 리터럴 객체로 재 정의 했다. 
// 따라서, prototype객체 내에는 constructor는 존재하지 않으므로 출력은 “undefined”가 된다. 
// 리터럴 객체의 prototype link는 Object.prototype 을 가리키게 된다. 

var bar = new Person(‘bar’);
// 변경된 Person.prototype을 통해서 bar 객체를 생성함. 

console.log(foo.country); // foo는 여전히 country가 없으므로, undefined 출력
console.log(bar.country); // Korea 출력됨 
console.log(foo.constructor); // Person(name)이 출력됨 
console.log(bar.constructor); // Object()가 출력됨 
```




