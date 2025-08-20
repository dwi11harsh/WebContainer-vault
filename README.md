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

_Day 3: aug 20, 2025_
<span style="color:rgb(108, 106, 106)">implementing a virtual file system for the container today</span> 

![[Screenshot 2025-08-20 at 10.23.57 PM.png]]
_not explaining this_

> **My thinking rn:** I am thinking that I should implement this with a two `struct` like something

```txt
struct_1 {
	some ops
	...
		struct_2 {
			some ops
			...
		}
}
```

> The outside struct will be the `VirtualFS` and the inner one will be `Directory`. The reason for this is that some of the ops are performed from outside the directory (like copy, move, delete, etc) and some needs to be performed form inside the directory (like listing all the files and directories, creating a new directory, and some more).
> I have some more things that I am thinking of rn but these will be apparent as I keep going with this idea 🤞. 

> I googled something and here is the result:
![[Screenshot 2025-08-20 at 10.35.40 PM.png]]
> so, I will be trying to implement these within the two structs I talked about and the method distribution will go something like:

<u>Directory methods:</u>
1. `mkdir`
2. `ls`
3. `rm`: for file and for dir
4. `grep`
5. `cat`
6. `head`
7. `touch`

<u>VirtualFS methods:</u>
1. `cd`
2. `cp`
3. `mv`
4. `pwd`
5. `rmdir`
6. `rename`

> I am still not sure of these ops (which one to implement, which one to not, where to put which one)
> will plan and change as I go

