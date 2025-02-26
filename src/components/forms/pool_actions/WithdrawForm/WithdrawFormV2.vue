<script setup lang="ts">
import { onBeforeMount, computed } from 'vue';
import TokenInput from '@/components/inputs/TokenInput/TokenInput.vue';
import { isLessThanOrEqualTo, isRequired } from '@/lib/utils/validations';
import useWeb3 from '@/services/web3/useWeb3';
import ProportionalWithdrawalInputV2 from './components/ProportionalWithdrawalInputV2.vue';
import WithdrawTotalsV2 from './components/WithdrawTotalsV2.vue';
import { useExitPool } from '@/providers/local/exit-pool.provider';
import useVeBal from '@/composables/useVeBAL';
import WithdrawPreviewModalV2 from './components/WithdrawPreviewModal/WithdrawPreviewModalV2.vue';
import { useTokens } from '@/providers/tokens.provider';
import {
  isPreMintedBptType,
  usePoolHelpers,
} from '@/composables/usePoolHelpers';
import { useI18n } from 'vue-i18n';
import { Pool } from '@/services/pool/types';

type Props = {
  pool: Pool;
};

/**
 * PROPS & EMITS
 */
const props = defineProps<Props>();

const pool = toRef(props, 'pool');

/**
 * STATE
 */
const showPreview = ref(false);

/**
 * COMPOSABLES
 */
const { t } = useI18n();
const { veBalTokenInfo } = useVeBal();
const { wrappedNativeAsset, nativeAsset } = useTokens();

const { isWalletReady, startConnectWithInjectedProvider, isMismatchedNetwork } =
  useWeb3();
const {
  isSingleAssetExit,
  singleAmountOut,
  isLoadingMax,
  queryError,
  maxError,
  isLoadingQuery,
  highPriceImpact,
  highPriceImpactAccepted,
  hasAcceptedHighPriceImpact,
  hasAmountsOut,
  validAmounts,
} = useExitPool();

const { isWrappedNativeAssetPool } = usePoolHelpers(pool);

/**
 * COMPUTED
 */
const singleAssetRules = computed(() => [
  isLessThanOrEqualTo(singleAmountOut.max, t('exceedsPoolBalance')),
]);

const hasValidInputs = computed(
  (): boolean => validAmounts.value && hasAcceptedHighPriceImpact.value
);

// Limit token select modal to a subset.
const subsetTokens = computed((): string[] => {
  if (isPreMintedBptType(pool.value.poolType)) return [];
  if (isWrappedNativeAssetPool.value)
    return [nativeAsset.address, ...pool.value.tokensList];

  return pool.value.tokensList;
});

const excludedTokens = computed((): string[] => {
  const tokens = [pool.value.address];
  if (veBalTokenInfo.value) {
    tokens.unshift(veBalTokenInfo.value.address);
  }
  return tokens;
});

/**
 * CALLBACKS
 */
onBeforeMount(() => {
  singleAmountOut.address = isPreMintedBptType(pool.value.poolType)
    ? wrappedNativeAsset.value.address
    : pool.value.tokensList[0];
});
</script>

<template>
  <div data-testid="withdraw-form">
    <ProportionalWithdrawalInputV2 v-if="!isSingleAssetExit" :pool="pool" />
    <template v-else>
      <!-- Single asset exit input -->
      <TokenInput
        v-model:isValid="singleAmountOut.valid"
        v-model:address="singleAmountOut.address"
        v-model:amount="singleAmountOut.value"
        :name="singleAmountOut.address"
        :rules="singleAssetRules"
        :customBalance="singleAmountOut.max || '0'"
        :balanceLabel="$t('max')"
        :balanceLoading="isLoadingMax"
        disableNativeAssetBuffer
        :excludedTokens="excludedTokens"
        :tokenSelectProps="{ ignoreBalances: true, subsetTokens }"
        ignoreWalletBalance
      />
    </template>

    <WithdrawTotalsV2 class="mt-4" />

    <div
      v-if="highPriceImpact"
      class="p-2 pb-2 mt-4 rounded-lg border dark:border-gray-700"
    >
      <BalCheckbox
        v-model="highPriceImpactAccepted"
        :rules="[isRequired($t('priceImpactCheckbox'))]"
        name="highPriceImpactAccepted"
        size="sm"
        :label="$t('priceImpactAccept', [$t('withdrawing')])"
      />
    </div>

    <BalAlert
      v-if="queryError || maxError"
      type="error"
      :title="$t('thereWasAnError')"
      :description="queryError || maxError"
      class="mt-4"
      block
    />

    <div class="mt-4">
      <BalBtn
        v-if="!isWalletReady"
        :label="$t('connectWallet')"
        color="gradient"
        block
        @click="startConnectWithInjectedProvider"
      />
      <BalBtn
        v-else
        :label="$t('preview')"
        color="gradient"
        :disabled="
          !hasAmountsOut ||
          !hasValidInputs ||
          isMismatchedNetwork ||
          isLoadingQuery ||
          isLoadingMax
        "
        block
        @click="showPreview = true"
      />
    </div>

    <teleport to="#modal">
      <WithdrawPreviewModalV2
        v-if="showPreview"
        :pool="pool"
        @close="showPreview = false"
      />
    </teleport>
  </div>
</template>
