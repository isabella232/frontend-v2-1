<script setup lang="ts">
import { Pool } from '@/services/pool/types';
import { riskLinks, risksTitle, poolSpecificRisks } from './pool-risks';

/**
 * TYPES
 */
type Props = {
  pool: Pool;
};

/**
 * PROPS
 */
defineProps<Props>();
</script>

<template>
  <div>
    <h3 class="px-4 lg:px-0 mb-5" v-text="$t('poolRisks.title')" />

    <p v-if="poolSpecificRisks(pool)" class="px-4 lg:px-0 mb-5">
      {{ poolSpecificRisks(pool) }}
    </p>

    <p class="px-4 lg:px-0 mb-3">
      {{ risksTitle(pool) }}
    </p>

    <ul class="px-8 lg:px-4 list-disc">
      <li v-for="{ title, hash } in riskLinks(pool)" :key="hash" class="link">
        <router-link :to="{ name: 'risks', hash }">{{ title }}</router-link>
      </li>
      <li class="link">
        <router-link :to="{ name: 'risks', hash: '#general-risks' }"
          >General balancer protocol risks</router-link
        >
      </li>
    </ul>
  </div>
</template>

