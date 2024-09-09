<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start" class="my-btn-wrap">
          <ion-button class="my-btn" @click="goBack">
            <ion-icon slot="start" :icon="chevronBack"></ion-icon>
          </ion-button>
        </ion-buttons>
        <ion-title class="text-center">Scanner</ion-title>
        <ion-buttons slot="primary" class="my-btn-wrap">
          <ion-button class="my-btn" @click="scanDocument">
            <ion-icon slot="start" :icon="cameraIcon"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <div>
        <ion-card
          v-for="(image, index) in images"
          :key="index"
          @click="openResultModal(image, index)"
          :class="{ selected: selImage && selImage.src === image.src }"
        >
          <ion-card-header>
            <ion-card-subtitle>{{ `Image ${index + 1}` }}</ion-card-subtitle>
          </ion-card-header>
          <ion-card-content>
            <ion-thumbnail slot="start">
              <img :src="image.src" :alt="`Image ${index + 1}`" />
            </ion-thumbnail>
          </ion-card-content>
        </ion-card>
      </div>

      <ion-modal
        v-if="showModal"
        @ionModalDidDismiss="showModal = false"
        :isOpen="showModal"
        canDismiss="true"
      >
        <ion-header>
          <ion-toolbar>
            <ion-title class="text-center">{{ modalTitle }}</ion-title>
            <ion-buttons slot="primary">
              <ion-button class="my-btn close" @click="closeResultModal">
                <ion-icon slot="start" :icon="closeIcon"></ion-icon>
              </ion-button>
            </ion-buttons>
          </ion-toolbar>
        </ion-header>
        <ion-content>
          <div ref="containerRef" class="result-container">
            <div v-if="canvases.length > 0" class="result-row">
              <div
                v-for="(canvas, index) in canvases"
                :key="index"
                class="result-item"
                @click="selectResCanvas(canvas)"
              >
                <ion-chip :color="canvas.chip">{{ canvas.caption }}</ion-chip>
                <canvas
                  :ref="(el: any) => appendCanvas(el, index)"
                  class="result-canvas"
                ></canvas>
              </div>
            </div>
          </div>

          <ion-card v-if="selResImageUri" class="mt-0">
            <ion-card-content>
              <img :src="selResImageUri" alt="Result Image" />
            </ion-card-content>
          </ion-card>
        </ion-content>
      </ion-modal>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { defineComponent, ref, watch, onMounted } from "vue";
import { useRouter } from "vue-router";
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonThumbnail,
  IonButtons,
  IonChip,
  IonModal,
  IonButton,
  IonIcon,
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardSubtitle,
  toastController
} from "@ionic/vue";
import {
  chevronBackOutline,
  cameraOutline,
  closeOutline
} from "ionicons/icons";

