<script lang="ts" setup>
import { boostedProtocolIconPaths } from '@/composables/useBoostedPool';
import { PoolMetadata } from '@/types/pools';

type Props = {
  metadata: PoolMetadata;
};

const props = defineProps<Props>();

const boostedProtocols = props.metadata?.boostedProtocols || [];

const iconURIs = boostedProtocols.map(
  protocol => boostedProtocolIconPaths[protocol]
);

const hasIcons = boostedProtocols.length > 0;

const width = 25 + (iconURIs.length - 1) * 16;
</script>

<template>
  <div
    data-testid="boosted-chip"
    class="flex relative items-center py-1 px-2 max-h-10 bg-gradient-to-tr from-yellow-500 to-pink-500 rounded-lg"
  >
    <BalAssetSet
      v-if="hasIcons"
      :logoURIs="iconURIs"
      :width="width"
      :size="20"
      :ringSize="1"
    />
    <span class="text-sm font-bold text-white">{{ $t('boosted') }}</span>
  </div>
</template>
