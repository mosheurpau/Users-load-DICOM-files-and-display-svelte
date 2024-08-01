<script lang="ts">
    import { onMount } from 'svelte';
  
    let cornerstone: any;
    let cornerstoneWADOImageLoader: any;
    let dicomParser: any;
    let dicomImageElement: HTMLDivElement | null = null;
  
    onMount(async () => {
      if (typeof window !== 'undefined') {
        const cornerstoneModule = await import('cornerstone-core');
        cornerstone = cornerstoneModule.default;
  
        const cornerstoneWADOImageLoaderModule = await import('cornerstone-wado-image-loader');
        cornerstoneWADOImageLoader = cornerstoneWADOImageLoaderModule.default;
  
        const dicomParserModule = await import('dicom-parser');
        dicomParser = dicomParserModule.default;
  
        cornerstoneWADOImageLoader.external.cornerstone = cornerstone;
        cornerstoneWADOImageLoader.external.dicomParser = dicomParser;
  
        if (dicomImageElement) {
          cornerstone.enable(dicomImageElement);
        }
      }
    });
  
    const handleFileChange = (event: Event) => {
      const target = event.target as HTMLInputElement;
      const file = target.files ? target.files[0] : null;
      if (file) {
        const imageId = cornerstoneWADOImageLoader.wadouri.fileManager.add(file);
        cornerstone.loadImage(imageId).then((image: any) => {
          if (dicomImageElement) {
            cornerstone.displayImage(dicomImageElement, image);
          }
        });
      }
    };
  
    const handleZoomIn = () => {
      if (dicomImageElement) {
        const viewport = cornerstone.getViewport(dicomImageElement);
        if (viewport) {
          viewport.scale += 0.1;
          cornerstone.setViewport(dicomImageElement, viewport);
        }
      }
    };
  
    const handleZoomOut = () => {
      if (dicomImageElement) {
        const viewport = cornerstone.getViewport(dicomImageElement);
        if (viewport) {
          viewport.scale -= 0.1;
          cornerstone.setViewport(dicomImageElement, viewport);
        }
      }
    };
  
    const handleRandomZoom = () => {
      if (dicomImageElement) {
        const viewport = cornerstone.getViewport(dicomImageElement);
        if (viewport) {
          viewport.scale = Math.random() * 2;
          cornerstone.setViewport(dicomImageElement, viewport);
        }
      }
    };
  
    const handleRotateDelta = () => {
      if (dicomImageElement) {
        const viewport = cornerstone.getViewport(dicomImageElement);
        if (viewport) {
          viewport.rotation += 30;
          cornerstone.setViewport(dicomImageElement, viewport);
        }
      }
    };
  
    const handleInvert = () => {
      if (dicomImageElement) {
        const viewport = cornerstone.getViewport(dicomImageElement);
        if (viewport) {
          viewport.invert = !viewport.invert;
          cornerstone.setViewport(dicomImageElement, viewport);
        }
      }
    };
  
    const handleResetViewport = () => {
      if (dicomImageElement) {
        const image = cornerstone.getImage(dicomImageElement);
        const defaultViewport = cornerstone.getDefaultViewportForImage(dicomImageElement, image);
        cornerstone.setViewport(dicomImageElement, defaultViewport);
      }
    };
  </script>
  
  <style>
    .dicom-viewer {
      width: 512px;
      height: 512px;
      background: black;
    }
  </style>
  
  <div class="text-center bg-slate-800">
    <input
      class="file-input file-input-bordered file-input-success w-full max-w-xs file-input-sm my-2"
      type="file"
      accept=".dcm"
      on:change={handleFileChange}
    />
    <div>
      <div
        class="dicom-viewer mx-auto"
        bind:this={dicomImageElement}
      ></div>
      <div class="mt-5 text-center">
        <button
          class="btn btn-outline btn-success btn-sm"
          on:click={handleZoomIn}
        >
          Zoom In
        </button>
        <button
          class="btn btn-outline btn-success btn-sm"
          on:click={handleZoomOut}
          style="margin-left: 10px;"
        >
          Zoom Out
        </button>
        <button
          class="btn btn-outline btn-success btn-sm"
          on:click={handleRandomZoom}
          style="margin-left: 10px;"
        >
          Random Zoom
        </button>
      </div>
  
      <div class="mt-2 text-center">
        <button
          class="btn btn-outline btn-success btn-sm mb-5"
          on:click={handleRotateDelta}
          style="margin-left: 10px;"
        >
          Rotate Delta 30
        </button>
        <button
          class="btn btn-outline btn-success btn-sm"
          on:click={handleInvert}
          style="margin-left: 10px;"
        >
          Invert
        </button>
        <button
          class="btn btn-outline btn-success btn-sm"
          on:click={handleResetViewport}
          style="margin-left: 10px;"
        >
          Reset Viewport
        </button>
      </div>
    </div>
  </div>
  