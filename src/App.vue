<script setup>
import { ref, computed } from 'vue'


const regexPattern = ref('[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}')
const testPayloadText = ref('Send your inquiries to contact@adhieeeh.dev or engineering-team@example.org for verification.')
const flagGlobal = ref(true)
const flagIgnoreCase = ref(true)
const flagMultiline = ref(false)


const compiledFlags = computed(() => {
  let flags = ''
  if (flagGlobal.value) flags += 'g'
  if (flagIgnoreCase.value) flags += 'i'
  if (flagMultiline.value) flags += 'm'
  return flags
})


const regexEvaluationMatrix = computed(() => {
  
  if (!regexPattern.value.trim()) {
    return { error: null, matchCount: 0, highlightedHtml: testPayloadText.value }
  }

  try {
    
    const nativeRegex = new RegExp(regexPattern.value, compiledFlags.value)
    
    
    const textToSearch = testPayloadText.value
    const allMatches = textToSearch.match(nativeRegex)
    const matchCount = allMatches ? allMatches.length : 0

    
    let escapedText = textToSearch
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')

    
    if (matchCount > 0) {
      if (flagGlobal.value) {
        
        escapedText = escapedText.replace(nativeRegex, (matchedToken) => {
          return `<mark class="regex-token-highlight">${matchedToken}</mark>`
        })
      } else {
        
        const singleMatchRegex = new RegExp(regexPattern.value, compiledFlags.value.replace('g', ''))
        escapedText = escapedText.replace(singleMatchRegex, (matchedToken) => {
          return `<mark class="regex-token-highlight">${matchedToken}</mark>`
        })
      }
    }

    return { error: null, matchCount, highlightedHtml: escapedText }
  } catch (err) {
    
    return { error: `Syntactic Pattern Compilation Fault: ${err.message}`, matchCount: 0, highlightedHtml: testPayloadText.value }
  }
})
</script>

<template>
  <div class="app-wrapper">
    
  
    <header class="app-header">
      <h1> DevRegex Sandbox</h1>
      <p>An interactive Vue 3 client-side pattern compiler designed to analyze regular expressions and tokenize test streams live.</p>
    </header>

   
    <main class="studio-desk">
      
      
      <section class="control-panel">
        <div class="panel-header">
          <h3>Pattern Configuration Engine</h3>
          <span 
            class="status-badge" 
            :class="regexEvaluationMatrix.error ? 'badge-error' : 'badge-success'"
          >
            {{ regexEvaluationMatrix.error ? 'MALFORMED' : `MATCHES: ${regexEvaluationMatrix.matchCount}` }}
          </span>
        </div>

       
        <div class="control-group">
          <label>Regular Expression Pattern (Raw String)</label>
          <div class="regex-input-wrapper">
            <span class="regex-delimiter">/</span>
            <input type="text" v-model="regexPattern" placeholder="Enter lookahead or parsing tokens...">
            <span class="regex-delimiter">/{{ compiledFlags }}</span>
          </div>
        </div>

        
        <div class="flags-matrix-row">
          <label class="flag-checkbox-container">
            <input type="checkbox" v-model="flagGlobal">
            <div class="flag-label-block">
              <strong>Global Matching (-g)</strong>
              <span>Finds all instances instead of exiting on the first match.</span>
            </div>
          </label>

          <label class="flag-checkbox-container">
            <input type="checkbox" v-model="flagIgnoreCase">
            <div class="flag-label-block">
              <strong>Case Insensitive (-i)</strong>
              <span>Ignores differences between lower and uppercase characters.</span>
            </div>
          </label>

          <label class="flag-checkbox-container">
            <input type="checkbox" v-model="flagMultiline">
            <div class="flag-label-block">
              <strong>Multi-line Target (-m)</strong>
              <span>Causes boundary tokens ^ and $ to match start/end lines.</span>
            </div>
          </label>
        </div>

        <!-- Live Compiler Error Alert Tray -->
        <div v-if="regexEvaluationMatrix.error" class="error-alert-tray">
          {{ regexEvaluationMatrix.error }}
        </div>
      </section>

      
      <section class="visualizer-panel">
        <h3>Test Ingestion Buffer Stream</h3>
        
        <textarea 
          v-model="testPayloadText" 
          placeholder="Paste or write standard code or log syntax strings here to test matching rules..." 
          class="test-string-input"
        ></textarea>

        <h3>Visualizer Token Render Canvas</h3>
        <div class="highlight-canvas-box">
          
          <pre v-html="regexEvaluationMatrix.highlightedHtml" class="rendered-preview-code"></pre>
        </div>
      </section>

    </main>
  </div>
