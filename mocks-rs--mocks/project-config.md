---
trigger: always_on
description: Testing approaches and patterns for mocks project
---


# Testing Strategy for Mocks Project

This rule defines comprehensive testing patterns for the mocks CLI tool, covering unit tests, integration tests, and end-to-end testing.

## Unit Testing Patterns

### Module-Level Testing
```rust
// Place unit tests within each module using #[cfg(test)]
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_parse_socket_addr_localhost() {
        let result = parse_socket_addr("localhost", 3000).unwrap();
        assert_eq!(result.ip().to_string(), "127.0.0.1");
        assert_eq!(result.port(), 3000);
    }

    #[test]
    fn test_parse_socket_addr_ip() {
        let result = parse_socket_addr("192.168.1.1", 8080).unwrap();
        assert_eq!(result.ip().to_string(), "192.168.1.1");
        assert_eq!(result.port(), 8080);
    }

    #[test]
    fn test_parse_socket_addr_invalid() {
        let result = parse_socket_addr("invalid.host", 3000);
        assert!(result.is_err());
        
        if let Err(MocksError::InvalidArgs(msg)) = result {
            assert!(msg.contains("invalid"));
        } else {
            panic!("Expected InvalidArgs error");
        }
    }
}
```

### Testing Error Paths
```rust
#[cfg(test)]
mod error_tests {
    use super::*;

    #[test]
    fn test_error_display() {
        let error = MocksError::InvalidArgs("test error".to_string());
        assert_eq!(error.to_string(), "Invalid arguments: test error");
    }

    #[test]
    fn test_error_conversion() {
        let status: StatusCode = MocksError::ResourceNotFound("users".to_string()).into();
        assert_eq!(status, StatusCode::NOT_FOUND);
    }

    // Test all error variants
    #[test]
    fn test_all_error_types() {
        let errors = vec![
            MocksError::ResourceNotFound("test".to_string()),
            MocksError::InvalidArgs("test".to_string()),
            MocksError::InvalidJson("test".to_string()),
            MocksError::FailedReadFile("test".to_string()),
            MocksError::FailedWriteFile("test".to_string()),
            MocksError::Exception("test".to_string()),
            MocksError::Aborted,
        ];

        for error in errors {
            // Ensure all errors implement required traits
            let _display = error.to_string();
            let _debug = format!("{:?}", error);
        }
    }
}
```

### Testing Async Functions
```rust
#[cfg(test)]
mod async_tests {
    use super::*;
    use tokio::test;

    #[tokio::test]
    async fn test_server_startup_invalid_address() {
        use std::net::{IpAddr, Ipv4Addr};
        
        let invalid_addr = SocketAddr::new(
            IpAddr::V4(Ipv4Addr::new(999, 999, 999, 999)), 
            3000
        );
        let storage = create_test_storage().await;
        
        let result = Server::startup(invalid_addr, storage).await;
        assert!(result.is_err());
    }
}
```

## Integration Testing Patterns

### Using Temporary Files
```rust
#[cfg(test)]
mod integration_tests {
    use super::*;
    use tempfile::{NamedTempFile, TempDir};
    use std::io::Write;

    fn create_test_storage_file() -> NamedTempFile {
        let mut temp_file = NamedTempFile::new().unwrap();
        let test_data = serde_json::json!({
            "posts": [
                {"id": 1, "title": "Test Post", "author": "test"}
            ],
            "profile": {
                "name": "testuser"
            }
        });
        
        write!(temp_file, "{}", serde_json::to_string_pretty(&test_data).unwrap()).unwrap();
        temp_file
    }

    #[test]
    fn test_storage_creation() {
        let temp_file = create_test_storage_file();
        let path = temp_file.path().to_str().unwrap();
        
        let storage = Storage::new(path, true).unwrap();
        assert_eq!(storage.file, path);
        assert!(storage.overwrite);
        
        // Verify data was loaded correctly
        let resources = storage.resources();
        assert!(resources.contains(&"posts".to_string()));
        assert!(resources.contains(&"profile".to_string()));
    }

    #[test]
    fn test_storage_operations() {
        let temp_file = create_test_storage_file();
        let path = temp_file.path().to_str().unwrap();
        
        let mut storage = Storage::new(path, true).unwrap();
        
        // Test insert operation
        let new_post = serde_json::json!({
            "title": "New Post",
            "author": "newuser"
        });
        
        let result = insert(&mut storage.data, "posts", new_post).unwrap();
        assert!(result.get("id").is_some());
        
        // Test select operation
        let all_posts = select_all(&storage.data, "posts").unwrap();
        assert!(all_posts.as_array().unwrap().len() >= 2);
    }
}
```

### Testing File I/O Operations
```rust
#[cfg(test)]
mod file_io_tests {
    use super::*;
    use tempfile::TempDir;
    use std::fs;

    #[test]
    fn test_writer_atomic_operation() {
        let temp_dir = TempDir::new().unwrap();
        let file_path = temp_dir.path().join("test.json");
        let path_str = file_path.to_str().unwrap();
        
        let test_data = serde_json::json!({"test": "data"});
        let writer = Writer::new(path_str);
        
        // Write data
        writer.write(&test_data).unwrap();
        

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mocks-rs/mocks](https://github.com/mocks-rs/mocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
