<script>
  import { invoke } from "@tauri-apps/api/core";

  /**
   * @typedef {Object} Modality
   * @property {string} id
   * @property {string} name
   * @property {string} icon
   * @property {string} description
   * @property {'file' | 'url' | 'textarea'} inputType
   * @property {string} [accept]
   * @property {boolean} [multiple]
   */

  /**
   * @typedef {Object} UploadedFile
   * @property {string} name
   * @property {string} size
   * @property {string} type
   * @property {File} file
   */

  /**
   * @typedef {Object} SearchResult
   * @property {number} id
   * @property {string} snippet
   * @property {string} location
   * @property {number} relevance
   */

  // Content modalities
  /** @type {Modality[]} */
  const modalities = [
    { id: 'pdf', name: 'PDF Document', icon: '📄', description: 'Search within PDF files', inputType: 'file', accept: '.pdf' },
    { id: 'text', name: 'Plain Text', icon: '📝', description: 'Search in text content', inputType: 'textarea' },
    { id: 'website', name: 'Website', icon: '🌐', description: 'Search webpage content', inputType: 'url' },
    { id: 'youtube', name: 'YouTube Video', icon: '📺', description: 'Search video transcripts', inputType: 'url' },
    { id: 'image', name: 'Image Gallery', icon: '🖼️', description: 'Search image descriptions', inputType: 'file', accept: 'image/*', multiple: true },
    { id: 'audio', name: 'Audio File', icon: '🎵', description: 'Search audio transcripts', inputType: 'file', accept: 'audio/*' },
  ];

  /** @type {Modality | null} */
  let selectedModality = $state(null);
  let contentInput = $state('');
  let searchQuery = $state('');
  /** @type {UploadedFile[]} */
  let uploadedFiles = $state([]);
  let isSearching = $state(false);
  /** @type {SearchResult[]} */
  let searchResults = $state([]);
  let dragOver = $state(false);

  /**
   * @param {Modality} modality
   */
  function selectModality(modality) {
    selectedModality = modality;
    contentInput = '';
    uploadedFiles = [];
    searchResults = [];
  }

  /**
   * @param {Event} event
   */
  function handleFileSelect(event) {
    const target = /** @type {HTMLInputElement} */ (event.target);
    const files = Array.from(target.files || []);
    uploadedFiles = files.map(f => ({
      name: f.name,
      size: formatFileSize(f.size),
      type: f.type,
      file: f
    }));
  }

  /**
   * @param {DragEvent} event
   */
  function handleDrop(event) {
    event.preventDefault();
    dragOver = false;
    const files = Array.from(event.dataTransfer?.files || []);
    if (selectedModality?.inputType === 'file') {
      uploadedFiles = files.map(f => ({
        name: f.name,
        size: formatFileSize(f.size),
        type: f.type,
        file: f
      }));
    }
  }

  /**
   * @param {DragEvent} event
   */
  function handleDragOver(event) {
    event.preventDefault();
    dragOver = true;
  }

  function handleDragLeave() {
    dragOver = false;
  }

  /**
   * @param {number} bytes
   */
  function formatFileSize(bytes) {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }

  /**
   * @param {number} index
   */
  function removeFile(index) {
    uploadedFiles = uploadedFiles.filter((_, i) => i !== index);
  }

  async function performSearch() {
    if (!searchQuery.trim()) return;
    
    isSearching = true;
    
    // Simulate search delay - replace with actual backend call
    await new Promise(resolve => setTimeout(resolve, 1500));
    
    // Mock results - in production, this would call your backend
    searchResults = [
      { 
        id: 1, 
        snippet: `Found relevant content matching "${searchQuery}"...`, 
        location: 'Page 3, Paragraph 2',
        relevance: 95
      },
      { 
        id: 2, 
        snippet: `Another match for "${searchQuery}" in the document...`, 
        location: 'Page 7, Section 4',
        relevance: 82
      },
      { 
        id: 3, 
        snippet: `Related content discussing ${searchQuery}...`, 
        location: 'Page 12, Introduction',
        relevance: 71
      },
    ];
    
    isSearching = false;
  }

  function goBack() {
    selectedModality = null;
    contentInput = '';
    uploadedFiles = [];
    searchResults = [];
    searchQuery = '';
  }

  function hasContent() {
    if (selectedModality?.inputType === 'file') {
      return uploadedFiles.length > 0;
    }
    return contentInput.trim().length > 0;
  }
