# 1.6 JavaScript import 하기

이 페이지에서는 코드에 대해서 상세히 설명하지 않습니다. 다만 웹어셈블리에서 자바스크립트를 호출하는 것이 가능한지만 확인합니다.  이 페이지에서 사용한 원본 코드는 [여기](https://github.com/rustwasm/wasm-bindgen/tree/main/examples/import_js)에서 확인할 수 있습니다. 원본 글은 [js-sys: WebAssembly in WebAssembly](https://rustwasm.github.io/docs/wasm-bindgen/examples/wasm-in-wasm.html)은 여기에서 확인할 수 있습니다.


> 이 페이지의 전체 소스는 [여기](https://github.com/latteonterrace/rust-wasm-importjs.git)를 참고합니다. 



git에서 clone하여 생성하였기에 다음과 같이 실행하여 초기화 합니다. 
```shell
cargo init
```

우리는 하나의 html 파일 wasm 으로 빌드될 lib.rs 파일, 그리고 임포트할 자바스크립트 파일을 생성합니다. 다음과 같을 것입니다. 


```shell
📂project-root
├── 📂src
│   └── 📄lib.rs
├── 📄defined-in-js.js
├── 📄index.html
└── 📄Cargo.toml
└── 📄package.json
```

Cargo.toml 파일은 다음과 같이 작성합니다.
**Cargo.toml**
```toml
[package]
name = "import_js"
version = "0.1.0"
authors = ["The wasm-bindgen Developers"]
edition = "2018"

[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2.83"
```

defined-in-js.js 파일은 다음과 같이 작성합니다. name() 함수와 MyClass 클래스를 정의합니다. 

**defined-in-js.js**

```javascript 
export function name() {
  return 'Rust';
}

export class MyClass {
  constructor() {
      this._number = 42;
  }

  get number() {
      return this._number;
  }

  set number(n) {
      return this._number = n;
  }

  render() {
      return `My number is: ${this.number}`;
  }
}
```

lib.rs 파일을 다음과 같이 정의합니다. '#[wasm_bindgen(module = "/defined-in-js.js")] 을 사용하여 자바스크립트 파일을 가져옵니다. 이것은 자바스크립트 파일에서 정의된 모든 기능을 가져옵니다.

\#[wasm_bindgen] 속성은 JS에서 기능을 가져오기 위해 extern 'C' { .. } 블록에서 사용할 수 있습니다. 이것이 js-sys 및 web-sys 크레이트가 구축되는 방식이지만 자신의 크레이트에서도 사용할 수 있습니다!

**lib.rs**
```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen(module = "/defined-in-js.js")]
extern "C" {
    fn name() -> String;

    type MyClass;

    #[wasm_bindgen(constructor)]
    fn new() -> MyClass;

    #[wasm_bindgen(method, getter)]
    fn number(this: &MyClass) -> u32;
    #[wasm_bindgen(method, setter)]
    fn set_number(this: &MyClass, number: u32) -> MyClass;
    #[wasm_bindgen(method)]
    fn render(this: &MyClass) -> String;
}

// lifted from the `console_log` example
#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
}

#[wasm_bindgen(start)]
pub fn run() {
    log(&format!("Hello from {}!", name())); // should output "Hello from Rust!"

    let x = MyClass::new();
    assert_eq!(x.number(), 42);
    x.set_number(10);
    log(&x.render());
}
```


위 코드에서 console.log()를 사용하여 name() 함수를 호출한 결과를 출력합니다.  JavaScript의 클래스와 대응되는 MyClass 인스턴스를 생성하고, number() getter를 호출하여 42를 반환하는지 확인합니다. 그런 다음 set_number() setter를 호출하여 10으로 설정하고 render() 메서드를 호출하여 "My number is: 10"을 반환하는지 확인합니다.

다음을 실행하여 pkg를 생성합니다. 
```shell
wasm-pack build --target web
```

index.html 파일을 생성하고 다음과 같이 입력합니다.   pkg 디렉토리에 생성된 import_js.js 파일을 가져옵니다. 그런 다음 init() 함수를 호출하여 WASM을 초기화합니다. 이것은 WASM이 로드되고 초기화되기 전에는 WASM에서 정의된 모든 기능을 사용할 수 없음을 의미합니다.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <script type="module">
    import init  from './pkg/import_js.js';

    async function run() {
      await init();
    }
    run();
  </script>
</script>  

</body>
</html>
```

VSCode를 사용한다면 Open With Live Server를 실행하여 브라우저에서 콘솔 로그를 확인합니다. 










