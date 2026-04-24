
<style>
.garden-card-grid {
    display: grid !important;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)) !important;
    gap: 1.5rem !important;
    padding: 1rem 0 !important;
    margin: 2rem 0 !important;
}

.garden-card {
    position: relative !important;
    background: rgba(45, 55, 72, 0.6) !important;
    border: 2px solid #D4AF37 !important; 
    border-radius: 16px !important;
    padding: 1.5rem !important;
    text-decoration: none !important;
    transition: all 0.3s ease !important;
    display: flex !important;
    flex-direction: column !important;
    height: 100% !important;
    backdrop-filter: blur(10px) !important;
    -webkit-backdrop-filter: blur(10px) !important;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1) !important;
    color: inherit !important;
}

.garden-card:hover {
    transform: translateY(-5px) !important;
    background: rgba(60, 80, 110, 0.8) !important;
    border-color: #D4AF37 !important;
    box-shadow: 0 10px 30px -10px rgba(212, 175, 55, 0.5) !important;
}

.card-icon {
    font-size: 1.5rem !important;
    margin-bottom: 1rem !important;
    display: block !important;
}

.card-title {
    font-size: 1.25rem !important;
    font-weight: 700 !important;
    color: #D4AF37 !important;
    margin-bottom: 0.5rem !important;
    display: block !important;
    text-decoration: none !important;
}

.card-desc {
    font-size: 0.9rem !important;
    color: #a0aec0 !important;
    line-height: 1.5 !important;
}

.garden-card * {
    color: inherit !important;
    text-decoration: none !important;
}
</style>


# AI Application Engineering Hub

<div class="garden-card-grid">
    <a href="articles/ai-engineering/prompt-patterns.md" class="garden-card">
        <span class="card-icon">🧩</span>
        <span class="card-title">Prompt Patterns</span>
        <span class="card-desc">Reducing the search space: Moving from magic spells to structural constraints.</span>
    </a>
    <a href="articles/ai-engineering/context-window-dynamics.md" class="garden-card">
        <span class="card-icon">🪟</span>
        <span class="card-title">Context Dynamics</span>
        <span class="card-desc">The Lost-in-the-Middle problem and the art of context pruning.</span>
    </a>
    <a href="articles/ai-engineering/rag-truth.md" class="garden-card">
        <span class="card-icon">🎯</span>
        <span class="card-title">RAG Truth</span>
        <span class="card-desc">Why vector similarity != relevance. The power of Hybrid Search and Re-ranking.</span>
    </a>
    <a href="articles/ai-engineering/local-ai-infra.md" class="garden-card">
        <span class="card-icon">🏗️</span>
        <span class="card-title">Local AI Infra</span>
        <span class="card-desc">From Proxmox GPU Passthrough to vLLM production pipelines.</span>
    </a>
    <a href="articles/ai-engineering/agent-engineering-rigor.md" class="garden-card">
        <span class="card-icon">🤖</span>
        <span class="card-title">Agent Engineering Rigor</span>
        <span class="card-desc">Constraining autonomy through state machines and iterative workflows.</span>
    </a>
</div>