</script>

<div class="app">
  <div class="background-pattern"></div>
  
  <header class="header">
    <div class="logo">
      <span class="logo-icon">◈</span>
      <span class="logo-text">ContextSearch</span>
    </div>
    {#if selectedModality}
      <button class="back-btn" onclick={goBack}>
        <span class="back-arrow">←</span>
        <span>Back to Modalities</span>
      </button>
    {/if}
  </header>

  <main class="main">
    {#if !selectedModality}
      <!-- Modality Selection Screen -->
      <section class="hero" style="animation-delay: 0ms">
        <h1 class="title">
          <span class="title-gradient">Context</span>Search
        </h1>
        <p class="subtitle">Search through any content type with precision</p>
      </section>

      <section class="modality-grid" style="animation-delay: 100ms">
        {#each modalities as modality, i}
          <button 
            class="modality-card" 
            onclick={() => selectModality(modality)}
            style="animation-delay: {150 + i * 50}ms"
          >
            <span class="modality-icon">{modality.icon}</span>
            <h3 class="modality-name">{modality.name}</h3>
            <p class="modality-desc">{modality.description}</p>
            <div class="modality-glow"></div>
          </button>
        {/each}
      </section>

    {:else}
      <!-- Content Input & Search Screen -->
      <section class="search-panel" style="animation-delay: 0ms">
        <div class="panel-header">
          <span class="panel-icon">{selectedModality.icon}</span>
          <div class="panel-info">
            <h2 class="panel-title">{selectedModality.name}</h2>
            <p class="panel-desc">{selectedModality.description}</p>
          </div>
        </div>

        <!-- Content Input Area -->
        <div class="content-input-section">
          <h3 class="section-label">1. Add Your Content</h3>
          
          {#if selectedModality.inputType === 'file'}
            <div 
              class="file-drop-zone" 
              class:drag-over={dragOver}
              class:has-files={uploadedFiles.length > 0}
              ondrop={handleDrop}
              ondragover={handleDragOver}
              ondragleave={handleDragLeave}
              role="button"
              tabindex="0"
            >
              {#if uploadedFiles.length === 0}
                <div class="drop-content">
                  <span class="drop-icon">⬆</span>
                  <p class="drop-text">Drag & drop files here</p>
                  <p class="drop-subtext">or</p>
                  <label class="file-input-label">
                    <span>Browse Files</span>
                    <input 
                      type="file" 
                      accept={selectedModality.accept}
                      multiple={selectedModality.multiple}
                      onchange={handleFileSelect}
                    />
                  </label>
                </div>
              {:else}
                <div class="uploaded-files">
                  {#each uploadedFiles as file, index}
                    <div class="file-item">
                      <span class="file-icon">📎</span>
                      <div class="file-info">
                        <span class="file-name">{file.name}</span>
                        <span class="file-size">{file.size}</span>
                      </div>
                      <button class="remove-file" onclick={() => removeFile(index)}>✕</button>
                    </div>
                  {/each}
                  <label class="add-more-files">
                    <span>+ Add more files</span>
                    <input 
                      type="file" 
                      accept={selectedModality.accept}
                      multiple={selectedModality.multiple}
                      onchange={handleFileSelect}
                    />
                  </label>
                </div>
              {/if}
            </div>

          {:else if selectedModality.inputType === 'url'}
            <div class="url-input-wrapper">
              <span class="url-icon">🔗</span>
              <input 
                type="url" 
                class="url-input"
                placeholder={selectedModality.id === 'youtube' ? 'Paste YouTube video URL...' : 'Paste website URL...'}
                bind:value={contentInput}
              />
              {#if contentInput}
                <button class="clear-input" onclick={() => contentInput = ''}>✕</button>
              {/if}
            </div>
            {#if selectedModality.id === 'youtube'}
              <p class="input-hint">Supports: youtube.com, youtu.be links</p>
            {:else}
              <p class="input-hint">Enter any public webpage URL</p>
            {/if}

          {:else if selectedModality.inputType === 'textarea'}
            <div class="textarea-wrapper">
              <textarea 
                class="text-input"
                placeholder="Paste or type your text content here..."
                bind:value={contentInput}
                rows="8"
              ></textarea>
              <div class="textarea-footer">
                <span class="char-count">{contentInput.length} characters</span>
              </div>
            </div>
          {/if}
        </div>

        <!-- Search Query Area -->
        <div class="search-input-section">
          <h3 class="section-label">2. What are you looking for?</h3>
          <div class="search-input-wrapper">
            <span class="search-icon">🔍</span>
            <input 
              type="text" 
              class="search-input"
              placeholder="Describe what you want to find..."
              bind:value={searchQuery}
              onkeydown={(e) => e.key === 'Enter' && hasContent() && performSearch()}
            />
            <button 
              class="search-btn" 
              onclick={performSearch}
              disabled={!hasContent() || !searchQuery.trim() || isSearching}
            >
              {#if isSearching}
                <span class="spinner"></span>
                <span>Searching...</span>
              {:else}
                <span>Search</span>
                <span class="search-arrow">→</span>
              {/if}
            </button>
          </div>
          {#if !hasContent()}
            <p class="input-warning">Please add content above before searching</p>
          {/if}
        </div>

        <!-- Search Results -->
        {#if searchResults.length > 0}
          <div class="results-section">
            <h3 class="section-label">Results</h3>
            <div class="results-list">
              {#each searchResults as result, i}
                <div class="result-card" style="animation-delay: {i * 100}ms">
                  <div class="result-header">
                    <span class="result-relevance" style="--relevance: {result.relevance}%">
                      {result.relevance}% match
                    </span>
                    <span class="result-location">{result.location}</span>
                  </div>
                  <p class="result-snippet">{result.snippet}</p>
                  <button class="result-action">View in context →</button>
                </div>
              {/each}
            </div>
          </div>
        {/if}
      </section>
    {/if}
  </main>

  <footer class="footer">
    <p>Built with precision for your search needs</p>
  </footer>
</div>

<style>
  /* CSS Variables */
  :global(:root) {
    --bg-primary: #0a0a0f;
    --bg-secondary: #12121a;
    --bg-tertiary: #1a1a25;
    --bg-card: #15151f;
    --text-primary: #e8e8f0;
    --text-secondary: #9090a8;
    --text-muted: #606078;
    --accent-primary: #00d4aa;
    --accent-secondary: #00a8ff;
    --accent-gradient: linear-gradient(135deg, #00d4aa 0%, #00a8ff 100%);
    --border-color: #2a2a3a;
    --border-glow: rgba(0, 212, 170, 0.3);
    --shadow-color: rgba(0, 0, 0, 0.4);
    --font-display: 'Outfit', sans-serif;
    --font-mono: 'JetBrains Mono', monospace;
  }

  :global(*) {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  :global(body) {
    font-family: var(--font-display);
    background: var(--bg-primary);
    color: var(--text-primary);
    min-height: 100vh;
    overflow-x: hidden;
  }

  .app {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    position: relative;
  }

  .background-pattern {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-image: 
      radial-gradient(circle at 20% 20%, rgba(0, 212, 170, 0.08) 0%, transparent 50%),
      radial-gradient(circle at 80% 80%, rgba(0, 168, 255, 0.08) 0%, transparent 50%),
      linear-gradient(rgba(255,255,255,0.02) 1px, transparent 1px),
      linear-gradient(90deg, rgba(255,255,255,0.02) 1px, transparent 1px);
    background-size: 100% 100%, 100% 100%, 50px 50px, 50px 50px;
    pointer-events: none;
    z-index: 0;
  }

  /* Header */
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1.5rem 3rem;
    position: relative;
    z-index: 10;
    border-bottom: 1px solid var(--border-color);
    backdrop-filter: blur(10px);
    background: rgba(10, 10, 15, 0.8);
  }

  .logo {
    display: flex;
    align-items: center;
    gap: 0.75rem;
  }

  .logo-icon {
    font-size: 1.75rem;
    background: var(--accent-gradient);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    animation: pulse 3s infinite;
  }

  @keyframes pulse {
    0%, 100% { opacity: 1; transform: scale(1); }
    50% { opacity: 0.8; transform: scale(1.05); }
  }

  .logo-text {
    font-size: 1.5rem;
    font-weight: 600;
    letter-spacing: -0.5px;
  }

  .back-btn {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.6rem 1.2rem;
    background: var(--bg-tertiary);
    border: 1px solid var(--border-color);
    border-radius: 8px;
    color: var(--text-secondary);
    font-family: var(--font-display);
    font-size: 0.9rem;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .back-btn:hover {
    background: var(--bg-card);
    border-color: var(--accent-primary);
    color: var(--text-primary);
  }

  .back-arrow {
    font-size: 1.1rem;
  }

  /* Main Content */
  .main {
    flex: 1;
    padding: 3rem;
    position: relative;
    z-index: 1;
    max-width: 1400px;
    margin: 0 auto;
    width: 100%;
  }

  /* Hero Section */
  .hero {
    text-align: center;
    margin-bottom: 4rem;
    animation: fadeInUp 0.6s ease forwards;
    opacity: 0;
  }

  @keyframes fadeInUp {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .title {
    font-size: 4rem;
    font-weight: 700;
    letter-spacing: -2px;
    margin-bottom: 1rem;
  }

  .title-gradient {
    background: var(--accent-gradient);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }

  .subtitle {
    font-size: 1.25rem;
    color: var(--text-secondary);
    font-weight: 300;
  }

  /* Modality Grid */
  .modality-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 1.5rem;
    animation: fadeInUp 0.6s ease forwards;
    opacity: 0;
  }

  .modality-card {
    position: relative;
    background: var(--bg-card);
    border: 1px solid var(--border-color);
    border-radius: 16px;
    padding: 2rem;
    text-align: left;
    cursor: pointer;
    transition: all 0.3s ease;
    overflow: hidden;
    font-family: var(--font-display);
    animation: fadeInUp 0.6s ease forwards;
    opacity: 0;
  }

  .modality-card:hover {
    transform: translateY(-4px);
    border-color: var(--accent-primary);
    box-shadow: 0 20px 40px var(--shadow-color), 0 0 30px var(--border-glow);
  }

  .modality-card:hover .modality-glow {
    opacity: 1;
  }

  .modality-glow {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: radial-gradient(circle at 50% 0%, rgba(0, 212, 170, 0.1) 0%, transparent 70%);
    opacity: 0;
    transition: opacity 0.3s ease;
    pointer-events: none;
  }

  .modality-icon {
    font-size: 3rem;
    display: block;
    margin-bottom: 1rem;
    filter: grayscale(0.2);
    transition: filter 0.3s ease;
  }

  .modality-card:hover .modality-icon {
    filter: grayscale(0);
  }

  .modality-name {
    font-size: 1.25rem;
    font-weight: 600;
    margin-bottom: 0.5rem;
    color: var(--text-primary);
  }

  .modality-desc {
    font-size: 0.9rem;
    color: var(--text-secondary);
    line-height: 1.5;
  }

  /* Search Panel */
  .search-panel {
    background: var(--bg-secondary);
    border: 1px solid var(--border-color);
    border-radius: 20px;
    padding: 2.5rem;
    animation: fadeInUp 0.5s ease forwards;
    opacity: 0;
  }

  .panel-header {
    display: flex;
    align-items: center;
    gap: 1.25rem;
    margin-bottom: 2.5rem;
    padding-bottom: 2rem;
    border-bottom: 1px solid var(--border-color);
  }

  .panel-icon {
    font-size: 3rem;
    background: var(--bg-tertiary);
    padding: 1rem;
    border-radius: 16px;
    border: 1px solid var(--border-color);
  }

  .panel-title {
    font-size: 1.75rem;
    font-weight: 600;
    margin-bottom: 0.25rem;
  }

  .panel-desc {
    color: var(--text-secondary);
    font-size: 0.95rem;
  }

  /* Section Labels */
  .section-label {
    font-size: 0.85rem;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 1px;
    color: var(--accent-primary);
    margin-bottom: 1rem;
    font-family: var(--font-mono);
  }

  /* Content Input Section */
  .content-input-section {
    margin-bottom: 2.5rem;
  }

  /* File Drop Zone */
  .file-drop-zone {
    border: 2px dashed var(--border-color);
    border-radius: 16px;
    padding: 3rem;
    text-align: center;
    transition: all 0.3s ease;
    background: var(--bg-tertiary);
  }

  .file-drop-zone.drag-over {
    border-color: var(--accent-primary);
    background: rgba(0, 212, 170, 0.05);
  }

  .file-drop-zone.has-files {
    padding: 1.5rem;
  }

  .drop-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.75rem;
  }

  .drop-icon {
    font-size: 3rem;
    opacity: 0.6;
  }

  .drop-text {
    font-size: 1.1rem;
    color: var(--text-primary);
  }

  .drop-subtext {
    color: var(--text-muted);
    font-size: 0.9rem;
  }

  .file-input-label {
    display: inline-block;
    padding: 0.75rem 1.5rem;
    background: var(--accent-gradient);
    border-radius: 8px;
    color: var(--bg-primary);
    font-weight: 600;
    cursor: pointer;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }

  .file-input-label:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba(0, 212, 170, 0.3);
  }

  .file-input-label input {
    display: none;
  }

  .uploaded-files {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }

  .file-item {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1rem;
    background: var(--bg-card);
    border: 1px solid var(--border-color);
    border-radius: 10px;
  }

  .file-icon {
    font-size: 1.5rem;
  }

  .file-info {
    flex: 1;
    text-align: left;
  }

  .file-name {
    display: block;
    font-weight: 500;
    color: var(--text-primary);
    margin-bottom: 0.25rem;
  }

  .file-size {
    font-size: 0.8rem;
    color: var(--text-muted);
    font-family: var(--font-mono);
  }

  .remove-file {
    width: 28px;
    height: 28px;
    border-radius: 50%;
    border: none;
    background: var(--bg-tertiary);
    color: var(--text-secondary);
    cursor: pointer;
    transition: all 0.2s ease;
    font-size: 0.9rem;
  }

  .remove-file:hover {
    background: #ff4757;
    color: white;
  }

  .add-more-files {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0.75rem;
    border: 1px dashed var(--border-color);
    border-radius: 8px;
    color: var(--text-secondary);
    cursor: pointer;
    transition: all 0.2s ease;
    font-size: 0.9rem;
  }

  .add-more-files:hover {
    border-color: var(--accent-primary);
    color: var(--accent-primary);
  }

  .add-more-files input {
    display: none;
  }

  /* URL Input */
  .url-input-wrapper {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1rem 1.5rem;
    background: var(--bg-tertiary);
    border: 2px solid var(--border-color);
    border-radius: 12px;
    transition: border-color 0.2s ease;
  }

  .url-input-wrapper:focus-within {
    border-color: var(--accent-primary);
  }

  .url-icon {
    font-size: 1.25rem;
    opacity: 0.6;
  }

  .url-input {
    flex: 1;
    background: transparent;
    border: none;
    outline: none;
    font-size: 1rem;
    color: var(--text-primary);
    font-family: var(--font-mono);
  }

  .url-input::placeholder {
    color: var(--text-muted);
  }

  .clear-input {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    border: none;
    background: var(--bg-card);
    color: var(--text-muted);
    cursor: pointer;
    transition: all 0.2s ease;
    font-size: 0.8rem;
  }

  .clear-input:hover {
    background: var(--text-muted);
    color: var(--bg-primary);
  }

  .input-hint {
    margin-top: 0.75rem;
    font-size: 0.85rem;
    color: var(--text-muted);
  }

  .input-warning {
    margin-top: 0.75rem;
    font-size: 0.85rem;
    color: #ff6b6b;
  }

  /* Textarea */
  .textarea-wrapper {
    background: var(--bg-tertiary);
    border: 2px solid var(--border-color);
    border-radius: 12px;
    overflow: hidden;
    transition: border-color 0.2s ease;
  }

  .textarea-wrapper:focus-within {
    border-color: var(--accent-primary);
  }

  .text-input {
    width: 100%;
    background: transparent;
    border: none;
    outline: none;
    padding: 1.25rem;
    font-size: 1rem;
    color: var(--text-primary);
    font-family: var(--font-display);
    resize: vertical;
    min-height: 180px;
    line-height: 1.6;
  }

  .text-input::placeholder {
    color: var(--text-muted);
  }

  .textarea-footer {
    padding: 0.75rem 1.25rem;
    border-top: 1px solid var(--border-color);
    background: var(--bg-card);
  }

  .char-count {
    font-size: 0.8rem;
    color: var(--text-muted);
    font-family: var(--font-mono);
  }

  /* Search Input Section */
  .search-input-section {
    margin-bottom: 2.5rem;
  }

  .search-input-wrapper {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 0.75rem 1rem;
    background: var(--bg-tertiary);
    border: 2px solid var(--border-color);
    border-radius: 14px;
    transition: border-color 0.2s ease;
  }

  .search-input-wrapper:focus-within {
    border-color: var(--accent-secondary);
  }

  .search-icon {
    font-size: 1.5rem;
    opacity: 0.6;
  }

  .search-input {
    flex: 1;
    background: transparent;
    border: none;
    outline: none;
    font-size: 1.1rem;
    color: var(--text-primary);
    font-family: var(--font-display);
    padding: 0.5rem 0;
  }

  .search-input::placeholder {
    color: var(--text-muted);
  }

  .search-btn {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.85rem 1.75rem;
    background: var(--accent-gradient);
    border: none;
    border-radius: 10px;
    color: var(--bg-primary);
    font-weight: 600;
    font-size: 1rem;
    cursor: pointer;
    transition: all 0.2s ease;
    font-family: var(--font-display);
  }

  .search-btn:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(0, 168, 255, 0.35);
  }

  .search-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .search-arrow {
    transition: transform 0.2s ease;
  }

  .search-btn:hover:not(:disabled) .search-arrow {
    transform: translateX(3px);
  }

  .spinner {
    width: 18px;
    height: 18px;
    border: 2px solid rgba(10, 10, 15, 0.3);
    border-top-color: var(--bg-primary);
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }

  @keyframes spin {
    to { transform: rotate(360deg); }
  }

  /* Results Section */
  .results-section {
    padding-top: 2rem;
    border-top: 1px solid var(--border-color);
  }

  .results-list {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .result-card {
    background: var(--bg-card);
    border: 1px solid var(--border-color);
    border-radius: 12px;
    padding: 1.5rem;
    transition: all 0.3s ease;
    animation: fadeInUp 0.4s ease forwards;
    opacity: 0;
  }

  .result-card:hover {
    border-color: var(--accent-secondary);
    box-shadow: 0 4px 20px rgba(0, 168, 255, 0.1);
  }

  .result-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;
  }

  .result-relevance {
    display: inline-block;
    padding: 0.35rem 0.75rem;
    background: linear-gradient(135deg, rgba(0, 212, 170, 0.2), rgba(0, 168, 255, 0.2));
    border-radius: 20px;
    font-size: 0.8rem;
    font-weight: 600;
    color: var(--accent-primary);
    font-family: var(--font-mono);
  }

  .result-location {
    font-size: 0.85rem;
    color: var(--text-muted);
    font-family: var(--font-mono);
  }

  .result-snippet {
    color: var(--text-secondary);
    line-height: 1.6;
    margin-bottom: 1rem;
  }

  .result-action {
    background: transparent;
    border: 1px solid var(--border-color);
    padding: 0.5rem 1rem;
    border-radius: 6px;
    color: var(--text-secondary);
    font-size: 0.85rem;
    cursor: pointer;
    transition: all 0.2s ease;
    font-family: var(--font-display);
  }

  .result-action:hover {
    border-color: var(--accent-secondary);
    color: var(--accent-secondary);
  }

  /* Footer */
  .footer {
    padding: 2rem;
    text-align: center;
    color: var(--text-muted);
    font-size: 0.85rem;
    border-top: 1px solid var(--border-color);
    position: relative;
    z-index: 1;
  }

  /* Responsive */
  @media (max-width: 768px) {
    .header {
      padding: 1rem 1.5rem;
    }

    .main {
      padding: 1.5rem;
    }

    .title {
      font-size: 2.5rem;
    }

    .modality-grid {
      grid-template-columns: 1fr;
    }

    .search-panel {
      padding: 1.5rem;
    }

    .panel-header {
      flex-direction: column;
      text-align: center;
    }

    .search-input-wrapper {
      flex-direction: column;
      padding: 1rem;
    }

    .search-input {
      width: 100%;
      text-align: center;
    }

    .search-btn {
      width: 100%;
      justify-content: center;
    }
  }
</style>
