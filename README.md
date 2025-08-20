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

> First commit is made with this code in [this github repo](https://github.com/dwi11harsh/v1.webcontainer)
# Requirements for a simple WebContainer

1. A virtual filesystem (to simulate disk operations)
2. Process execution capabilities
3. Terminal interface
4. Package management

# [003] Virtual File System
_lets see how this goes_

1. Create a rust module that implements:
```rust
pub struct VirtualFS {
	files: HashMap<String, Vec<u8>>,
}

impl VirtualFS {
	pub fn new() -> Self {/* */}
	pub fn write_file(&mut self, path: &str, content: &[u8]) -> Result<(), String> {/* */}
	pub fn read_file(&self, path: &str) -> Result<Vec<u8>, String> {/* */}
}
```
_will be adding more methods when I need them_

**instead of trying to access real file system, we will implement a complete virtual file system in assembly**

`lib.rs`
```rust
use std::collections::HashMap;
use wasm_bidgen::prelude::*;

#[wasm_bidgen]
pub struct VirtualFS {
	// track curr virtual dir
	curr_dir: String,
	files: Hashmap<String, Vec<u8>>,
}

#[wasm_bidgen]
impl VirtualFS {
	#[wasm_bidgen(constructor)]
	pub fn new() -> Self {
		VirtualFS {
			curr_dir: "/".to_string(),
			files: HashMap::new(),
		}
	}

	pub fn get_curr_dir(&self) -> {
		self.curr_dir.clone()
	}

	pub fn change_dir(&mut self, path: &str) -> Result<(), String> {
		// basic path validation and update
		if path.starts_with('/') {
			self.curr_dir = path.to_string();
		} else {
			// handle relative paths
			let mut new_path = self.curr_dir.clone();
			if !new_path.ends_with('/') {
				new_path.push('/');
			}
			
			new_path.push_str(path);
			self.curr_dir = new_path;
		}
		Ok(())
	}

	pub fn all_files(&self) -> String {
		format!("current virtual dir: {}", self.curr_dir)
	}
}
```

`index.html`
```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <h1>WebContainer Core</h1>
    <script type="module">
      import init, { VirtualFS } from "./pkg/webcontainer_core.js";
      await init();
      console.log("init successful");
      
      // create virtual file system instance
      const fs = new VirtualFS();
      console.log("Current directory:", fs.get_current_dir());
      
      // change directory in your virtual file system
      fs.change_dir("/home/user");
      console.log("New directory:", fs.all_files());
    </script>
  </body>
</html>
```

> Second commit is made with this code in [this github repo](https://github.com/dwi11harsh/v1.webcontainer)