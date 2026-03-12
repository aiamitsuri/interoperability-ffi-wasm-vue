# interoperability-ffi-wasm-vue

# Initialize Vue project
- npm create vite@latest interoperability-ffi-wasm-vue -- --template vue
- cd interoperability-ffi-wasm-vue
- npm install

# Install package and Wasm plugins
- npm install @aiamitsuri/interoperability-ffi-wasm@0.1.12
- npm install -D vite-plugin-wasm vite-plugin-top-level-await

# Run
- npm run dev
