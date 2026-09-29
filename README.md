# Product Service

The Product Service is a simple web service built using Rust and the Warp web framework. It is responsible for serving the product catalog, which includes a list of products that can be fetched via a RESTful API.

## Requirements

- Rust (latest stable version) and Cargo, installed below
- Start inside the repository's `product-service` directory. The main guide already takes you there.

## Setup Instructions

1. Update the package list and install the build tools:

   ```bash
   sudo apt update
   sudo apt install build-essential
   ```

2. Install Rust and accept the default installation:

   ```bash
   curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
   ```

3. Load the Rust tools into this terminal:

   ```bash
   source "$HOME/.cargo/env"
   ```

4. Build and start the service:

   ```bash
   cargo run
   ```

   Keep this terminal open. Cargo prints build output; the application itself does not print a listening message. Do not start a second copy from the main guide.

The service binds to `0.0.0.0:3030` (all IPv4 interfaces). On the VM, test `http://localhost:3030/products`. From your laptop, use `http://<VM-PUBLIC-IP>:3030/products` with port 3030 allowed by the NSG. VS Code port forwarding is an optional alternative for accessing a forwarded port through your laptop's localhost.

## Testing

From another terminal:

```bash
curl http://localhost:3030/products
```

Expect three products with IDs, names, and prices. You can also install the VS Code **REST Client** extension and run `test-product-service.http`.
