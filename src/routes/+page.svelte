<script>
  import { onMount } from 'svelte';
  let dicomFiles = [];
  let selectedFile = null;
  let containerRef;
  let bigImageRef;
  
  let cornerstone;
  let cornerstoneWADOImageLoader;
  let dicomParser;

  const handleFileChange = (event) => {
    dicomFiles = Array.from(event.target.files);
  };

  const handleImageClick = (file) => {
    selectedFile = file;
  };

  const handleZoomIn = () => {
    if (bigImageRef) {
      const viewport = cornerstone.getViewport(bigImageRef);
      viewport.scale += 0.1;
      cornerstone.setViewport(bigImageRef, viewport);
    }
  };

  const handleZoomOut = () => {
    if (bigImageRef) {
      const viewport = cornerstone.getViewport(bigImageRef);
      viewport.scale -= 0.1;
      cornerstone.setViewport(bigImageRef, viewport);
    }
  };

  const handleRandomZoom = () => {
    if (bigImageRef) {
      const viewport = cornerstone.getViewport(bigImageRef);
      viewport.scale = Math.random() * 2;
      cornerstone.setViewport(bigImageRef, viewport);
    }
  };

  const handleRotateDelta = () => {
    if (bigImageRef) {
      const viewport = cornerstone.getViewport(bigImageRef);
      viewport.rotation += 30;
      cornerstone.setViewport(bigImageRef, viewport);
    }
  };

  const handleInvert = () => {
    if (bigImageRef) {
      const viewport = cornerstone.getViewport(bigImageRef);
      viewport.invert = !viewport.invert;
      cornerstone.setViewport(bigImageRef, viewport);
    }
  };

  const handleResetViewport = () => {
    if (bigImageRef && selectedFile) {
      const imageId = cornerstoneWADOImageLoader.wadouri.fileManager.add(selectedFile);
      cornerstone.loadImage(imageId).then((image) => {
        const defaultViewport = cornerstone.getDefaultViewportForImage(bigImageRef, image);
        cornerstone.setViewport(bigImageRef, defaultViewport);
      });
    }
  };

  onMount(async () => {
    if (typeof window !== 'undefined') {
      cornerstone = (await import('cornerstone-core')).default;
      cornerstoneWADOImageLoader = (await import('cornerstone-wado-image-loader')).default;
      dicomParser = (await import('dicom-parser')).default;
      
      cornerstoneWADOImageLoader.external.cornerstone = cornerstone;
      cornerstoneWADOImageLoader.external.dicomParser = dicomParser;

      const container = containerRef;

      if (container && dicomFiles.length) {
        container.innerHTML = ''; // Clear previous content
        dicomFiles.forEach((file) => {
          const element = document.createElement('div');
          element.style.width = '200px';
          element.style.height = '200px';
          element.style.margin = '10px';
          element.style.background = 'black';
          element.style.cursor = 'pointer';
          container.appendChild(element);

          cornerstone.enable(element);
          const imageId = cornerstoneWADOImageLoader.wadouri.fileManager.add(file);
          cornerstone.loadImage(imageId).then((image) => {
            cornerstone.displayImage(element, image);
            element.addEventListener('click', () => handleImageClick(file));
          });
        });
      }

      return () => {
        if (container) {
          container.innerHTML = '';
        }
      };
    }
  });
</script>

<style>
  .container {
    height: 600px;
    width: 200px;
    display: flex;
    flex-direction: column;
    overflow-y: auto;
    overflow-x: hidden;
    background-color: #00A96E;
  }
  .big-image {
    width: 100%;
    height: 100%;
    background: black;
  }
</style>

<div class="text-center">
  <h1 class="text-4xl font-bold mb-2 text-center">DICOM Viewer</h1>
  <input
    class="file-input file-input-bordered file-input-success w-full max-w-xs file-input-sm my-2"
    type="file"
    accept=".dcm"
    multiple
    on:change={handleFileChange}
  />
  <div class="grid grid-cols-5 gap-1 px-5">
    <div class="container" bind:this={containerRef}></div>

    <div class="bg-slate-800 col-span-4 flex justify-center items-center" style="height: 600px; width: 1100px;">
      <div class="my-auto big-image" bind:this={bigImageRef}></div>
    </div>
  </div>
  {#if selectedFile}
    <div class="text-center mt-5">
      <button class="btn btn-outline btn-success btn-sm" on:click={handleZoomIn}>Zoom In</button>
      <button class="btn btn-outline btn-success btn-sm" on:click={handleZoomOut} style="margin-left: 10px;">Zoom Out</button>
      <button class="btn btn-outline btn-success btn-sm" on:click={handleRandomZoom} style="margin-left: 10px;">Random Zoom</button>
      <button class="btn btn-outline btn-success btn-sm" on:click={handleRotateDelta} style="margin-left: 10px;">Rotate Delta 30</button>
      <button class="btn btn-outline btn-success btn-sm" on:click={handleInvert} style="margin-left: 10px;">Invert</button>
      <button class="btn btn-outline btn-success btn-sm" on:click={handleResetViewport} style="margin-left: 10px;">Reset Viewport</button>
    </div>
  {/if}
</div>