export default defineComponent({
  name: "DocumentScanner",
  components: {
    IonPage,
    IonHeader,
    IonToolbar,
    IonTitle,
    IonContent,
    IonThumbnail,
    IonButtons,
    IonChip,
    IonModal,
    IonButton,
    IonIcon,
    IonCard,
    IonCardContent,
    IonCardHeader,
    IonCardSubtitle
  },
  setup() {
    const router = useRouter();
    const containerRef = ref<HTMLDivElement | null>(null);
    const selImage = ref<{ src: string } | undefined>(undefined);
    const canvases = ref<
      {
        canvas: HTMLCanvasElement;
        caption: string;
        chip: string;
        height: number;
      }[]
    >([]);
    const showModal = ref(false);
    const selResImageUri = ref<string | undefined>(undefined);
    const modalTitle = ref<string>("");
    const chevronBack = chevronBackOutline;
    const cameraIcon = cameraOutline;
    const closeIcon = closeOutline;

    const images = [
      { src: "/paper-high.png" },
      { src: "/paper-2.png" },
      { src: "/paper-blue.png" }
    ];

    onMounted(() => {
      if (!window.cv) {
        loadOpencvLibs();
      }
    });

    const goBack = () => {
      try {
        router.back();
      } catch (e) {
        console.error("goBack:", e);
      }
    };

    const showToast = async (
      message: string,
      position: "top" | "middle" | "bottom"
    ) => {
      try {
        const toast = await toastController.create({
          message,
          duration: 1500,
          position
        });
        await toast.present();
      } catch (e) {
        console.error("showToast:", e);
      }
    };

    const loadOpencvLibs = () => {
      try {
        const scriptOpencv = document.createElement("script");
        /**
         * https://docs.opencv.org/4.7.0/opencv.js
         */
        scriptOpencv.src = "/assets/js/opencv-js/v4.7.0/opencv.js";
        scriptOpencv.async = true;
        scriptOpencv.onload = () => {
          console.log("opencv.js loaded");
          const scriptJscanify = document.createElement("script");
          /**
           * https://cdn.jsdelivr.net/gh/ColonelParrot/jscanify@master/src/jscanify.min.js
           */
          scriptJscanify.src = "/assets/js/jscanify/jscanify.js";
          scriptJscanify.async = true;
          scriptJscanify.onload = () => {
            console.log("jscanify loaded");
            showToast("opencv libraries loaded OK", "bottom");
          };
          document.head.appendChild(scriptJscanify);
        };
        document.head.appendChild(scriptOpencv);
      } catch (e: any) {
        console.error("loadOpencvLibs:", e);
        showToast(e.message, "bottom");
      }
    };

    const scanDocument = () => {
      try {
        //-- TODO
        console.log("scanDocument");
      } catch (e) {
        console.error("scanDocument:", e);
      }
    };

    const openResultModal = (image: { src: string }, index: number) => {
      try {
        selImage.value = image;
        modalTitle.value = `Image ${index}`;
        selResImageUri.value = undefined; //-- RESET
        showModal.value = true;
      } catch (e) {
        console.error("openResultModal:", e);
      }
    };

    const closeResultModal = () => {
      try {
        showModal.value = false;
      } catch (e) {
        console.error("closeResultModal:", e);
      }
    };

    const appendCanvas = (el: HTMLCanvasElement, index: number) => {
      try {
        if (el && canvases.value[index]) {
          const context = el.getContext("2d");
          if (context) {
            const canvasData = canvases.value[index].canvas;
            el.width = canvasData.width;
            el.height = canvasData.height;
            context.drawImage(canvasData, 0, 0, el.width, el.height);
          }
        }
      } catch (e) {
        console.error("appendCanvas:", e);
      }
    };

    const selectResCanvas = (canvas: { canvas: HTMLCanvasElement }) => {
      try {
        const dataUrl = canvas.canvas.toDataURL();
        selResImageUri.value = dataUrl;
      } catch (e) {
        console.error("selectResCanvas:", e);
      }
    };

    watch(selImage, (newVal) => {
      try {
        if (!window.cv) {
          let msg = "opencv.js is not initialized yet";
          showToast(msg, "bottom");
          loadOpencvLibs();
          throw new Error(msg);
        }

        const scanner = new (window as any).jscanify();

        if (newVal) {
          canvases.value = [];
          const newImg = document.createElement("img");
          newImg.src = newVal.src;

          newImg.onload = () => {
            try {
              const w = 386;
              const h = 500;
              const extractCanvas = scanner.extractPaper(newImg, w, h);
              const highlightCanvas = scanner.highlightPaper(newImg);
              const extractHeight =
                (w / extractCanvas.width) * extractCanvas.height;
              const highlightHeight =
                (w / highlightCanvas.width) * highlightCanvas.height;
              canvases.value = [
                {
                  canvas: extractCanvas,
                  caption: "Extracted",
                  chip: "success",
                  height: extractHeight
                },
                {
                  canvas: highlightCanvas,
                  caption: "Highlighted",
                  chip: "warning",
                  height: highlightHeight
                }
              ];
            } catch (err) {
              console.error(err);
            }
          };
        }
      } catch (e) {
        console.error("watch_selImage:", e);
      }
    });

    return {
      images,
      selImage,
      openResultModal,
      closeResultModal,
      canvases,
      containerRef,
      appendCanvas,
      showModal,
      selResImageUri,
      modalTitle,
      selectResCanvas,
      chevronBack,
      cameraIcon,
      closeIcon,
      goBack,
      scanDocument
    };
  }
});
</script>

<style scoped>
.result-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 16px;
}
.result-row {
  display: flex;
  justify-content: space-around;
  width: 100%;
  gap: 16px;
}
.result-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  cursor: pointer;
}
.result-canvas {
  width: 100%;
  object-fit: contain;
}
.selected {
  background-color: #ffe2de;
  border: 1px solid #ff6f5b;
  box-sizing: border-box;
}
ion-buttons.my-btn-wrap {
  margin-left: 8px;
  margin-right: 8px;
}
ion-button.my-btn {
  --background: #e7e5ff;
  --background-hover: #e7e5ff;
  --background-activated: #88f4be;
  --background-focused: #88f4be;
  --color: #7065e4;
  --border-radius: 8px;
}
ion-button.my-btn.close {
  --background: #ffe2de;
  --background-hover: #ffe2de;
  --background-activated: #747f9e;
  --background-focused: #747f9e;
  --color: #ff6f5b;
}
</style>
