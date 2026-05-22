---
layout: page
title: Documentation
permalink: /docs/
---

[![Build Status](https://github.com/bhangun/iket/actions/workflows/ci.yml/badge.svg)](https://github.com/bhangun/iket/actions)
[![Go Report Card](https://goreportcard.com/badge/github.com/bhangun/iket)](https://goreportcard.com/report/github.com/bhangun/iket)
[![GitHub release](https://img.shields.io/github/v/tag/bhangun/iket?label=release)](https://github.com/bhangun/iket/releases)
[![License](https://img.shields.io/github/license/bhangun/iket)](https://github.com/bhangun/iket/blob/main/LICENSE)
[![Docker Pulls](https://img.shields.io/docker/pulls/bhangun/iket)](https://hub.docker.com/r/bhangun/iket)

# Iket Documentation

This website is now the canonical home for Iket documentation. Product guides are maintained here as browsable pages instead of being treated as repo-only markdown.

<div class="release-banner">
  <span class="release-banner__label">Latest release</span>
  <strong>{{ site.latest_version }}</strong>
  <span>Use this as the baseline when following install, CLI, and deployment guides.</span>
  <a href="{{ site.releases_url }}">Open releases</a>
</div>

<div class="callout">
  <p><strong>Suggested reading path:</strong> start with installation, move into CLI workflows, then use management, plugin, and reference docs as your deployment gets deeper.</p>
</div>

## Recent Updates

<div class="changefeed-grid">
  {% for item in site.data.changefeed %}
    <article class="changefeed-card">
      <span class="changefeed-card__label">{{ item.label }}</span>
      <h3><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h3>
      <p>{{ item.summary }}</p>
    </article>
  {% endfor %}
</div>

## Start Here

<div class="start-rail">
  <article class="start-rail__card">
    <span class="start-rail__step">Step 1</span>
    <h3><a href="{{ '/docs/install/' | relative_url }}">Install and bring up Iket</a></h3>
    <p>Use this first if you are starting from zero and need the quickest path to a running gateway or CLI-only setup.</p>
    <p class="start-rail__meta">Best for: first-time setup, Docker, local bring-up</p>
  </article>
  <article class="start-rail__card">
    <span class="start-rail__step">Step 2</span>
    <h3><a href="{{ '/docs/cli-commands/' | relative_url }}">Learn the CLI path</a></h3>
    <p>Use this next to verify access, inspect gateway state, and understand the safe change workflow before touching live configuration.</p>
    <p class="start-rail__meta">Best for: operators, rollout workflows, diagnostics</p>
  </article>
  <article class="start-rail__card">
    <span class="start-rail__step">Step 3</span>
    <h3><a href="{{ '/docs/management-api-integration/' | relative_url }}">Choose your deeper path</a></h3>
    <p>From here, continue into management API integration, plugin development, or deployment references depending on how you plan to run Iket.</p>
    <p class="start-rail__meta">Best for: automation, extension, production planning</p>
  </article>
</div>

## Docs Sections

<div class="section-grid">
  <article class="section-card">
    <span class="section-card__eyebrow">Start Here</span>
    <h3><a href="{{ '/docs/start-here/' | relative_url }}">Installation, CLI, and production basics</a></h3>
    <p>Use this path when you are new to Iket and want the most practical route from setup to safe operations.</p>
  </article>
  <article class="section-card">
    <span class="section-card__eyebrow">Core Docs</span>
    <h3><a href="{{ '/docs/core-docs/' | relative_url }}">Configuration and platform reference</a></h3>
    <p>Use this section when you need deeper reference material for config storage, editions, API shape, and route behavior.</p>
  </article>
  <article class="section-card">
    <span class="section-card__eyebrow">Plugin Docs</span>
    <h3><a href="{{ '/docs/plugin-docs/' | relative_url }}">Extensions and middleware patterns</a></h3>
    <p>Use this section when built-in features are not enough and you want to extend Iket with plugins or middleware.</p>
  </article>
  <article class="section-card">
    <span class="section-card__eyebrow">Reference Notes</span>
    <h3><a href="{{ '/docs/reference-notes/' | relative_url }}">Supporting technical notes</a></h3>
    <p>Use this area for lower-frequency but still useful supporting material around engineering and maintenance workflows.</p>
  </article>
</div>

## What You’ll Find Here

<div class="feature-grid">
  <div class="feature-card">
    <h3>Install and deploy</h3>
    <p>Bring up Iket locally, in Docker, or in production-ready environments.</p>
  </div>
  <div class="feature-card">
    <h3>Operate safely</h3>
    <p>Use the CLI, management APIs, mTLS, proposals, and rollout workflows cleanly.</p>
  </div>
  <div class="feature-card">
    <h3>Extend the gateway</h3>
    <p>Build plugins, middleware, and custom behaviors without fighting the core architecture.</p>
  </div>
  <div class="feature-card">
    <h3>Reference features</h3>
    <p>Review configuration, editions, route behavior, and supporting technical notes.</p>
  </div>
</div>

{% assign docs_pages = site.pages | where: "layout", "docs_page" | sort: "weight" | sort: "section_order" %}
{% capture audience_values %}{% for item in docs_pages %}{% for label in item.audience %}{{ label }}|{% endfor %}{% endfor %}{% endcapture %}
{% capture topic_values %}{% for item in docs_pages %}{% for label in item.topics %}{{ label }}|{% endfor %}{% endfor %}{% endcapture %}
{% assign audiences = audience_values | split: "|" | uniq | sort %}
{% assign topics = topic_values | split: "|" | uniq | sort %}

## Browse Docs

<div class="docs-search" data-docs-search-root>
  <label class="docs-search__label" for="docs-search-input">Find a doc fast</label>
  <div class="docs-search__bar">
    <input id="docs-search-input" class="docs-search__input" type="search" placeholder="Search by workflow, topic, or page name" data-docs-search-input>
    <button class="docs-search__clear" type="button" data-docs-search-clear hidden>Clear</button>
  </div>
  <p class="docs-search__status" data-docs-search-status>Search across all documentation cards.</p>
</div>

<div class="docs-filters" data-docs-filters>
  <div class="docs-filters__group">
    <p class="docs-filters__label">Audience</p>
    <div class="docs-filters__chips">
      <button class="docs-filters__chip is-active" type="button" data-filter-kind="audience" data-filter-value="">All audiences</button>
      {% for label in audiences %}
        {% unless label == "" %}
          <button class="docs-filters__chip" type="button" data-filter-kind="audience" data-filter-value="{{ label }}">{{ label }}</button>
        {% endunless %}
      {% endfor %}
    </div>
  </div>

  <div class="docs-filters__group">
    <p class="docs-filters__label">Topic</p>
    <div class="docs-filters__chips">
      <button class="docs-filters__chip is-active" type="button" data-filter-kind="topic" data-filter-value="">All topics</button>
      {% for label in topics %}
        {% unless label == "" %}
          <button class="docs-filters__chip" type="button" data-filter-kind="topic" data-filter-value="{{ label }}">{{ label }}</button>
        {% endunless %}
      {% endfor %}
    </div>
  </div>

  <div class="docs-filters__footer">
    <p class="docs-filters__status" data-filter-status>Showing all docs.</p>
    <div class="docs-filters__actions">
      <button class="docs-filters__reset" type="button" data-clear-filters>Clear filters</button>
      <button class="docs-filters__share" type="button" data-copy-filter-link>Copy filtered link</button>
    </div>
  </div>
</div>

## Quick Command Paths

<div class="docs-grid">
  <div class="doc-card docs-hub__card" data-doc-card data-doc-audience="operator|developer" data-doc-topics="installation|deployment|cli" data-doc-search-text="install setup bootstrap onboarding full gateway cli-only source build docker macos linux">
    <h3><a href="{{ '/docs/install/' | relative_url }}">Install and bootstrap</a></h3>
    <p>Use this when you need the fastest path from zero to a running gateway or CLI setup.</p>
  </div>
  <div class="doc-card docs-hub__card" data-doc-card data-doc-audience="operator|developer" data-doc-topics="cli|administration|operations" data-doc-search-text="cli commands setup context diff apply proposal canary logs cert operator workflow">
    <h3><a href="{{ '/docs/cli-commands/' | relative_url }}">Run operator workflows</a></h3>
    <p>Use this when you need the main `iket` commands for administration, rollout, and diagnostics.</p>
  </div>
  <div class="doc-card docs-hub__card" data-doc-card data-doc-audience="developer|operator" data-doc-topics="api|management|security" data-doc-search-text="api management mtls admin integration websocket endpoints automation client">
    <h3><a href="{{ '/docs/management-api-integration/' | relative_url }}">Integrate the management API</a></h3>
    <p>Use this when you need secure remote administration or API-driven control surface access.</p>
  </div>
  <div class="doc-card docs-hub__card" data-doc-card data-doc-audience="developer" data-doc-topics="plugins|extension|quickstart" data-doc-search-text="plugin development quickstart middleware extension golang go custom behavior">
    <h3><a href="{{ '/docs/plugin-quickstart/' | relative_url }}">Build a plugin</a></h3>
    <p>Use this when you want to extend Iket behavior without modifying the gateway core.</p>
  </div>
</div>

## Featured Docs

<div class="featured-docs-grid">
  {% for item in site.data.featured_docs %}
    <article class="featured-doc">
      <span class="featured-doc__tag">{{ item.tag }}</span>
      <h3><a href="{{ item.url | relative_url }}">{{ item.title }}</a></h3>
      <p>{{ item.summary }}</p>
    </article>
  {% endfor %}
</div>

## FAQ

<div class="faq-list">
  <details class="faq-item" open>
    <summary>Where should I start if I am new to Iket?</summary>
    <p>Start with the installation guide, then move into the CLI command reference so you can verify access, inspect the gateway, and understand the safe change workflow.</p>
  </details>
  <details class="faq-item">
    <summary>Do I need the plugin system right away?</summary>
    <p>No. Most teams should start with the built-in gateway, policy, and rollout features, then add plugins only when they have a clear extension need.</p>
  </details>
  <details class="faq-item">
    <summary>Is Iket only for internal APIs?</summary>
    <p>No. Iket is designed for classic APIs, internal service traffic, and newer agent or AI-facing routes, with the same operational model across them.</p>
  </details>
  <details class="faq-item">
    <summary>How should I approach production changes?</summary>
    <p>Use diffs, proposals, approvals, canaries, and revisions as the normal path rather than pushing configuration changes blindly.</p>
  </details>
</div>

<div class="docs-hub">
  <div class="docs-hub__intro">
    <p>Start with setup and operations if you are bringing up a new gateway, then move into management, plugins, and deeper technical notes as your deployment grows.</p>
  </div>

  <div class="docs-hub__grid">
    {% assign nav_sections = docs_pages | group_by: "section" %}
    {% for section in nav_sections %}
      <section class="docs-hub__section" data-docs-section>
        <h2>{{ section.name }}</h2>
        {% assign section_items = section.items | sort: "weight" %}
        {% assign lead_doc = section_items | first %}
        {% if lead_doc.summary %}
          <p class="docs-hub__summary">{{ lead_doc.summary }}</p>
        {% endif %}
        <div class="docs-grid">
          {% for item in section_items %}
            <article class="doc-card docs-hub__card" data-doc-card data-doc-audience="{{ item.audience | join: '|' }}" data-doc-topics="{{ item.topics | join: '|' }}">
              <h3><a href="{{ item.url }}">{{ item.title }}</a></h3>
              {% if item.summary %}
                <p class="docs-hub__item-summary">{{ item.summary }}</p>
              {% endif %}
              {% if item.audience or item.topics %}
                <div class="docs-meta" aria-label="Document metadata">
                  {% for label in item.audience %}
                    <span class="docs-meta__chip docs-meta__chip--audience">{{ label }}</span>
                  {% endfor %}
                  {% for label in item.topics %}
                    <span class="docs-meta__chip">{{ label }}</span>
                  {% endfor %}
                </div>
              {% endif %}
            </article>
          {% endfor %}
        </div>
      </section>
    {% endfor %}
  </div>
</div>

## Repository

<div class="link-grid">
  <div class="panel">
    <h3><a href="https://github.com/bhangun/iket">GitHub project</a></h3>
    <p>Browse the source, issues, and commit history.</p>
  </div>
  <div class="panel">
    <h3><a href="https://github.com/bhangun/iket/releases">Releases</a></h3>
    <p>Download published versions and track release notes.</p>
  </div>
</div>

<script>
  (function () {
    var filtersRoot = document.querySelector("[data-docs-filters]");
    if (!filtersRoot) {
      return;
    }

    var cards = Array.prototype.slice.call(document.querySelectorAll("[data-doc-card]"));
    var sections = Array.prototype.slice.call(document.querySelectorAll("[data-docs-section]"));
    var status = filtersRoot.querySelector("[data-filter-status]");
    var searchRoot = document.querySelector("[data-docs-search-root]");
    var searchInput = searchRoot ? searchRoot.querySelector("[data-docs-search-input]") : null;
    var searchClear = searchRoot ? searchRoot.querySelector("[data-docs-search-clear]") : null;
    var searchStatus = searchRoot ? searchRoot.querySelector("[data-docs-search-status]") : null;
    var copyButton = filtersRoot.querySelector("[data-copy-filter-link]");
    var clearButton = filtersRoot.querySelector("[data-clear-filters]");
    var active = { audience: "", topic: "", search: "" };

    function tokensFor(card, attr) {
      var raw = card.getAttribute(attr) || "";
      return raw ? raw.split("|") : [];
    }

    function updateButtons(kind, value) {
      var buttons = filtersRoot.querySelectorAll('[data-filter-kind="' + kind + '"]');
      Array.prototype.forEach.call(buttons, function (button) {
        button.classList.toggle("is-active", button.getAttribute("data-filter-value") === value);
      });
    }

    function syncUrl() {
      var url = new URL(window.location.href);

      if (active.audience) {
        url.searchParams.set("audience", active.audience);
      } else {
        url.searchParams.delete("audience");
      }

      if (active.topic) {
        url.searchParams.set("topic", active.topic);
      } else {
        url.searchParams.delete("topic");
      }

      var next = url.pathname + (url.search ? url.search : "") + (url.hash ? url.hash : "");
      window.history.replaceState({}, "", next);
    }

    function currentShareUrl() {
      return window.location.href;
    }

    function setCopyButtonLabel(label) {
      if (copyButton) {
        copyButton.textContent = label;
      }
    }

    function updateActionState() {
      if (clearButton) {
        clearButton.disabled = !active.audience && !active.topic && !active.search;
      }
    }

    function searchTextFor(card) {
      return ((card.getAttribute("data-doc-search-text") || "") + " " + (card.textContent || ""))
        .toLowerCase()
        .replace(/\s+/g, " ")
        .trim();
    }

    function fallbackCopy(text) {
      var input = document.createElement("input");
      input.type = "text";
      input.value = text;
      input.setAttribute("readonly", "");
      input.style.position = "absolute";
      input.style.left = "-9999px";
      document.body.appendChild(input);
      input.select();
      input.setSelectionRange(0, input.value.length);
      document.execCommand("copy");
      document.body.removeChild(input);
    }

    function copyFilteredLink() {
      var text = currentShareUrl();

      if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(text).then(function () {
          setCopyButtonLabel("Copied link");
          window.setTimeout(function () {
            setCopyButtonLabel("Copy filtered link");
          }, 1600);
        }).catch(function () {
          fallbackCopy(text);
          setCopyButtonLabel("Copied link");
          window.setTimeout(function () {
            setCopyButtonLabel("Copy filtered link");
          }, 1600);
        });
        return;
      }

      fallbackCopy(text);
      setCopyButtonLabel("Copied link");
      window.setTimeout(function () {
        setCopyButtonLabel("Copy filtered link");
      }, 1600);
    }

    function applyFilters() {
      var visibleCount = 0;

      cards.forEach(function (card) {
        var matchesAudience = !active.audience || tokensFor(card, "data-doc-audience").indexOf(active.audience) >= 0;
        var matchesTopic = !active.topic || tokensFor(card, "data-doc-topics").indexOf(active.topic) >= 0;
        var matchesSearch = !active.search || searchTextFor(card).indexOf(active.search) >= 0;
        var visible = matchesAudience && matchesTopic && matchesSearch;
        card.hidden = !visible;
        if (visible) {
          visibleCount += 1;
        }
      });

      sections.forEach(function (section) {
        var hasVisibleCards = section.querySelector("[data-doc-card]:not([hidden])");
        section.hidden = !hasVisibleCards;
      });

      if (!status) {
        return;
      }

      if (!active.audience && !active.topic && !active.search) {
        status.textContent = "Showing all docs.";
        if (searchStatus) {
          searchStatus.textContent = "Search across all documentation cards.";
        }
        updateActionState();
        return;
      }

      var parts = [];
      if (active.audience) {
        parts.push("audience: " + active.audience);
      }
      if (active.topic) {
        parts.push("topic: " + active.topic);
      }
      if (active.search) {
        parts.push("search: " + active.search);
      }
      status.textContent = "Showing " + visibleCount + " doc" + (visibleCount === 1 ? "" : "s") + " for " + parts.join(", ") + ".";
      if (searchStatus) {
        searchStatus.textContent = active.search
          ? "Matching " + visibleCount + " result" + (visibleCount === 1 ? "" : "s") + " for \"" + active.search + "\"."
          : "Search across all documentation cards.";
      }
      updateActionState();
    }

    function clearFilters() {
      active.audience = "";
      active.topic = "";
      active.search = "";
      updateButtons("audience", "");
      updateButtons("topic", "");
      if (searchInput) {
        searchInput.value = "";
      }
      if (searchClear) {
        searchClear.hidden = true;
      }
      setCopyButtonLabel("Copy filtered link");
      syncUrl();
      applyFilters();
    }

    function applyStateFromUrl() {
      var url = new URL(window.location.href);
      var audience = url.searchParams.get("audience") || "";
      var topic = url.searchParams.get("topic") || "";
      var search = url.searchParams.get("search") || "";

      if (audience && filtersRoot.querySelector('[data-filter-kind="audience"][data-filter-value="' + audience + '"]')) {
        active.audience = audience;
      }
      if (topic && filtersRoot.querySelector('[data-filter-kind="topic"][data-filter-value="' + topic + '"]')) {
        active.topic = topic;
      }
      if (search) {
        active.search = search.toLowerCase();
      }

      updateButtons("audience", active.audience);
      updateButtons("topic", active.topic);
      if (searchInput) {
        searchInput.value = active.search;
      }
      if (searchClear) {
        searchClear.hidden = !active.search;
      }
    }

    filtersRoot.addEventListener("click", function (event) {
      if (event.target.closest("[data-clear-filters]")) {
        clearFilters();
        return;
      }

      if (event.target.closest("[data-copy-filter-link]")) {
        copyFilteredLink();
        return;
      }

      var button = event.target.closest("[data-filter-kind]");
      if (!button) {
        return;
      }

      var kind = button.getAttribute("data-filter-kind");
      active[kind] = button.getAttribute("data-filter-value") || "";
      updateButtons(kind, active[kind]);
      syncUrl();
      applyFilters();
    });

    if (searchInput) {
      searchInput.addEventListener("input", function () {
        active.search = (searchInput.value || "").toLowerCase().trim();
        if (searchClear) {
          searchClear.hidden = !active.search;
        }

        var url = new URL(window.location.href);
        if (active.search) {
          url.searchParams.set("search", active.search);
        } else {
          url.searchParams.delete("search");
        }
        window.history.replaceState({}, "", url.pathname + (url.search ? url.search : "") + (url.hash ? url.hash : ""));

        applyFilters();
      });
    }

    if (searchClear) {
      searchClear.addEventListener("click", function () {
        if (!searchInput) {
          return;
        }
        searchInput.value = "";
        active.search = "";
        searchClear.hidden = true;
        setCopyButtonLabel("Copy filtered link");

        var url = new URL(window.location.href);
        url.searchParams.delete("search");
        window.history.replaceState({}, "", url.pathname + (url.search ? url.search : "") + (url.hash ? url.hash : ""));

        applyFilters();
        searchInput.focus();
      });
    }

    applyStateFromUrl();
    applyFilters();
    updateActionState();
  })();
</script>
