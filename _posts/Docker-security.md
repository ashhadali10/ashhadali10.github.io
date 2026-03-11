---
title: "Securing Docker Containers — Practical Hands-On Guide"
date: 2025-11-23
permalink: /docker-security/
layout: default
---

&lt;style&gt;
/* Article Styles - Matching Your Theme */
.article-container {
    max-width: 800px;
    margin: 0 auto;
    padding: 2rem 1rem;
    color: #cbd5e1;
    font-family: 'Inter', sans-serif;
    line-height: 1.7;
}

/* Big Title */
.article-title {
    font-size: 2.5rem;
    font-weight: 700;
    color: #ffffff;
    margin-bottom: 1rem;
    line-height: 1.2;
    background: linear-gradient(to right, #22d3ee, #34d399);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
}

.article-meta {
    color: #64748b;
    font-size: 0.9rem;
    margin-bottom: 2rem;
    padding-bottom: 1rem;
    border-bottom: 1px solid #334155;
}

/* Headings - Cyan Color */
.article-container h2 {
    color: #22d3ee;
    font-size: 1.8rem;
    font-weight: 600;
    margin-top: 2.5rem;
    margin-bottom: 1rem;
    padding-bottom: 0.5rem;
    border-bottom: 2px solid #1e293b;
}

.article-container h3 {
    color: #34d399;
    font-size: 1.4rem;
    font-weight: 600;
    margin-top: 2rem;
    margin-bottom: 0.8rem;
}

.article-container h4 {
    color: #22d3ee;
    font-size: 1.2rem;
    font-weight: 600;
    margin-top: 1.5rem;
    margin-bottom: 0.6rem;
}

/* Paragraphs */
.article-container p {
    margin-bottom: 1.2rem;
    color: #cbd5e1;
}

/* Links */
.article-container a {
    color: #22d3ee;
    text-decoration: none;
    border-bottom: 1px dotted #22d3ee;
    transition: all 0.3s;
}

.article-container a:hover {
    color: #34d399;
    border-bottom-color: #34d399;
}

/* Code Blocks - Dark Theme */
.article-container pre {
    background: #0f172a;
    border: 1px solid #334155;
    border-radius: 8px;
    padding: 1.2rem;
    overflow-x: auto;
    margin: 1.5rem 0;
    position: relative;
}

.article-container pre::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 3px;
    background: linear-gradient(to right, #22d3ee, #34d399);
    border-radius: 8px 8px 0 0;
}

.article-container code {
    font-family: 'JetBrains Mono', monospace;
    font-size: 0.9rem;
    color: #e2e8f0;
}

/* Inline Code */
.article-container p code, .article-container li code {
    background: #1e293b;
    color: #22d3ee;
    padding: 0.2rem 0.4rem;
    border-radius: 4px;
    font-size: 0.85rem;
    border: 1px solid #334155;
}

/* Lists */
.article-container ul, .article-container ol {
    margin-bottom: 1.2rem;
    padding-left: 1.5rem;
}

.article-container li {
    margin-bottom: 0.5rem;
    color: #cbd5e1;
}

.article-container li::marker {
    color: #22d3ee;
}

/* Table of Contents */
.toc {
    background: #1e293b;
    border: 1px solid #334155;
    border-radius: 12px;
    padding: 1.5rem;
    margin: 2rem 0;
    position: relative;
    overflow: hidden;
}

.toc::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 4px;
    height: 100%;
    background: linear-gradient(to bottom, #22d3ee, #34d399);
}

.toc h3 {
    color: #ffffff;
    margin-top: 0;
    margin-bottom: 1rem;
    font-size: 1.2rem;
}

.toc ul {
    list-style: none;
    padding-left: 0;
    margin-bottom: 0;
}

.toc li {
    margin-bottom: 0.5rem;
}

.toc a {
    color: #94a3b8;
    text-decoration: none;
    border-bottom: none;
    display: block;
    padding: 0.3rem 0;
    transition: all 0.3s;
}

.toc a:hover {
    color: #22d3ee;
    padding-left: 0.5rem;
}

.toc a::before {
    content: '→';
    margin-right: 0.5rem;
    color: #22d3ee;
    opacity: 0;
    transition: opacity 0.3s;
}

.toc a:hover::before {
    opacity: 1;
}

/* Tables */
.article-container table {
    width: 100%;
    border-collapse: collapse;
    margin: 1.5rem 0;
    background: #1e293b;
    border-radius: 8px;
    overflow: hidden;
}

.article-container th {
    background: #0f172a;
    color: #22d3ee;
    font-weight: 600;
    padding: 0.8rem;
    text-align: left;
    border-bottom: 2px solid #334155;
}

.article-container td {
    padding: 0.8rem;
    border-bottom: 1px solid #334155;
    color: #cbd5e1;
}

.article-container tr:hover {
    background: #252f47;
}

/* Blockquote */
.article-container blockquote {
    border-left: 4px solid #22d3ee;
    background: #1e293b;
    padding: 1rem 1.5rem;
    margin: 1.5rem 0;
    border-radius: 0 8px 8px 0;
    font-style: italic;
    color: #94a3b8;
}

/* Strong/Bold */
.article-container strong {
    color: #ffffff;
    font-weight: 600;
}

/* Back to Blog Button */
.back-to-blog {
    display: inline-flex;
    align-items: center;
    color: #22d3ee;
    text-decoration: none;
    margin-bottom: 2rem;
    font-weight: 500;
    transition: all 0.3s;
}

.back-to-blog:hover {
    color: #34d399;
    transform: translateX(-5px);
}

.back-to-blog i {
    margin-right: 0.5rem;
}

/* Copy Button for Code Blocks */
.code-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #1e293b;
    padding: 0.5rem 1rem;
    border-radius: 8px 8px 0 0;
    border: 1px solid #334155;
    border-bottom: none;
    margin-top: 1.5rem;
}

