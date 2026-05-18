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

## What You’ll Find Here

- installation and upgrade guidance
- CLI and management API workflows
- production and operations playbooks
- plugin development and extension guides
- reference material for gateway behavior and features

<div class="docs-hub">
  <div class="docs-hub__intro">
    <p>Start with installation and deployment if you are bringing up a new gateway, then move into management, plugins, and reference material as your setup grows.</p>
  </div>

  {% assign docs_pages = site.pages | where: "layout", "docs_page" | sort: "weight" | sort: "section_order" %}
  {% capture audience_values %}{% for item in docs_pages %}{% for label in item.audience %}{{ label }}|{% endfor %}{% endfor %}{% endcapture %}
  {% capture topic_values %}{% for item in docs_pages %}{% for label in item.topics %}{{ label }}|{% endfor %}{% endfor %}{% endcapture %}
  {% assign audiences = audience_values | split: "|" | uniq | sort %}
  {% assign topics = topic_values | split: "|" | uniq | sort %}

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

    <p class="docs-filters__status" data-filter-status>Showing all docs.</p>
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
        <ul>
          {% for item in section_items %}
            <li class="docs-hub__item" data-doc-audience="{{ item.audience | join: '|' }}" data-doc-topics="{{ item.topics | join: '|' }}">
              <a href="{{ item.url }}">{{ item.title }}</a>
              {% if item.summary %}
                <p class="docs-hub__item-summary">{{ item.summary }}</p>
              {% endif %}
              {% if item.audience or item.topics %}
                <div class="docs-meta docs-meta--compact" aria-label="Document metadata">
                  {% for label in item.audience %}
                    <span class="docs-meta__chip docs-meta__chip--audience">{{ label }}</span>
                  {% endfor %}
                  {% for label in item.topics %}
                    <span class="docs-meta__chip">{{ label }}</span>
                  {% endfor %}
                </div>
              {% endif %}
            </li>
          {% endfor %}
        </ul>
      </section>
    {% endfor %}
  </div>
</div>

<script>
  (function () {
    var filtersRoot = document.querySelector("[data-docs-filters]");
    if (!filtersRoot) {
      return;
    }

    var items = Array.prototype.slice.call(document.querySelectorAll(".docs-hub__item"));
    var sections = Array.prototype.slice.call(document.querySelectorAll("[data-docs-section]"));
    var status = filtersRoot.querySelector("[data-filter-status]");
    var active = { audience: "", topic: "" };

    function tokensFor(item, attr) {
      var raw = item.getAttribute(attr) || "";
      return raw ? raw.split("|") : [];
    }

    function updateButtons(kind, value) {
      var buttons = filtersRoot.querySelectorAll('[data-filter-kind="' + kind + '"]');
      Array.prototype.forEach.call(buttons, function (button) {
        button.classList.toggle("is-active", button.getAttribute("data-filter-value") === value);
      });
    }

    function applyFilters() {
      var visibleCount = 0;

      items.forEach(function (item) {
        var matchesAudience = !active.audience || tokensFor(item, "data-doc-audience").indexOf(active.audience) >= 0;
        var matchesTopic = !active.topic || tokensFor(item, "data-doc-topics").indexOf(active.topic) >= 0;
        var visible = matchesAudience && matchesTopic;
        item.hidden = !visible;
        if (visible) {
          visibleCount += 1;
        }
      });

      sections.forEach(function (section) {
        var hasVisibleItems = section.querySelector(".docs-hub__item:not([hidden])");
        section.hidden = !hasVisibleItems;
      });

      if (!status) {
        return;
      }

      if (!active.audience && !active.topic) {
        status.textContent = "Showing all docs.";
        return;
      }

      var parts = [];
      if (active.audience) {
        parts.push("audience: " + active.audience);
      }
      if (active.topic) {
        parts.push("topic: " + active.topic);
      }
      status.textContent = "Showing " + visibleCount + " doc" + (visibleCount === 1 ? "" : "s") + " for " + parts.join(", ") + ".";
    }

    filtersRoot.addEventListener("click", function (event) {
      var button = event.target.closest("[data-filter-kind]");
      if (!button) {
        return;
      }

      var kind = button.getAttribute("data-filter-kind");
      active[kind] = button.getAttribute("data-filter-value") || "";
      updateButtons(kind, active[kind]);
      applyFilters();
    });

    applyFilters();
  })();
</script>

## Repository

- [GitHub project](https://github.com/bhangun/iket)
- [Releases](https://github.com/bhangun/iket/releases)
