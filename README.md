> Implementing Stackblitz's webcontainer using rust.
> using first principles only

_Day 2: aug 19, 2025_
<span style="color:rgb(108, 106, 106)">bisi kal ka poora delt kiya, starting from scratch again</span>

# [001] Step 1: Setup

1. add webassembly target
```shell
rustup target add wasm32-unknown-unknown
```

2. install wasm-pack
```shell
cargo install wasm-pack
```

# [002] Step 2: Simple Project

1. initialize
```shell
cargo new webcontainer-core --lib
```

2. update cargo.toml
```toml
[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2"
```

3. `src/lib.rs`
```rust
use wasm_bindgen::prelude::*;
#[wasm_bindgen]
pub fn add(a: i32, b: i32) -> String {
	String::from(format!("the required output is {}", a + b))
}
```

4. compile to web assembly
```shell
wasm-pack build --target web
```

5. create basic web interface to test the code (in root)
```html
<!-- index.html -->
<!DOCTYPE html>

<html>
	<body>
		<input placeholder="input first number" type="number" id="num1" />
		<input placeholder="input second number" type="number" id="num2" />
		<button onclick="{logValue()}">Get Result</button>
		<div id="result"></div>
		<script type="module">
			import init, { add } from "./pkg/webcontainer_core.js";
			const logValue = async () => {
				let num1 = document.getElementById("num1").value;
				let num2 = document.getElementById("num2").value;
				// Initialize the WebAssembly module first
				await init();
				
				// Convert input values to numbers
				num1 = Number(num1);
				num2 = Number(num2);
				let ans = add(num1, num2);
				let resdiv = document.getElementById("result");
				resdiv.innerHTML = ans;
			};
			// Make logValue globally accessible
			window.logValue = logValue;
		</script>
	</body>
</html>
```
_do remember to initialise WebAssembly module before using it_

6. serve the html file
```shell
npx serve .
```
