<script>
  let elements = [];
  let history = [];
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
        saveToHistory();
        elements = [
          ...elements,
          {
            type: "image",
            src: e.target.result,
            x: 50,
            y: 50,
            width: 96,
            height: 96,
            id: Date.now(),
          },
        ];
      };
      reader.readAsDataURL(file);
    }
  }

  function addText() {
    const text = prompt("Enter your text:");
    if (text) {
      saveToHistory();
      elements = [
        ...elements,
        {
          type: "text",
          content: text,
          x: 50,
          y: 50,
          fontSize: 16,
          id: Date.now(),
        },
      ];
    }
  }

  function handleMouseDown(event, element, action = "drag") {
    if (action === "drag") {
      isDragging = true;
      const phoneRect = phoneContainer.getBoundingClientRect();
      startPos = {
        x: event.clientX - (element.x + phoneRect.left),
        y: event.clientY - (element.y + phoneRect.top),
      };
    } else if (action === "resize") {
      isResizing = true;
      startPos = {
        x: event.clientX,
        y: event.clientY,
      };
      startSize = {
        width: element.width || 96,
        height: element.height || 96,
        fontSize: element.fontSize || 16,
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

      elements = elements.map((el) =>
        el.id === currentElement.id ? { ...el, x: newX, y: newY } : el
      );
    }

    if (isResizing) {
      const dx = event.clientX - startPos.x;
      const dy = event.clientY - startPos.y;

      if (currentElement.type === "image" || currentElement.type === "rectangle" || currentElement.type === "circle") {
        const newWidth = Math.max(20, Math.min(280, startSize.width + dx));
        const newHeight = Math.max(20, Math.min(560, startSize.height + dy));

        elements = elements.map((el) =>
          el.id === currentElement.id
            ? { ...el, width: newWidth, height: newHeight }
            : el
        );
      } else if (currentElement.type === "text") {
        const newFontSize = Math.max(8, Math.min(72, startSize.fontSize + dx / 4));
        elements = elements.map((el) =>
          el.id === currentElement.id ? { ...el, fontSize: newFontSize } : el
        );
      }
    }
  }

  function handleMouseUp() {
    isDragging = false;
    isResizing = false;
    currentElement = null;
  }

  function saveToHistory() {
    history.push(JSON.stringify(elements));
  }

  function undo() {
    if (history.length > 0) {
      const previousState = history.pop();
      elements = JSON.parse(previousState);
    }
  }

  function saveImage() {
    const phoneRect = phoneContainer.getBoundingClientRect();
    const canvas = document.createElement("canvas");
    canvas.width = phoneRect.width;
    canvas.height = phoneRect.height;
    const ctx = canvas.getContext("2d");
    ctx.fillStyle = "white";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    elements.forEach((element) => {
      if (element.type === "image") {
        const img = new Image();
        img.src = element.src;
        img.onload = () => {
          ctx.drawImage(img, element.x, element.y, element.width, element.height);
        };
      } else if (element.type === "text") {
        ctx.font = `${element.fontSize}px sans-serif`;
        ctx.fillStyle = "black";
        ctx.fillText(element.content, element.x, element.y + element.fontSize);
      } else if (element.type === "rectangle" || element.type === "circle" || element.type === "line") {
        ctx.lineWidth = element.borderWidth;
        ctx.strokeStyle = element.borderColor;
        ctx.fillStyle = element.fillColor;

        if (element.type === "rectangle") {
          ctx.fillRect(element.x, element.y, element.width, element.height);
          ctx.strokeRect(element.x, element.y, element.width, element.height);
        } else if (element.type === "circle") {
          ctx.beginPath();
          const radius = Math.min(element.width, element.height) / 2;
          ctx.arc(element.x + radius, element.y + radius, radius, 0, 2 * Math.PI);
          ctx.fill();
          ctx.stroke();
        } else if (element.type === "line") {
          ctx.beginPath();
          ctx.moveTo(element.x, element.y);
          ctx.lineTo(element.x + element.width, element.y);
          ctx.stroke();
        }
      }
    });

    setTimeout(() => {
      const link = document.createElement("a");
      link.download = "phone_design.png";
      link.href = canvas.toDataURL("image/png");
      link.click();
    }, 100);
  }

  function addShape(type) {
    const defaultShape = {
      type,
      x: 50,
      y: 50,
      width: 100,
      height: 100,
      borderColor: "#000",
      fillColor: "#FFF",
      borderWidth: 2,
      id: Date.now(),
    };

    if (type === "line") {
      defaultShape.width = 150;
      defaultShape.height = 2;
    }

    saveToHistory();
    elements = [...elements, defaultShape];
  }
</script>

<div class="min-h-screen bg-gray-100 p-8">
  <h1 class="text-3xl font-bold text-center mb-8">Mobile Phone Designer</h1>
  <div class="max-w-4xl mx-auto bg-white rounded-lg shadow-lg p-8 border-2 border-gray-200">
    <div class="flex items-center justify-center gap-8">
      <div bind:this={phoneContainer} class="relative w-[280px] h-[560px] bg-white rounded-[2rem] shadow-lg overflow-hidden border-8 border-gray-200" on:mousemove={handleMouseMove} on:mouseup={handleMouseUp} on:mouseleave={handleMouseUp}>
        <div class="relative w-full h-full">
          {#each elements as element (element.id)}
            {#if element.type === "image"}
              <div class="absolute group" style="left: {element.x}px; top: {element.y}px;">
                <img src={element.src} alt="Uploaded" class="cursor-move touch-none" style="width: {element.width}px; height: {element.height}px;" on:mousedown={(e) => handleMouseDown(e, element, "drag")} />
                <div class="absolute bottom-0 right-0 w-4 h-4 bg-blue-500 opacity-0 group-hover:opacity-100 cursor-se-resize" on:mousedown={(e) => handleMouseDown(e, element, "resize")}></div>
              </div>
            {:else if element.type === "text"}
              <div class="absolute group" style="left: {element.x}px; top: {element.y}px;">
                <div class="cursor-move px-2 py-1 bg-white/50 touch-none" style="font-size: {element.fontSize}px;" on:mousedown={(e) => handleMouseDown(e, element, "drag")}>
                  {element.content}
                </div>
                <div class="absolute bottom-0 right-0 w-4 h-4 bg-blue-500 opacity-0 group-hover:opacity-100 cursor-se-resize" on:mousedown={(e) => handleMouseDown(e, element, "resize")}></div>
              </div>
            {:else if element.type === "rectangle" || element.type === "circle" || element.type === "line"}
              <div class="absolute group" style="left: {element.x}px; top: {element.y}px;">
                <div class="cursor-move touch-none" style="width: {element.width}px; height: {element.height}px; border: {element.borderWidth}px solid {element.borderColor}; background-color: {element.fillColor}; border-radius: {element.type === 'circle' ? '50%' : '0'};" on:mousedown={(e) => handleMouseDown(e, element, "drag")}></div>
                <div class="absolute bottom-0 right-0 w-4 h-4 bg-blue-500 opacity-0 group-hover:opacity-100 cursor-se-resize" on:mousedown={(e) => handleMouseDown(e, element, "resize")}></div>
              </div>
            {/if}
          {/each}
        </div>
      </div>

      <div class="flex flex-col gap-4">
        <label class="w-12 h-12 flex items-center justify-center bg-white border rounded cursor-pointer hover:bg-gray-50">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
          </svg>
          <input type="file" accept="image/*" class="hidden" on:change={handleImageUpload} aria-label="Upload image" />
        </label>

        <button class="w-12 h-12 flex items-center justify-center bg-white border rounded cursor-pointer hover:bg-gray-50" on:click={addText} aria-label="Add text">
          Text
        </button>

        <button class="w-12 h-12 flex items-center justify-center bg-white border rounded cursor-pointer hover:bg-gray-50" on:click={() => addShape("rectangle")} aria-label="Add Rectangle">
          🟥
        </button>

        <button class="w-12 h-12 flex items-center justify-center bg-white border rounded cursor-pointer hover:bg-gray-50" on:click={() => addShape("circle")} aria-label="Add Circle">
          🟠
        </button>

        <button class="w-12 h-12 flex items-center justify-center bg-white border rounded cursor-pointer hover:bg-gray-50" on:click={() => addShape("line")} aria-label="Add Line">
          ➖
        </button>

        <button class="w-12 h-12 flex items-center justify-center bg-white border rounded cursor-pointer hover:bg-gray-50" on:click={undo} aria-label="Undo">
          Undo
        </button>

        <button class="w-12 h-12 flex items-center justify-center bg-white border rounded cursor-pointer hover:bg-gray-50" on:click={saveImage} aria-label="Save Image">
          💾
        </button>
      </div>
    </div>
  </div>
</div>