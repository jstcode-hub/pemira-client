<script lang="ts" setup>
definePageMeta({
    layout: 'election',
    middleware: 'election',
});

const electionStore = useElectionStore();

electionStore.setProgress(2);

const video = ref<HTMLVideoElement>();
const videoStream = ref<MediaStream>();
const sources = ref<MediaDeviceInfo[]>([]);
const canvas = ref<HTMLCanvasElement>();
const picture = ref();
const portrait = ref(false);
const isMobile = ref(false);

const back = () => {
    navigateTo('/election/1');
};

const getDevices = async () => {
    if (!navigator.mediaDevices) {
        return;
    }

    const devices = await navigator.mediaDevices.enumerateDevices();
    sources.value = devices.filter((device) => device.kind === 'videoinput');

    if (sources.value.length === 0) {
        return;
    }

    if (electionStore.deviceId === undefined) {
        electionStore.deviceId = sources.value[0].deviceId;
    }
};

const stopCamera = () => {
    if (videoStream.value) {
        videoStream.value.getTracks().forEach((track) => {
            track.stop();
        });
    }
};

const afterError = ref(false);

const startCamera = (id?: string, forceUser = false) => {
    if (videoStream.value) {
        videoStream.value.getTracks().forEach((track) => track.stop());
    }

    navigator.mediaDevices
        .getUserMedia({
            video: {
                deviceId: id,
                aspectRatio: 16 / 9,
                facingMode:
                    isMobile.value && !forceUser ? 'environment' : undefined,
            },
            audio: false,
        })
        .then((stream) => {
            videoStream.value = stream;
            video.value!.srcObject = stream;
            video.value!.play();
        })
        .catch((err) => {
            console.error(err);
            if (afterError.value) {
                return;
            }
            afterError.value = true;
            startCamera(undefined, true);
        });
};

const capture = () => {
    if (!video.value || !video.value.srcObject) {
        return;
    }

    const context = canvas.value!.getContext('2d')!;

    canvas.value!.width = video.value.videoWidth;
    canvas.value!.height = video.value.videoHeight;

    context.save();
    context.scale(-1, 1);
    context.translate(-video.value.videoWidth, 0);

    context.drawImage(
        video.value,
        0,
        0,
        video.value.videoWidth,
        video.value.videoHeight,
    );

    const data = canvas.value!.toDataURL('image/png');
    picture.value = data;
};

const next = () => {
    electionStore.setKtmPicture(picture.value);
    navigateTo('/election/3');
};

const mobileCheck = () => {
    let check = false;
    (function (a) {
        if (
            /(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino/i.test(
                a,
            ) ||
            /1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(
                a.substr(0, 4),
            )
        )
            check = true;
    })(navigator.userAgent || navigator.vendor);
    return check;
};

onMounted(() => {
    isMobile.value = mobileCheck();

    if (!isMobile.value) {
        getDevices();
    } else {
        portrait.value = true;
    }

    startCamera(electionStore.deviceId);
});

onUnmounted(() => {
    stopCamera();
});

watch(
    () => electionStore.deviceId,
    async () => {
        if (!navigator.mediaDevices || video.value === undefined) {
            return;
        }

        startCamera(electionStore.deviceId);
    },
);
</script>

<template>
    <NuxtLayout>
        <UiCard class="grow flex flex-col bg-transparent rounded-none">
            <UiCardHeader class="flex items-center">
                <UiCardTitle>Foto KTM</UiCardTitle>
                <UiCardDescription>
                    Pastikan foto ktm terlihat jelas dan tidak blur
                </UiCardDescription>
            </UiCardHeader>
            <UiCardFooter class="sticky z-10 top-2 flex justify-center gap-4">
                <template v-if="picture">
                    <UiButton
                        size="lg"
                        :disabled="!videoStream || !picture"
                        variant="secondary"
                        @click="() => (picture = undefined)"
                    >
                        Ulangi
                    </UiButton>
                    <ConfirmationDialog
                        title="Apakah kamu yakin?"
                        description="Pastikan ktm terlihat jelas dan tidak blur karena akan digunakan untuk verifikasi pemilihanmu."
                        @confirm="next"
                    >
                        <UiButton
                            size="lg"
                            :disabled="!videoStream || !picture"
                        >
                            Selanjutnya
                        </UiButton>
                    </ConfirmationDialog>
                </template>
                <template v-else>
                    <UiButton size="lg" variant="secondary" @click="back">
                        Kembali
                    </UiButton>
                    <UiButton
                        :disabled="!videoStream"
                        size="lg"
                        class="sticky top-0"
                        @click="capture"
                    >
                        Ambil Foto
                    </UiButton>
                </template>
            </UiCardFooter>
            <UiCardContent v-if="!isMobile" class="flex items-center">
                <UiSelect v-model="electionStore.deviceId">
                    <UiSelectTrigger class="mx-auto max-w-sm">
                        <UiSelectValue
                            placeholder="Pilih Perangkat"
                            class="select-none"
                        />
                    </UiSelectTrigger>
                    <UiSelectContent>
                        <UiSelectGroup>
                            <UiSelectItem
                                v-for="source in sources"
                                :value="source.deviceId"
                            >
                                {{ source.label }}
                            </UiSelectItem>
                        </UiSelectGroup>
                    </UiSelectContent>
                </UiSelect>
            </UiCardContent>
            <UiCardContent>
                <div
                    class="relative lg:max-w-2xl xl:max-w-3xl mx-auto border-4 rounded-lg overflow-hidden"
                >
                    <UiAspectRatio
                        v-show="picture"
                        :ratio="portrait ? 9 / 16 : 16 / 9"
                        class="flex"
                    >
                        <canvas
                            ref="canvas"
                            class="w-full h-full bg-green-400"
                        ></canvas>
                    </UiAspectRatio>
                    <video
                        v-show="!picture"
                        ref="video"
                        autoplay="true"
                        muted="true"
                        playsinline="true"
                        class="w-full h-auto object-cover"
                    ></video>
                    <div
                        v-if="!picture"
                        class="absolute top-0 right-0 flex gap-2 items-center text-white text-xs font-medium p-4 animate-pulse"
                    >
                        <div>Recording</div>
                        <span
                            class="w-4 h-4 bg-red-400 border border-white rounded-full"
                        ></span>
                    </div>
                    <!-- <template v-if="portrait">
                        <div
                            class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 rotate-90 w-3/4 border-4 border-dashed rounded-lg opacity-50"
                        >
                            <UiAspectRatio :ratio="10 / 17" class="relative">
                                <div
                                    class="absolute h-[17%] w-[37%] bottom-[3%] left-1/2 transform -translate-x-1/2 border-4 border-dashed rounded-lg opacity-50"
                                ></div>
                            </UiAspectRatio>
                        </div>
                    </template> -->

                    <!-- v-else -->
                    <div
                        class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 w-3/4 border-4 border-dashed rounded-lg opacity-50"
                    >
                        <UiAspectRatio :ratio="17 / 10"></UiAspectRatio>
                    </div>
                </div>
            </UiCardContent>
        </UiCard>
    </NuxtLayout>
</template>
