<script lang="ts" setup>
const route = useRoute();
const count = ref(0);
const loading = ref(false);

const getCount = async () => {
    loading.value = true;

    const { data, error } = await useApiFetch(
        `/events/${route.params.event}/ballots/count`,
    );

    loading.value = false;

    if (error.value) {
        return;
    }

    count.value = data.value ?? 0;
};

onMounted(() => {
    getCount();
});
</script>

<template>
    <UiCard>
        <UiCardHeader>
            <UiCardTitle class="flex justify-between text-white">
                Total Surat Suara
                <Icon
                    name="fluent:people-16-regular"
                    class="text-white md:w-fit w-10"
                />
            </UiCardTitle>
            <UiCardDescription class="text-2xl text-white">
                <Icon v-if="loading" name="svg-spinners:ring-resize" />
                <span v-else>{{ count }}</span>
            </UiCardDescription>
        </UiCardHeader>
    </UiCard>
</template>
