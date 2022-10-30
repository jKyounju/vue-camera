<template>
    <div id="container">
        <div id="camera-container">
            <camera
                ref="camera"
                @loading="logEvent('loading')"
                @started="logEvent('started')"
                @error="(error) => logEvent('error: ' + error)"
            >
            </camera>
        </div>
        <div id="btn-group">
                <button
                    type="button"
                    class="btn camera btn-outline-dark"
                    @click="download"
                >
                    <span class="material-symbols-outlined photo">
                        photo_camera
                    </span>
                </button>
            </div>
    </div>

    <!-- <img :src="currentSnapshot" /> -->

    <!-- <select @change="changeCamera">
        <option
            v-for="camera in cameras"
            :key="camera"
            :value="camera.deviceId"
        >
            {{ camera.deviceId }}
        </option>
    </select> -->
</template>

<script lang="ts">
import { defineComponent, onMounted, Ref, ref } from "vue";

import Camera from "@/components/Camera.vue";

export default defineComponent({
    name: "App",
    components: {
        Camera,
    },
    setup() {
        const camera = ref<InstanceType<typeof Camera>>();

        const cameras: Ref<MediaDeviceInfo[]> = ref([]);

        onMounted(async () => {
            if (!camera.value) return;
            try {
                cameras.value = await camera.value.devices(["videoinput"]);
            } catch (e) {
                console.error(e);
            }
        });

        const start = () => camera.value?.start();
        const stop = () => camera.value?.stop();
        const pause = () => camera.value?.pause();
        const resume = () => camera.value?.resume();
        const snapshot = async () => {
            const blob = await camera.value?.snapshot({
                width: 224,
                height: 224,
            });
            currentSnapshot.value = URL.createObjectURL(blob!);
        };
        const download = async () => {
            const blob = await camera.value?.snapshot({
                width: 224,
                height: 224,
            });
            // currentSnapshot.value = URL.createObjectURL(blob!);
            let url = URL.createObjectURL(blob!);
            console.log(currentSnapshot.value);
            let image = new Image();
            image.setAttribute("crossOrigin", "anonymous");
            image.width = 224;
            image.height = 224;
            image.src = url;
            image.onload = () => {
                let canvas = document.createElement("canvas");
                canvas.width = image.width;
                canvas.height = image.height;
                let ctx =
                    canvas.getContext("2d") !== null
                        ? canvas.getContext("2d")
                        : new CanvasRenderingContext2D();
                if (ctx !== null) {
                    ctx.drawImage(image, 0, 0, image.width, image.height);
                    canvas.toBlob(() => {
                        let eleLink = document.createElement("a");
                        eleLink.download = "snapshot";
                        eleLink.href = url;
                        eleLink.click();
                        eleLink.remove();
                        URL.revokeObjectURL(url);
                    });
                }
            };
        };

        const logEvent = (name: string) => console.log(name);

        const currentSnapshot = ref();

        const changeCamera = (event: Event) => {
            const target = event.target as HTMLSelectElement;
            camera.value?.changeCamera(target.value);
        };

        return {
            camera,
            start,
            stop,
            pause,
            resume,
            logEvent,
            snapshot,
            download,
            currentSnapshot,
            cameras,
            changeCamera,
        };
    },
});
</script>

<style scoped>
#container {
    position: relative;
    width: 300px;
    height: 350px;
    border: 1px solid #000;
    overflow: hidden;
}

#camera-container {
    position: absolute;
    width: 300px;
    height: 300px;
}

#btn-group .camera {
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    margin: auto;
    width: 40px;
    height: 40px;
    border-radius: 100%;
    border: 3px solid #000;
}

#btn-group .camera .photo {
    position: absolute;
    bottom: 5px;
    right: 4.5px;
    font-size: 25px;
    font-weight: 1000;
}
</style>
