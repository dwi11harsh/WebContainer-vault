```rust
#[derive(Debug, Clone)]

pub struct Directory {
	// name of this directory (not the full path)
	pub name: String,
	// optional parent directory none for root)
	pub parent: Option<Box<Directory>>,
	// child directories by name
	pub children: HashMap<String, Directory>,
	// files in this directory by name
	pub files: HashMap<String, Vec<u8>>,
}
```
# mkdir
```rust
impl Directory {
	// constructor
	pub fn mkdir(name: String, parent: Option<Box<Directory>>) -> Self {
	
		Directory {
			name,
			parent,
			children: HashMap::new(),
			files: HashMap::new(),
		}
}
```

# ls
```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
pub enum FileType {
	Dir,
	File,
}
...

pub fn ls(&self) -> HashMap<FileType, String> {
	let mut list: HashMap<FileType, String> = HashMap::new();
	// directory names
	for (name, _) in &self.children {
		list.insert(FileType::Dir, name.clone());
	}
	// file names
	for (name, _) in &self.files {
		list.insert(FileType::File, name.clone());
	}
	list
}
```

