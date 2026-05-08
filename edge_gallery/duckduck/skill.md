-----

## name: web-search
description: >
Searches the web using DuckDuckGo and returns relevant results.
Use this skill when the user asks to search online, look up current
information, find facts, news, definitions, or any topic on the web.

## What this skill does

Queries DuckDuckGo’s Instant Answer API (no API key required) and returns
a structured summary including direct answers, definitions, abstracts,
and related results.

## Instructions for the model

1. Extract the search query from the user’s message.
1. Call this skill with the extracted query as input.
1. Present the results clearly, summarizing the most relevant information.
1. If results are limited, suggest the user refine their search.

## JavaScript

```js
window.ai_edge_gallery_get_result = async function(query) {
  try {
    const endpoint =
      "https://api.duckduckgo.com/?format=json&no_html=1&skip_disambig=1&q=" +
      encodeURIComponent(query);

    const res = await fetch(endpoint);
    if (!res.ok) throw new Error("HTTP " + res.status);
    const data = await res.json();

    const parts = [];

    // Respuesta directa (ej: conversiones, calculos)
    if (data.Answer) {
      parts.push("DIRECT ANSWER:\n" + data.Answer);
    }

    // Definicion
    if (data.Definition) {
      parts.push(
        "DEFINITION:\n" + data.Definition +
        (data.DefinitionURL ? "\nSource: " + data.DefinitionURL : "")
      );
    }

    // Abstract principal
    if (data.AbstractText) {
      parts.push(
        "SUMMARY:\n" + data.AbstractText +
        (data.AbstractURL ? "\nSource: " + data.AbstractURL : "") +
        (data.AbstractSource ? " (" + data.AbstractSource + ")" : "")
      );
    }

    // Resultados relacionados (top 5)
    const topics = (data.RelatedTopics || [])
      .filter(t => t.Text && t.FirstURL)
      .slice(0, 5);

    if (topics.length > 0) {
      const topicLines = topics.map(
        (t, i) => (i + 1) + ". " + t.Text + "\n   " + t.FirstURL
      );
      parts.push("RELATED RESULTS:\n" + topicLines.join("\n\n"));
    }

    if (parts.length === 0) {
      return (
        'No structured results found for "' + query + '".\n' +
        "DuckDuckGo Instant Answers works best for: facts, definitions, " +
        "conversions, known entities. For broader web results, try a " +
        "more specific query."
      );
    }

    return (
      'Search results for: "' + query + '"\n' +
      "─".repeat(40) + "\n\n" +
      parts.join("\n\n" + "─".repeat(40) + "\n\n")
    );

  } catch (err) {
    return "Search failed: " + err.message + 
           "\nCheck your internet connection and try again.";
  }
};
```
