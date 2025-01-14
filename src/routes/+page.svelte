<script>
  let elements = [];
  let isDragging = false;
  let isResizing = false;
  let currentElement = null;
  let startPos = { x: 0, y: 0 };
  let phoneContainer;
  let startSize = { width: 0, height: 0 };

  function handleImageUpload(event) {
    const file = event.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        elements = [...elements, {
          type: 'image',
          src: e.target.result,
          x: 50,
          y: 50,
          width: 96,
          height: 96,
          id: Date.now()
        }];
      };
      reader.readAsDataURL(file);
    }
  }

  function addText() {
    const text = prompt('Enter your text:');
    if (text) {
      elements = [...elements, {
        type: 'text',
        content: text,
        x: 50,
        y: 50,
        fontSize: 16,
        id: Date.now()
      }];
    }
  }

  function handleMouseDown(event, element, action = 'drag') {
    if (action === 'drag') {
      isDragging = true;
      const phoneRect = phoneContainer.getBoundingClientRect();
      startPos = {
        x: event.clientX - (element.x + phoneRect.left),
        y: event.clientY - (element.y + phoneRect.top)
      };
    } else if (action === 'resize') {
      isResizing = true;
      startPos = {
        x: event.clientX,
        y: event.clientY
      };
      startSize = {
        width: element.width || 96,
        height: element.height || 96,
        fontSize: element.fontSize || 16
      };
    }
    currentElement = element;
    event.preventDefault();
  }

  function handleMouseMove(event) {
    if (!currentElement) return;

    if (isDragging) {
      const phoneRect = phoneContainer.getBoundingClientRect();
      let newX = event.clientX - startPos.x - phoneRect.left;
      let newY = event.clientY - startPos.y - phoneRect.top;

      const maxX = phoneRect.width - (currentElement.width || 50);
      const maxY = phoneRect.height - (currentElement.height || 30);

      newX = Math.max(0, Math.min(newX, maxX));
      newY = Math.max(0, Math.min(newY, maxY));

      elements = elements.map(el => 
        el.id === currentElement.id 
          ? { ...el, x: newX, y: newY }
          : el
      );
    }

    if (isResizing) {
      const dx = event.clientX - startPos.x;
      const dy = event.clientY - startPos.y;

      if (currentElement.type === 'image') {
        const newWidth = Math.max(20, Math.min(280, startSize.width + dx));
        const newHeight = Math.max(20, Math.min(560, startSize.height + dy));

        elements = elements.map(el =>
          el.id === currentElement.id
            ? { ...el, width: newWidth, height: newHeight }
            : el
        );
      } else if (currentElement.type === 'text') {
        const newFontSize = Math.max(8, Math.min(72, startSize.fontSize + dx / 4));
        elements = elements.map(el =>
          el.id === currentElement.id
            ? { ...el, fontSize: newFontSize }
            : el
        );
      }
    }
  }

  function handleMouseUp() {
    isDragging = false;
    isResizing = false;
    currentElement = null;
  }
</script>

<div class="min-h-screen bg-gray-100 p-8">
  <!-- Title -->
  <h1 class="text-3xl font-bold text-center mb-8">Mobile Phone Designer</h1>
  
  <!-- Main Container with Border -->
  <div class="max-w-4xl mx-auto bg-white rounded-lg shadow-lg p-8 border-2 border-gray-200">
    <div class="flex items-center justify-center gap-8">
      <!-- Phone Container -->
      <div 
        bind:this={phoneContainer}
        class="relative w-[280px] h-[560px] bg-white rounded-[2rem] shadow-lg overflow-hidden border-8 border-gray-200"
        on:mousemove={handleMouseMove}
        on:mouseup={handleMouseUp}
        on:mouseleave={handleMouseUp}
      >
        <!-- Design Elements Container -->
        <div class="relative w-full h-full">
          {#each elements as element (element.id)}
            {#if element.type === 'image'}
              <div 
                class="absolute group"
                style="left: {element.x}px; top: {element.y}px;"
              >
                <img
                  src={element.src}
                  alt="Uploaded"
                  class="cursor-move touch-none"
                  style="width: {element.width}px; height: {element.height}px;"
                  on:mousedown={(e) => handleMouseDown(e, element, 'drag')}
                />
                <!-- Resize handle -->
                <div 
                  class="absolute bottom-0 right-0 w-4 h-4 bg-blue-500 opacity-0 group-hover:opacity-100 cursor-se-resize"
                  on:mousedown={(e) => handleMouseDown(e, element, 'resize')}
                ></div>
              </div>
            {:else if element.type === 'text'}
              <div 
                class="absolute group"
                style="left: {element.x}px; top: {element.y}px;"
              >
                <div
                  class="cursor-move px-2 py-1 bg-white/50 touch-none"
                  style="font-size: {element.fontSize}px;"
                  on:mousedown={(e) => handleMouseDown(e, element, 'drag')}
                >
                  {element.content}
                </div>
                <!-- Resize handle -->
                <div 
                  class="absolute bottom-0 right-0 w-4 h-4 bg-blue-500 opacity-0 group-hover:opacity-100 cursor-se-resize"
                  on:mousedown={(e) => handleMouseDown(e, element, 'resize')}
                ></div>
              </div>
            {/if}
          {/each}
        </div>
      </div>

      <!-- Tools Column -->
      <div class="flex flex-col gap-4">
        <label class="w-12 h-12 flex items-center justify-center bg-white border rounded cursor-pointer hover:bg-gray-50">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
          </svg>
          <input
            type="file"
            accept="image/*"
            class="hidden"
            on:change={handleImageUpload}
            aria-label="Upload image"
          />
        </label>

        <button
          class="w-12 h-12 flex items-center justify-center bg-white border rounded cursor-pointer hover:bg-gray-50"
          on:click={addText}
          aria-label="Add text"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 5h12M3 10h12M3 15h12" />
          </svg>
        </button>
      </div>
    </div>
  </div>
</div>