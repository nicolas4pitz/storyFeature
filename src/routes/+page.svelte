<script lang="ts">

let fileInput: HTMLInputElement;

let filesStorage: any[] = [];

import { onMount } from 'svelte';

onMount(() => {
  filesStorage = getStoredFiles();
  console.log('Arquivos carregados na inicialização:', filesStorage);
});

let createNewStory = () => {
  fileInput.click();
}

let getStoredFiles = () => {
  try {
    const storedFiles = localStorage.getItem('uploadedFiles');
    if (storedFiles) {
      return JSON.parse(storedFiles);
    }
    return [];
  } catch (error) {
    console.error('Erro ao recuperar arquivos do localStorage:', error);
    return [];
  }
}


let removeFile = (index: number) => {
  filesStorage.splice(index, 1);
  localStorage.setItem('uploadedFiles', JSON.stringify(filesStorage));
  console.log(`Arquivo ${index} removido`);
}



let handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement;
  const files = target.files;
  
  if (files && files.length > 0) {
    const file = files[0];
    console.log('Arquivo selecionado:', file.name);
    
    const reader = new FileReader();
    
    reader.onload = function(event) {
      const base64String = event.target?.result as string;
      
      const fileData = {
        id: Date.now(), // ID único baseado no timestamp
        name: file.name,
        type: file.type,
        size: file.size,
        lastModified: file.lastModified,
        base64: base64String,
        uploadedAt: new Date().toISOString()
      };
      
      try {
        // Adiciona o novo arquivo ao array
        filesStorage = [...filesStorage, fileData];
        localStorage.setItem('uploadedFiles', JSON.stringify(filesStorage));
        console.log('Arquivo salvo no localStorage!');
        
        const storageSize = new Blob([JSON.stringify(filesStorage)]).size;
        console.log(`Tamanho total no storage: ${(storageSize / 1024 / 1024).toFixed(2)} MB`);
        
      } catch (error) {
        console.error('Erro ao salvar no localStorage:', error);
        alert('Arquivo muito grande para o localStorage (limite ~5-10MB)');
      }
    };
    
    reader.onerror = function(error) {
      console.error('Erro ao ler arquivo:', error);
    };
    
    reader.readAsDataURL(file);
  }
}

</script>

<div id="carossel" class="w-60 h-14 m-2 rounded-md border flex items-center gap-2">
  <!-- Input file oculto -->
  <input 
    type="file" 
    bind:this={fileInput}
    on:change={handleFileSelect}
    accept="image/*,video/*"
    style="display: none;"
  />

  <button aria-label="add story" id="storysadd" class="center flex ml-2" on:click={createNewStory}>
    <svg
    xmlns="http://www.w3.org/2000/svg"
    width="30px"
    height="30px"
    viewBox="0 0 24 24"
    class="stroke-zinc-400 fill-none group-hover:fill-red-800 group-active:stroke-red-200 group-active:fill-red-600 group-active:duration-0 duration-300"
  >
    <path d="M8 12H16" stroke-width="1.5"></path>
    <path d="M12 16V8" stroke-width="1.5"></path>
  </svg>

  
</button>

  <!-- For loop para exibir todos os arquivos -->
  {#each filesStorage as file, index (file.id)}
    <div class="w-8 h-8 border border-zinc-900 rounded-full overflow-hidden bg-gray-200 relative cursor-pointer" 
         on:click={() => removeFile(index)}
         title="Remover - {file.name} story">
      {#if file.type.startsWith('image/')}
        <img 
          src={file.base64} 
          alt={file.name}
          class="w-full h-full object-cover brightness-75 hover:brightness-100 transition-all duration-300"
        />
      {:else if file.type.startsWith('video/')}
        <video class="w-full h-full object-cover brightness-75">
          <source src={file.base64} type={file.type} />
        </video>
        <div class="absolute inset-0 flex items-center justify-center">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="white" opacity="0.8">
            <path d="M8 5v14l11-7z"/>
          </svg>
        </div>
      {:else}
        <div class="w-full h-full flex items-center justify-center bg-gray-400">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="white">
            <path d="M14,2H6A2,2 0 0,0 4,4V20A2,2 0 0,0 6,22H18A2,2 0 0,0 20,20V8L14,2M18,20H6V4H13V9H18V20Z"/>
          </svg>
        </div>
      {/if}
    </div>
  {/each}
</div>




<style>

  *{
    box-sizing: border-box;
  }

  #storysadd{
    border: 1px solid black;
    width: 32px;
    border-radius: 50px;
    height: 32px;
  }

  .w-8 {
    width: 2rem;
  }
  
  .h-8 {
    height: 2rem;
  }
  
  .rounded-full {
    border-radius: 50%;
  }
  
  .overflow-hidden {
    overflow: hidden;
  }
  
  .bg-gray-200 {
    background-color: #e5e7eb;
  }
  
  .bg-gray-400 {
    background-color: #9ca3af;
  }
  
  .relative {
    position: relative;
  }
  
  .absolute {
    position: absolute;
  }
  
  .inset-0 {
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
  }
  
  .object-cover {
    object-fit: cover;
  }
  
  .brightness-75 {
    filter: brightness(0.75);
  }
  
  .brightness-75:hover {
    filter: brightness(1);
  }
  
  .transition-all {
    transition: all 0.3s ease;
  }
  
  .duration-300 {
    transition-duration: 300ms;
  }
  
  .flex {
    display: flex;
  }
  
  .items-center {
    align-items: center;
  }
  
  .justify-center {
    justify-content: center;
  }


</style>