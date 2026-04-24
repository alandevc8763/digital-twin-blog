
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


# SRE & Infra Hub

<div class="garden-card-grid">
    <a href="articles/performance/cpu-numa.md" class="garden-card">
        <span class="card-icon">⚡</span>
        <span class="card-title">CPU Topology & NUMA</span>
        <span class="card-desc">The physics of memory access and how to eliminate the Remote Access penalty.</span>
    </a>
    <a href="articles/performance/cfs-scheduler.md" class="garden-card">
        <span class="card-icon">⚖️</span>
        <span class="card-title">CFS Scheduler</span>
        <span class="card-desc">Beyond fairness: How to achieve deterministic execution in Linux.</span>
    </a>
    <a href="articles/performance/psi-metrics.md" class="garden-card">
        <span class="card-icon">📉</span>
        <span class="card-title">PSI Metrics</span>
        <span class="card-desc">Stop looking at utilization. Measure system suffering via pressure stalls.</span>
    </a>
    <a href="articles/performance/vfs-tracing.md" class="garden-card">
        <span class="card-icon">🔍</span>
        <span class="card-title">VFS Tracing</span>
        <span class="card-desc">Uncovering the hidden I/O tax between syscalls and disk platters.</span>
    </a>
    <a href="articles/sre-stability-myth.md" class="garden-card">
        <span class="card-icon">🛡️</span>
        <span class="card-title">SRE Stability Myth</span>
        <span class="card-desc">The architectural truth about building systems that don't break.</span>
    </a>
</div>
