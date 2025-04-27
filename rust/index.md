# Rust


To run it:
```Shell
rustc list_files.rs -o list_files
./list_files /path/to/directory
```


*list_files.rs*
```Rust
use std::env;
use std::fs;
use std::path::Path;

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() != 2 {
        eprintln!("Usage: {} <directory_path>", args[0]);
        std::process::exit(1);
    }

    let path = Path::new(&args[1]);

    if !path.is_dir() {
        eprintln!("Error: {} is not a directory", path.display());
        std::process::exit(1);
    }

    match fs::read_dir(path) {
        Ok(entries) => {
            println!("Listing files in directory: {}", path.display());
            for entry in entries {
                match entry {
                    Ok(entry) => {
                        println!("{}", entry.file_name().to_string_lossy());
                    }
                    Err(e) => {
                        eprintln!("Error reading entry: {}", e);
                    }
                }
            }
        }
        Err(e) => {
            eprintln!("Error opening directory: {}", e);
            std::process::exit(1);
        }
    }
}
```