.code-header span {
    color: #64748b;
    font-size: 0.8rem;
    font-family: 'JetBrains Mono', monospace;
}

.copy-btn {
    background: transparent;
    border: 1px solid #334155;
    color: #64748b;
    padding: 0.3rem 0.8rem;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.8rem;
    transition: all 0.3s;
}

.copy-btn:hover {
    background: #22d3ee;
    color: #0f172a;
    border-color: #22d3ee;
}

/* Responsive */
@media (max-width: 768px) {
    .article-title {
        font-size: 1.8rem;
    }
    
    .article-container h2 {
        font-size: 1.4rem;
    }
    
    .article-container h3 {
        font-size: 1.2rem;
    }
}
&lt;/style&gt;

&lt;link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet"&gt;
&lt;link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css"&gt;

&lt;div class="article-container" style="padding-top: 100px;"&gt;
    
    &lt;a href="/blog/" class="back-to-blog"&gt;
        &lt;i class="fas fa-arrow-left"&gt;&lt;/i&gt; Back to Blog
    &lt;/a&gt;

    &lt;h1 class="article-title"&gt;Securing Docker Containers — Practical Hands-On Guide&lt;/h1&gt;
    
    &lt;div class="article-meta"&gt;
        &lt;i class="far fa-calendar-alt"&gt;&lt;/i&gt; November 23, 2025 &nbsp;•&nbsp; 
        &lt;i class="far fa-clock"&gt;&lt;/i&gt; 15 min read &nbsp;•&nbsp; 
        &lt;i class="fas fa-tag"&gt;&lt;/i&gt; Docker, Security, DevSecOps
    &lt;/div&gt;

    &lt;p&gt;&lt;strong&gt;Learn production-grade techniques to secure Docker containers:&lt;/strong&gt; non-root users, Docker Content Trust, Docker Scout scanning, capability limiting, read-only filesystems, and a full hardened compose example.&lt;/p&gt;

    &lt;blockquote&gt;
        &lt;strong&gt;Hook — A Hard Truth Every Senior Engineer Knows:&lt;/strong&gt; If a container in your fleet is running as root, it's not a "sandbox" — it's a live bridge to the host. In real incidents, attackers have used unscanned images, writable containers, or excessive Linux capabilities to escalate from an app process to host compromise.
    &lt;/blockquote&gt;

    &lt;!-- Table of Contents --&gt;
    &lt;div class="toc"&gt;
        &lt;h3&gt;&lt;i class="fas fa-list-ul"&gt;&lt;/i&gt; Table of Contents&lt;/h3&gt;
        &lt;ul&gt;
            &lt;li&gt;&lt;a href="#objectives"&gt;1. Objectives & Prerequisites&lt;/a&gt;&lt;/li&gt;
            &lt;li&gt;&lt;a href="#lab-environment"&gt;2. Lab Environment Overview&lt;/a&gt;&lt;/li&gt;
            &lt;li&gt;&lt;a href="#task1"&gt;3. Task 1 — Run Containers as Non-Root Users&lt;/a&gt;&lt;/li&gt;
            &lt;li&gt;&lt;a href="#task2"&gt;4. Task 2 — Docker Content Trust&lt;/a&gt;&lt;/li&gt;
            &lt;li&gt;&lt;a href="#task3"&gt;5. Task 3 — Security Scanning with Docker Scout&lt;/a&gt;&lt;/li&gt;
            &lt;li&gt;&lt;a href="#task4"&gt;6. Task 4 — Limit Container Privileges&lt;/a&gt;&lt;/li&gt;
            &lt;li&gt;&lt;a href="#task5"&gt;7. Task 5 — Read-Only Filesystems&lt;/a&gt;&lt;/li&gt;
            &lt;li&gt;&lt;a href="#task6"&gt;8. Task 6 — Comprehensive Secure Docker Compose&lt;/a&gt;&lt;/li&gt;
            &lt;li&gt;&lt;a href="#troubleshooting"&gt;9. Troubleshooting & Common Mistakes&lt;/a&gt;&lt;/li&gt;
            &lt;li&gt;&lt;a href="#cleanup"&gt;10. Lab Cleanup&lt;/a&gt;&lt;/li&gt;
            &lt;li&gt;&lt;a href="#takeaways"&gt;11. Key Takeaways & Next Steps&lt;/a&gt;&lt;/li&gt;
        &lt;/ul&gt;
    &lt;/div&gt;

    &lt;h2 id="objectives"&gt;1. Objectives & Prerequisites&lt;/h2&gt;
    
    &lt;h3&gt;What You'll Achieve&lt;/h3&gt;
    &lt;p&gt;By the end of this article you will be able to:&lt;/p&gt;
    &lt;ul&gt;
        &lt;li&gt;Implement Docker security best practices for containerized apps&lt;/li&gt;
        &lt;li&gt;Run containers as non-root users using &lt;code&gt;USER&lt;/code&gt; in Dockerfile&lt;/li&gt;
        &lt;li&gt;Set up Docker Content Trust and sign images&lt;/li&gt;
        &lt;li&gt;Use Docker Scout to scan images for CVEs and compare bases&lt;/li&gt;
        &lt;li&gt;Limit container privileges by dropping capabilities and applying security options&lt;/li&gt;
        &lt;li&gt;Make containers read-only and use tmpfs for writable runtime storage&lt;/li&gt;
        &lt;li&gt;Combine all controls in a production-grade docker-compose deployment&lt;/li&gt;
    &lt;/ul&gt;

    &lt;h3&gt;Prerequisites&lt;/h3&gt;
    &lt;ul&gt;
        &lt;li&gt;Basic Docker knowledge (images, containers, Dockerfile)&lt;/li&gt;
        &lt;li&gt;Linux CLI familiarity&lt;/li&gt;
        &lt;li&gt;Understanding of user permissions and filesystems in Linux&lt;/li&gt;
    &lt;/ul&gt;

    &lt;h2 id="task1"&gt;3. Task 1 — Run Containers as Non-Root Users&lt;/h2&gt;
    
    &lt;h3&gt;What It Is&lt;/h3&gt;
    &lt;p&gt;By default, Docker containers run as &lt;code&gt;root&lt;/code&gt; (UID 0). If an attacker escapes the container, they have root on the host. Running as a non-root user limits the blast radius.&lt;/p&gt;

    &lt;h3&gt;Implementation&lt;/h3&gt;
    
    &lt;div class="code-header"&gt;
        &lt;span&gt;Dockerfile&lt;/span&gt;
        &lt;button class="copy-btn" onclick="copyCode(this)"&gt;Copy&lt;/button&gt;
    &lt;/div&gt;
