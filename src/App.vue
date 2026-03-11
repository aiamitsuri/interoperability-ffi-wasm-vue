<script setup>
import { ref, onMounted } from 'vue';
import * as wasm from "@aiamitsuri/interoperability-ffi-wasm";

const results = ref(null);
const loading = ref(true);
const error = ref(null);

const loadData = async () => {
  loading.value = true;
  error.value = null;
  
  try {
    await wasm.default();

    const params = {
      language: "node",
      integration: "done",
      crates: "wasm",
      developmentkit: "app",
      page: "1",
      ids: null
    };

    const data = await wasm.fetch_from_js(params);
    console.log("Data received from Rust:", data);
    results.value = data;
  } catch (err) {
    console.error("Wasm Error:", err);
    error.value = err.toString();
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  loadData();
});
</script>

<template>
  <div style="padding: 20px; font-family: sans-serif;">
    <h1>🚩 Interoperability Search (Vue + Rust)</h1>

    <button 
      @click="loadData" 
      :disabled="loading"
      style="margin-bottom: 20px; padding: 8px 16px; cursor: pointer;"
    >
      {{ loading ? 'Refreshing...' : '🔄 Refresh Data' }}
    </button>

    <div v-if="loading && !results">
      <p>🚀 Initializing Rust Wasm...</p>
    </div>

    <div v-else-if="error">
      <p style="color: red;">❌ Error: {{ error }}</p>
    </div>

    <div v-else-if="results">
      <h3>Found {{ results.pagination?.total_items || 0 }} Items</h3>
      <pre style="background: #f4f4f4; padding: 10px; border-radius: 8px; overflow: auto;">
        {{ JSON.stringify(results, null, 2) }}
      </pre>
    </div>

    <div v-else>
      <p>No results found.</p>
    </div>
  </div>
</template>