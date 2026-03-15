<script setup>
import { ref, onMounted, watch } from 'vue';
import * as wasm from "@aiamitsuri/interoperability-ffi-wasm";

const results = ref(null);
const loading = ref(true);
const error = ref(null);
const page = ref(1);

const loadData = async () => {
  loading.value = true;
  try {
    await wasm.default();
    const params = {
      language: null,
      integration: null,
      crates: null,
      developmentkit: null,
      page: page.value.toString(),
      ids: null,
    };
    const data = await wasm.fetch_from_js(params);
    results.value = data;
  } catch (err) {
    error.value = err.toString();
  } finally {
    loading.value = false;
  }
};

watch(page, () => loadData());
onMounted(() => loadData());

const handleNext = () => page.value++;
const handlePrev = () => { if (page.value > 1) page.value--; };
</script>

<template>
  <div style="padding: 20px; font-family: sans-serif; text-align: left; max-width: 1000px;">
    <h1 style="margin-top: 0;">Interoperability Search</h1>

    <div v-if="loading && !results">🚀 Initializing Rust Wasm...</div>
    <div v-else-if="error" style="color: red;">❌ Error: {{ error }}</div>

    <div v-else>
      <!-- Controls Layout -->
      <div style="margin-bottom: 20px; display: flex; gap: 12px; align-items: center; justify-content: flex-start;">
        <button @click="loadData" :disabled="loading" style="padding: 6px 12px; cursor: pointer;">
          {{ loading ? 'Refreshing...' : '🔄 Refresh' }}
        </button>

        <button 
          @click="handlePrev" 
          :disabled="loading || !results?.pagination?.prev_page_url"
          style="padding: 6px 12px; cursor: pointer;"
        >
          Previous
        </button>

        <span style="font-weight: bold;">
          Page {{ results?.pagination?.current_page }} / {{ results?.pagination?.total_pages }}
        </span>

        <button 
          @click="handleNext" 
          :disabled="loading || !results?.pagination?.next_page_url"
          style="padding: 6px 12px; cursor: pointer;"
        >
          Next
        </button>
      </div>

      <div v-if="results">
        <h3 style="margin-bottom: 8px;">Found {{ results.pagination?.total_items || 0 }} Items</h3>
        <pre style="background: #f4f4f4; padding: 15px; border-radius: 8px; overflow-x: auto; white-space: pre-wrap; word-wrap: break-word;">{{ JSON.stringify(results, null, 2) }}</pre>
      </div>
      <p v-else>No results found.</p>
    </div>
  </div>
</template>

<style scoped>
button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}
</style>