&lt;pre&gt;&lt;code&gt;# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Set proper ownership
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Expose non-privileged port
EXPOSE 8080&lt;/code&gt;&lt;/pre&gt;

    &lt;h2 id="task2"&gt;4. Task 2 — Docker Content Trust&lt;/h2&gt;
    
    &lt;h3&gt;What It Is&lt;/h3&gt;
    &lt;p&gt;Docker Content Trust (DCT) uses cryptographic signatures (Notary) to ensure image integrity and authenticity.&lt;/p&gt;

    &lt;h3&gt;Why It Matters&lt;/h3&gt;
    &lt;p&gt;Prevents pulling tampered or malicious images — essential for supply-chain security and compliance.&lt;/p&gt;

    &lt;blockquote&gt;
        &lt;strong&gt;Analogy:&lt;/strong&gt; DCT is a tamper-evident seal on a package: if it's broken, you don't accept the package.
    &lt;/blockquote&gt;

    &lt;h3&gt;Enable DCT&lt;/h3&gt;
    
    &lt;div class="code-header"&gt;
        &lt;span&gt;bash&lt;/span&gt;
        &lt;button class="copy-btn" onclick="copyCode(this)"&gt;Copy&lt;/button&gt;
    &lt;/div&gt;
&lt;pre&gt;&lt;code&gt;# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Verify before pull
docker pull alpine:latest

