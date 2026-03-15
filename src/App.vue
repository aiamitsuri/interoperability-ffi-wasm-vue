<script setup>
import { ref, onMounted, watch } from 'vue';
import * as wasm from "@aiamitsuri/interoperability-ffi-wasm";

const results = ref(null);
const loading = ref(true);
const page = ref(1);

const loadData = async () => {
  loading.value = true;
  try {
    await wasm.default();
    const data = await wasm.fetch_from_js({
      page: page.value.toString(),
      ids: null
    });
    results.value = data;
  } catch (err) {
    console.error("Wasm Error:", err);
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
    <h1 style="margin-top: 0;">Interoperability FFI (Vue)</h1>

    <div v-if="loading && !results">🚀 Initializing Rust Wasm...</div>
    
    <div v-else>
      <div style="margin-bottom: 20px; display: flex; gap: 12px; align-items: center;">
        <button @click="handlePrev" :disabled="loading || !results?.pagination?.prev_page_url">
          Previous
        </button>

        <span style="font-weight: bold;">
          Page {{ results?.pagination?.current_page || page }} / {{ results?.pagination?.total_pages || 1 }}
        </span>

        <button @click="handleNext" :disabled="loading || !results?.pagination?.next_page_url">
          Next
        </button>
      </div>

      <div v-if="results?.data" style="display: flex; flex-direction: column; gap: 15px;">
        <div v-for="item in results.data" :key="item.id" 
             style="background: #f9f9f9; border: 1px solid #eee; padding: 15px; border-radius: 8px;">
          <h3 style="margin: 0 0 10px 0;">{{ item.title }}</h3>
          
          <div style="display: flex; gap: 15px; font-size: 0.9em; color: #666; margin-bottom: 8px; flex-wrap: wrap;">
            <span><strong>Language:</strong> {{ item.language }}</span>
            <span :style="{ color: item.integration === 'completed' ? 'green' : 'orange' }">
              <strong>Integration:</strong> {{ item.integration }}
            </span>
            
            <!-- Hide Opensources if pending or empty -->
            <span v-if="item.integration !== 'pending' && item.opensources?.length" 
                  style="color: #007bff; font-weight: bold;">
              📂 Opensources: {{ item.opensources.length }}
            </span>
          </div>

          <div style="font-size: 0.85em; display: flex; flex-direction: column; gap: 4px;">
            <p v-if="item.crates" style="margin: 0;">
              <strong>Crates:</strong> {{ item.crates.join(', ') }}
            </p>
            <p v-if="item.developmentkit" style="margin: 0;">
              <strong>Development Kit:</strong> {{ item.developmentkit.join(', ') }}
            </p>
          </div>
        </div>
      </div>
      <p v-else>No results found.</p>
    </div>
  </div>
</template>

<style scoped>
button {
  padding: 6px 12px;
  cursor: pointer;
}
button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}
</style>