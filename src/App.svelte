<script>
  import imageCompression from 'browser-image-compression';
  import JSZip from 'jszip';

  let file, compressedFile, compressedURL, zipURL;
  let quality = "medium";
  let showToast = false;
  let toastMessage = "";

  const qualityOptions = {
    high: { maxSizeMB: 2, maxWidthOrHeight: 1200 },
    medium: { maxSizeMB: 1, maxWidthOrHeight: 800 },
    low: { maxSizeMB: 0.5, maxWidthOrHeight: 500 },
  };

  function showNotification(message) {
    toastMessage = message;
    showToast = true;
    setTimeout(() => (showToast = false), 3000);
  }

  async function handleFileUpload(event) {
    file = event.target.files[0];
    if (!file) return;

    const options = { ...qualityOptions[quality], useWebWorker: true };

    try {
      compressedFile = await imageCompression(file, options);
      compressedURL = URL.createObjectURL(compressedFile);
      showNotification("Gambar berhasil dikompres!");
    } catch (error) {
      console.error('Error compressing file:', error);
      showNotification("Gagal mengompres gambar.");
    }
  }

  async function createZip() {
    if (!compressedFile) return;

    const zip = new JSZip();
    zip.file(compressedFile.name, compressedFile);
    
    const blob = await zip.generateAsync({ type: 'blob' });
    zipURL = URL.createObjectURL(blob);
    showNotification("File ZIP berhasil dibuat!");
  }
</script>

<div class="min-h-screen w-full flex flex-col items-center justify-center bg-gradient-to-br from-gray-900 via-gray-800 to-gray-700 text-white px-4 py-12">
  <div class="w-full max-w-2xl bg-gray-800/80 backdrop-blur-lg p-8 rounded-xl shadow-xl border border-gray-700">
    <h2 class="text-3xl font-bold text-center mb-6 bg-gradient-to-r from-blue-400 to-purple-500 bg-clip-text text-transparent">Image Compressor</h2>
    <div class="space-y-6">
      <div>
        <label for="quality" class="block text-gray-300 font-medium mb-2">Pilih Kualitas:</label>
        <select id="quality" bind:value={quality} class="w-full p-3 bg-gray-700 border border-gray-600 rounded-lg focus:ring-2 focus:ring-blue-400 focus:outline-none">
          <option value="high">High</option>
          <option value="medium" selected>Medium</option>
          <option value="low">Low</option>
        </select>
      </div>
      <div>
        <input type="file" accept="image/*" on:change={handleFileUpload} class="w-full p-3 border border-gray-600 rounded-lg bg-gray-700 text-gray-300 cursor-pointer" />
      </div>
      {#if compressedURL}
        <div class="text-center">
          <p class="text-gray-300 font-medium mb-3">Gambar Terkompres:</p>
          <img src={compressedURL} alt="Compressed" class="mx-auto mt-3 rounded-md shadow-md border border-gray-600 max-h-64" />
          <div class="mt-6 space-y-3">
            <a href={compressedURL} download="compressed.jpg" class="block w-full bg-blue-500 text-white py-3 rounded-lg hover:bg-blue-600">Download Gambar</a>
            <button on:click={createZip} class="block w-full bg-green-500 text-white py-3 rounded-lg hover:bg-green-600">Buat ZIP</button>
          </div>
        </div>
      {/if}
      {#if zipURL}
        <a href={zipURL} download="compressed.zip" class="block w-full bg-purple-500 text-white py-3 rounded-lg hover:bg-purple-600 text-center">Download ZIP</a>
      {/if}
    </div>
  </div>
  {#if showToast}
    <div class="fixed top-4 right-4 bg-green-500 text-white px-6 py-3 rounded-lg shadow-lg animate-fade-in">{toastMessage}</div>
  {/if}
</div>

<style>
  @keyframes fade-in {
    from { opacity: 0; transform: translateY(-20px); }
    to { opacity: 1; transform: translateY(0); }
  }
  .animate-fade-in { animation: fade-in 0.3s ease-out; }
</style>