# Sign your own images
docker trust sign myregistry/myimage:tag&lt;/code&gt;&lt;/pre&gt;

    &lt;h2 id="task6"&gt;8. Task 6 — Comprehensive Secure Docker Compose&lt;/h2&gt;

    &lt;p&gt;Here's a production-ready secure docker-compose configuration:&lt;/p&gt;

    &lt;div class="code-header"&gt;
        &lt;span&gt;docker-compose.secure.yml&lt;/span&gt;
        &lt;button class="copy-btn" onclick="copyCode(this)"&gt;Copy&lt;/button&gt;
    &lt;/div&gt;
&lt;pre&gt;&lt;code&gt;version: '3.8'

services:
  secure-app:
    build:
      context: .
      dockerfile: Dockerfile.secure
    image: secure-app:v1
    user: "1000:1000"
    read_only: true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    security_opt:
      - no-new-privileges:true
      - seccomp:./seccomp-profile.json
    tmpfs:
      - /tmp:noexec,nosuid,size=100m
      - /var/cache:size=50m
    environment:
      - NODE_ENV=production
    ports:
      - "8080:8080"
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3&lt;/code&gt;&lt;/pre&gt;

    &lt;h2 id="cleanup"&gt;10. Lab Cleanup&lt;/h2&gt;
    
    &lt;p&gt;Clean up all resources after completing the lab:&lt;/p&gt;

    &lt;div class="code-header"&gt;
        &lt;span&gt;bash&lt;/span&gt;
        &lt;button class="copy-btn" onclick="copyCode(this)"&gt;Copy&lt;/button&gt;
    &lt;/div&gt;
&lt;pre&gt;&lt;code&gt;# Stop and remove containers
docker-compose -f docker-compose.secure.yml down

# Stop all running containers
docker stop $(docker ps -aq) 2&gt;/dev/null || true

# Remove all containers
docker rm $(docker ps -aq) 2&gt;/dev/null || true

# Remove images
docker rmi secure-app:v1 secure-app:v2 readonly-app:v1 2&gt;/dev/null || true

# Stop local registry
docker stop registry && docker rm registry 2&gt;/dev/null || true

# Clean directories and files
rm -rf secure-app/
rm -f *.json *.pub mykey

# Unset environment variable
unset DOCKER_CONTENT_TRUST

# Verify capabilities (debugging)
strace -e trace=capget,capset docker run --rm your-image

