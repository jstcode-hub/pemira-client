<script lang="ts" setup>
defineProps<{
    loading?: boolean;
}>();

const emit = defineEmits<{
    (e: 'signIn', email: string, password: string): void;
}>();

const form = reactive({
    email: '',
    password: '',
});

// Validasi format email
const isValidEmail = (email: string) => {
    const emailPattern = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    return emailPattern.test(email);
};

// Menangani klik atau penekanan tombol Enter untuk login
const handleSignIn = () => {
    if (!form.email || !form.password) {
        useAlertStore().show('Email dan password harus diisi', 'error');
        return;
    }
    if (!isValidEmail(form.email)) {
        useAlertStore().show('Email tidak valid', 'error');
        return;
    }
    emit('signIn', form.email, form.password);
};
</script>

<template>
    <UiCard
        class="flex min-h-full min-w-full flex-col justify-center px-6 py-12 lg:px-8 rounded-none bg-background"
    >
        <UiCardHeader class="text-center">
            <img
                class="w-32 h-32 aspect-square mx-auto"
                src="/logo.png"
                alt="Logo"
            />
            <h2 class="mt-10 text-center text-3xl">PEMIRA 2025</h2>
            <UiCardDescription> Admin Login Section</UiCardDescription>
        </UiCardHeader>
        <div class="mt-1 sm:mx-auto sm:w-full sm:max-w-sm">
            <UiCardContent class="space-y-6">
                <div class="grid gap-2">
                    <Alert />
                    <UiLabel>Email address</UiLabel>
                    <UiInput
                        v-model="form.email"
                        :disabled="$props.loading"
                        type="email"
                        placeholder="Email address"
                        class="bg-input"
                    />
                </div>
                <div class="grid gap-2">
                    <UiLabel>Password</UiLabel>
                    <UiInput
                        class="bg-input"
                        v-model="form.password"
                        :disabled="$props.loading"
                        type="password"
                        placeholder="Masukkan password"
                        @keydown.enter="handleSignIn"
                    />
                </div>
            </UiCardContent>
            <UiCardFooter>
                <UiButton
                    :loading="$props.loading"
                    class="w-full hover:bg-[#8e94a0] hover:text-white"
                    @click="handleSignIn"
                >
                    <Icon
                        v-if="!$props.loading"
                        name="material-symbols:login"
                        size="20"
                    ></Icon>
                    Masuk
                </UiButton>
            </UiCardFooter>
        </div>
    </UiCard>
</template>
