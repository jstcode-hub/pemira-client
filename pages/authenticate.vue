<script lang="ts" setup>
definePageMeta({
    layout: 'main',
});

onMounted(async () => {
    const route = useRoute();
    const accessToken = route.hash.split('&')[0].split('=')[1];

    if (!accessToken) navigateTo('/login');

    const { data, error, response } = await useApiFetch('/auth/login', {
        method: 'POST',
        body: JSON.stringify({ accessToken }),
    });

    if (error.value || !data.value) {
        navigateTo(`/login?error=${data.value.message || error.value}`);
        return;
    }

    useAuth().signIn(data.value);
    navigateTo('/');
});
</script>

<template>
    <NuxtLayout>
        <div class="grow flex items-center justify-center">
            <UiCard class="max-w-sm w-full">
                <UiCardHeader class="text-center">
                    <h2 class="text-3xl">PEMIRA</h2>
                    <UiCardDescription class="text-sm mt-0">
                        E-Vote HIMATIFA
                    </UiCardDescription>
                </UiCardHeader>
                <UiCardContent class="grid gap-4 justify-center">
                    <div class="flex justify-center">
                        <Icon name="svg-spinners:90-ring-with-bg" size="24" />
                    </div>
                    <UiCardDescription> Sedang memuat... </UiCardDescription>
                </UiCardContent>
            </UiCard>
        </div>
    </NuxtLayout>
</template>