# Add capability only if justified
# --cap-add=NET_ADMIN&lt;/code&gt;&lt;/pre&gt;

    &lt;h2 id="troubleshooting"&gt;9. Troubleshooting & Common Mistakes&lt;/h2&gt;

    &lt;table&gt;
        &lt;thead&gt;
            &lt;tr&gt;
                &lt;th&gt;Mistake&lt;/th&gt;
                &lt;th&gt;Fix&lt;/th&gt;
            &lt;/tr&gt;
        &lt;/thead&gt;
        &lt;tbody&gt;
            &lt;tr&gt;
                &lt;td&gt;Forgetting &lt;code&gt;chown&lt;/code&gt; → app files owned by root cause permission errors&lt;/td&gt;
                &lt;td&gt;&lt;code&gt;chown -R appuser:appuser /app&lt;/code&gt; before &lt;code&gt;USER&lt;/code&gt;&lt;/td&gt;
            &lt;/tr&gt;
            &lt;tr&gt;
                &lt;td&gt;Using USER but still exposing privileged ports (&lt;1024) without CAP&lt;/td&gt;
                &lt;td&gt;Use non-root ports (e.g., 8080) or add minimal capability &lt;code&gt;NET_BIND_SERVICE&lt;/code&gt; only&lt;/td&gt;
            &lt;/tr&gt;
            &lt;tr&gt;
                &lt;td&gt;Read-only filesystem breaks apps that need /tmp&lt;/td&gt;
                &lt;td&gt;Mount &lt;code&gt;tmpfs&lt;/code&gt; for temporary writable directories&lt;/td&gt;
            &lt;/tr&gt;
            &lt;tr&gt;
                &lt;td&gt;Dropping ALL capabilities breaks legitimate functions&lt;/td&gt;
                &lt;td&gt;Test thoroughly and add back only required capabilities&lt;/td&gt;
            &lt;/tr&gt;
        &lt;/tbody&gt;
    &lt;/table&gt;

    &lt;h2 id="takeaways"&gt;11. Key Takeaways & Next Steps&lt;/h2&gt;
    
    &lt;ul&gt;
        &lt;li&gt;&lt;strong&gt;Never run as root&lt;/strong&gt; — always use &lt;code&gt;USER&lt;/code&gt; directive&lt;/li&gt;
        &lt;li&gt;&lt;strong&gt;Sign your images&lt;/strong&gt; — enable Docker Content Trust in production&lt;/li&gt;
        &lt;li&gt;&lt;strong&gt;Scan before deploy&lt;/strong&gt; — integrate Docker Scout in CI/CD&lt;/li&gt;
        &lt;li&gt;&lt;strong&gt;Principle of least privilege&lt;/strong&gt; — drop all caps, add back minimally&lt;/li&gt;
        &lt;li&gt;&lt;strong&gt;Immutable infrastructure&lt;/strong&gt; — use read-only root filesystems&lt;/li&gt;
    &lt;/ul&gt;

    &lt;p&gt;&lt;strong&gt;Next Steps:&lt;/strong&gt; Apply these patterns to your existing Docker workloads. Start with non-root users, then layer on additional security controls progressively.&lt;/p&gt;

    &lt;hr style="border-color: #334155; margin: 3rem 0;"&gt;
    
    &lt;p style="text-align: center; color: #64748b;"&gt;
        &lt;i class="fas fa-shield-alt"&gt;&lt;/i&gt; Happy Securing! | 
        &lt;a href="/blog/"&gt;Back to Blog&lt;/a&gt;
    &lt;/p&gt;

&lt;/div&gt;

&lt;script&gt;
// Copy code functionality
function copyCode(btn) {
    const pre = btn.closest('.code-header').nextElementSibling;
    const code = pre.querySelector('code') || pre;
    const text = code.innerText;
    
    navigator.clipboard.writeText(text).then(() =&gt; {
        const original = btn.innerText;
        btn.innerText = 'Copied!';
        btn.style.background = '#22d3ee';
        btn.style.color = '#0f172a';
        
        setTimeout(() =&gt; {
            btn.innerText = original;
            btn.style.background = 'transparent';
            btn.style.color = '#64748b';
        }, 2000);
    });
}

// Smooth scroll for TOC links
document.querySelectorAll('.toc a').forEach(anchor =&gt; {
    anchor.addEventListener('click', function (e) {
        e.preventDefault();
        const target = document.querySelector(this.getAttribute('href'));
        if (target) {
            target.scrollIntoView({
                behavior: 'smooth',
                block: 'start'
            });
        }
    });
});
&lt;/script&gt;