</template>

<style>

.regex-token-highlight {
  background-color: rgba(245, 158, 11, 0.25);
  border-bottom: 2px solid #f59e0b;
  color: #fbbf24;
  font-weight: bold;
  padding: 1px 2px;
  border-radius: 3px;
}
</style>

<style scoped>
.app-wrapper {
  max-width: 1200px;
  margin: 40px auto;
  padding: 0 24px;
  font-family: monospace;
  color: #f8fafc;
}

.app-header {
  border-bottom: 2px solid #1e293b;
  padding-bottom: 20px;
  margin-bottom: 35px;
}

.app-header h1 {
  margin: 0;
  font-size: 30px;
  color: #38bdf8;
  letter-spacing: -0.5px;
}

.app-header p {
  margin: 4px 0 0 0;
  color: #64748b;
  font-size: 14px;
}

.studio-desk {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(360px, 1fr));
  gap: 40px;
}

.control-panel {
  background-color: #0f172a;
  border: 1px solid #1e293b;
  padding: 25px;
  border-radius: 16px;
  box-shadow: 0 10px 15px -3px rgba(0,0,0,0.3);
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
  border-bottom: 1px solid #1e293b;
  padding-bottom: 12px;
}

.panel-header h3 {
  margin: 0;
  font-size: 14px;
  color: #64748b;
  text-transform: uppercase;
}

.status-badge {
  font-size: 11px;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 6px;
}

.badge-success {
  color: #10b981;
  background-color: rgba(16, 185, 129, 0.1);
  border: 1px solid rgba(16, 185, 129, 0.2);
}

.badge-error {
  color: #ef4444;
  background-color: rgba(239, 68, 68, 0.1);
  border: 1px solid rgba(239, 68, 68, 0.2);
}

.control-group {
  margin-bottom: 25px;
}

.control-group label {
  display: block;
  font-size: 12px;
  font-weight: 700;
  color: #cbd5e1;
  margin-bottom: 8px;
}

.regex-input-wrapper {
  display: flex;
  align-items: center;
  background-color: #020617;
  border: 1px solid #1e293b;
  border-radius: 8px;
  padding: 0 14px;
}

.regex-delimiter {
  color: #475569;
  font-size: 16px;
  font-weight: bold;
}

.regex-input-wrapper input {
  flex: 1;
  background: none;
  border: none;
  padding: 12px 10px;
  color: #38bdf8;
  font-family: monospace;
  font-size: 14px;
  outline: none;
}

.flags-matrix-row {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.flag-checkbox-container {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  cursor: pointer;
  padding: 10px;
  border-radius: 8px;
  transition: background-color 0.2s;
}

.flag-checkbox-container:hover {
  background-color: #020617;
}

.flag-checkbox-container input[type="checkbox"] {
  margin-top: 3px;
  accent-color: #38bdf8;
  cursor: pointer;
}

.flag-label-block strong {
  display: block;
  font-size: 13px;
  color: #cbd5e1;
}

.flag-label-block span {
  display: block;
  font-size: 11px;
  color: #475569;
  margin-top: 2px;
}

.error-alert-tray {
  margin-top: 20px;
  background-color: rgba(239, 68, 68, 0.05);
  border-left: 4px solid #ef4444;
  padding: 12px 16px;
  border-radius: 0 8px 8px 0;
  color: #f87171;
  font-size: 12px;
  line-height: 1.5;
}

.visualizer-panel h3 {
  margin-top: 0;
  margin-bottom: 12px;
  font-size: 13px;
  color: #475569;
  text-transform: uppercase;
}

.test-string-input {
  width: 100%;
  height: 120px;
  background-color: #0f172a;
  border: 1px solid #1e293b;
  border-radius: 12px;
  padding: 15px;
  color: #cbd5e1;
  font-family: monospace;
  font-size: 13px;
  line-height: 1.6;
  resize: vertical;
  outline: none;
  box-sizing: border-box;
  margin-bottom: 25px;
}

.test-string-input:focus {
  border-color: #38bdf8;
}

.highlight-canvas-box {
  background-color: #020617;
  border: 1px dashed #334155;
  border-radius: 16px;
  padding: 25px;
  min-height: 160px;
  overflow: auto;
}

.rendered-preview-code {
  margin: 0;
  font-family: monospace;
  font-size: 13px;
  line-height: 1.6;
  white-space: pre-wrap;
  word-break: break-all;
  color: #94a3b8;
}
</style>