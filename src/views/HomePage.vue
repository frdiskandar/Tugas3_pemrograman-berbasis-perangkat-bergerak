<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue';
import { IonContent, IonPage } from '@ionic/vue';

interface Currency {
  id: string;
  name: string;
  symbol: string;
  rank: string;
  price_usd: string;
}

interface CurrencyResponse {
  data: Currency[];
}

const API_URL = 'https://api.coinlore.net/api/tickers/';
const currencies = ref<Currency[]>([]);
const isLoading = ref(false);
const errorMessage = ref('');
let activeController: AbortController | null = null;

const formatPrice = (price: string) => {
  const numericPrice = Number(price);

  if (!Number.isFinite(numericPrice)) {
    return price;
  }

  return numericPrice.toLocaleString('en-US', {
    minimumFractionDigits: 0,
    maximumFractionDigits: 6,
  });
};

const loadCurrencies = async () => {
  activeController?.abort();
  const controller = new AbortController();
  activeController = controller;
  isLoading.value = true;
  errorMessage.value = '';

  try {
    const response = await fetch(API_URL, { signal: controller.signal });

    if (!response.ok) {
      throw new Error(`Request failed with status ${response.status}`);
    }

    const payload: unknown = await response.json();

    if (
      typeof payload !== 'object' ||
      payload === null ||
      !('data' in payload) ||
      !Array.isArray((payload as CurrencyResponse).data)
    ) {
      throw new Error('The API returned an invalid response.');
    }

    currencies.value = (payload as CurrencyResponse).data;
  } catch (error) {
    if (error instanceof DOMException && error.name === 'AbortError') {
      return;
    }

    errorMessage.value = 'Unable to load currency data. Please try again.';
  } finally {
    if (activeController === controller) {
      isLoading.value = false;
      activeController = null;
    }
  }
};

onMounted(loadCurrencies);
onUnmounted(() => activeController?.abort());


</script>


<template>
<ion-page>
    <ion-content>
      <main class="currency-page">
        <button class="refresh-button" type="button" :disabled="isLoading" @click="loadCurrencies">
          {{ isLoading ? 'Loading...' : 'Refresh' }}
        </button>

        <p v-if="errorMessage" class="status-message error-message" role="alert">
          {{ errorMessage }}
        </p>
        <p v-else-if="isLoading && currencies.length === 0" class="status-message">
          Loading currency data...
        </p>

        <section v-else-if="currencies.length > 0" class="currency-list" aria-label="Currency list">
          <article v-for="currency in currencies" :key="currency.id" class="currency-card">
            <div class="currency-column rank-column">
              <span class="column-label">Rank</span>
              <span class="rank-value">{{ currency.rank }}</span>
            </div>
            <div class="currency-column name-column">
              <span class="column-label">{{ currency.name }}</span>
              <span class="symbol-value">{{ currency.symbol }}</span>
            </div>
            <div class="currency-column price-column">
              <span class="column-label">USD</span>
              <span class="price-value">{{ formatPrice(currency.price_usd) }}</span>
            </div>
          </article>
        </section>

        <p v-else class="status-message">No currency data available.</p>
      </main>
    </ion-content>
  </ion-page>
</template>


<style scoped>
.currency-page {
  box-sizing: border-box;
  min-height: 100%;
  padding: 10px 8px 24px;
  background: #ffffff;
}

.refresh-button {
  display: block;
  min-width: 77px;
  margin: 0 auto 9px;
  padding: 9px 16px;
  border: 0;
  border-radius: 5px;
  background: #0864d8;
  color: #ffffff;
  cursor: pointer;
  font: inherit;
  font-size: 11px;
  font-weight: 500;
}

.refresh-button:disabled {
  cursor: wait;
  opacity: 0.7;
}

.currency-list {
  overflow: hidden;
  border: 1px solid #e3c66e;
  border-bottom: 0;
}

.currency-card {
  display: grid;
  grid-template-columns: 50px minmax(0, 1fr) minmax(90px, 1.15fr);
  min-height: 45px;
  border-bottom: 1px solid #e3c66e;
  background: #fff1c8;
  color: #111111;
}

.currency-column {
  display: flex;
  min-width: 0;
  flex-direction: column;
  justify-content: center;
  padding: 5px 8px;
}

.column-label {
  overflow: hidden;
  font-size: 9px;
  line-height: 13px;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.rank-column {
  align-items: center;
  padding-right: 2px;
  padding-left: 2px;
}

.rank-value,
.symbol-value,
.price-value {
  font-size: 17px;
  line-height: 20px;
}

.name-column {
  padding-right: 4px;
  padding-left: 9px;
}

.price-column {
  padding-right: 8px;
  padding-left: 4px;
}

.price-value {
  white-space: nowrap;
}

.status-message {
  margin: 36px 12px;
  color: #555555;
  font-size: 14px;
  text-align: center;
}

.error-message {
  color: #b42318;
}

@media (min-width: 480px) {
  .currency-page {
    max-width: 440px;
    margin: 0 auto;
  }
}

</style>