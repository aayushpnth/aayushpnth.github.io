---
layout: home
title: Aayush Pantha – Red Team Operator
---

<div class="text-center my-5 py-5">
  <h1 class="display-3 fw-bold text-danger">AAYUSH PANTHA</h1>
  <p class="lead fs-4 text-muted">Blue Teamer Turned Red Teamer</p>
  <p class="fs-5 mb-4">
    <span class="badge bg-dark me-1">CRTP</span>
    <span class="badge bg-dark me-1">CompTIA PenTest+</span>
    <span class="badge bg-dark me-1">ACP</span>
    <span class="badge bg-dark">CASA</span>
  </p>
  <p class="fs-4 text-primary">I break enterprise defenses for vendors. Palo Alto, Cloudflare, AWS, Fortinet — they bleed under my payloads.</p>
</div>

<div class="row g-4">
  <div class="col-md-6">
    <div class="card border-danger shadow-sm h-100">
      <div class="card-body">
        <h3 class="card-title text-danger">What I Do</h3>
        <ul class="list-unstyled fs-5">
          <li>Lead ransomware/EDR/C2/AD carnage at SecureIQLab</li>
          <li>Craft 500+ Cobalt Strike profiles & 700+ ransomware mutants</li>
          <li>Own Active Directory forests (Golden tickets, DCSync, delegation abuse)</li>
          <li>Expose vendor hype vs. real-world survival rate</li>
        </ul>
      </div>
    </div>
  </div>

  <div class="col-md-6">
    <div class="card border-primary shadow-sm h-100">
      <div class="card-body">
        <h3 class="card-title text-primary">Quick Links</h3>
        <ul class="list-unstyled fs-5">
          <li><a href="/about/" class="text-decoration-none">→ About Me (skills, tools, kills)</a></li>
          <li><a href="/posts/" class="text-decoration-none">→ All Posts & Writeups</a></li>
          <li><a href="/categories/" class="text-decoration-none">→ Categories (HackTheBox, AD, Ransomware...)</a></li>
          <li><a href="/tags/" class="text-decoration-none">→ Tags (c2, ofbiz, bloodhound...)</a></li>
        </ul>
      </div>
    </div>
  </div>
</div>

<div class="mt-5">
  <h2 class="text-center mb-4">Latest Drops</h2>
  <div class="row g-4">
    {% for post in site.posts limit: 4 %}
      <div class="col-md-6 col-lg-3">
        <div class="card h-100 shadow-sm">
          <div class="card-body">
            <h5 class="card-title"><a href="{{ post.url }}" class="text-decoration-none">{{ post.title }}</a></h5>
            <p class="card-text text-muted small">
              {{ post.date | date: "%b %-d, %Y" }}
              {% if post.categories.size > 0 %}
                • {{ post.categories | join: ", " }}
              {% endif %}
            </p>
          </div>
        </div>
      </div>
    {% endfor %}
  </div>
</div>

<div class="text-center mt-5 py-4">
  <p class="fs-4 fw-bold text-danger">Fuck marketing. I deliver metrics that make vendors sweat.</p>
  <p class="fs-5">– Aayush</p>
</div>