---
title: Understanding Browser Sniffing with uakit.js
excerpt: A deep dive into User-Agent detection, why it matters, and how uakit.js provides a clean, type-safe API for browser sniffing
date: 2024-01-26
---

<section>
  <div class="row">
    <div class="col-4 col-md-8 col-lg-12">
      <article class="brixs-article">
        <header>
          <h1>Understanding Browser Sniffing with uakit.js</h1>
          <p>2024-01-26 • Deep dive • 8 min read</p>
        </header>

        <section>
          <h2>Introduction</h2>
          <p>
            Every browser sends identity hints to websites. Historically the biggest one was the
            <strong>User-Agent (UA) string</strong>. Even though the web has moved toward feature
            detection and standards, UA-related work still shows up in real products: analytics and debugging
            ("why does this break only on Samsung Internet?"), temporary mitigations ("disable this animation on iOS 15"),
            and progressive migrations ("ship the new flow to modern Chromium only"). This article explains what UA is,
            what "sniffing" means, when it's justified, and why
            <strong>uakit.js</strong> is intentionally built as a small set of narrowly-scoped queries.
          </p>

          <blockquote brixsBlockquote="info">
            <span brixsTag>TL;DR:</span> Prefer feature detection whenever possible. If you must sniff, keep it centralized,
            testable, and conservative. uakit.js exposes a tiny surface area
            (<code class="brixs-code">sniff.browser</code>, <code class="brixs-code">sniff.os</code>, etc.) so the rest of your app
            doesn’t scatter regexes across the codebase.
          </blockquote>
        </section>

        <section>
          <h2>What is the User-Agent String?</h2>
          <p>
            The <strong>User-Agent</strong> is a request header (and often accessible in client JS via
            <code class="brixs-code">navigator.userAgent</code>) that contains information about: browser family
            (Chrome/Edge/Firefox/Safari…), rendering engine hints (historically WebKit/Blink/Gecko), OS hints
            (Windows/macOS/iOS/Android), and device hints (mobile/tablet). In practice, it’s a messy compatibility artifact.
          </p>

          <h3>Why it’s messy</h3>
          <p>
            UA strings are not a clean API. Browsers include tokens for compatibility ("Safari" appears in many Chromium UAs),
            vendors change formatting over time, and privacy initiatives are actively reducing fingerprinting.
          </p>

          <blockquote brixsBlockquote="warning">
            <span brixsTag="warning">Important:</span> UA is not a stable contract. Treat UA inspection as a best-effort heuristic,
            not a guarantee.
          </blockquote>
        </section>

        <section>
          <h2>What is Browser Sniffing?</h2>
          <p>
            <strong>Browser sniffing</strong> (or UA sniffing) means inferring environment details by
            inspecting identity hints (UA string, platform hints, vendor tokens), rather than using a capability-based check.
            Common sniffing outcomes include: browser family (Chrome vs Safari), browser major version (16 vs 17),
            mobile vs desktop, and OS family and version.
          </p>

          <h3>Feature detection vs sniffing</h3>
          <p>Feature detection checks if the platform can do something:</p>

          <pre brixsPre><code class="language-typescript">// Feature detection (preferred)

const hasPopover =
typeof HTMLElement !== "undefined" && "popover" in HTMLElement.prototype;
</code></pre>

          <p>Sniffing checks "what are you":</p>

          <pre brixsPre><code class="language-typescript">// Browser sniffing (use sparingly)

const ua = navigator.userAgent;
const isSafari = /Safari/.test(ua) && !/Chrome|Chromium/.test(ua);
</code></pre>

          <blockquote brixsBlockquote="success">
            <span brixsTag>Best Practice:</span> Prefer capability checks. If you’re using sniffing to enable a feature, there’s
            often a better capability check. Use sniffing mainly for
            <em>workarounds</em> and <em>analytics</em>, not as a primary
            feature gate.
          </blockquote>
        </section>

        <section>
          <h2>Why is Browser Sniffing Important?</h2>
          <p>
            Despite its flaws, UA sniffing is still used because it can answer questions that feature detection sometimes
            can’t answer cheaply: <strong>known-bad combinations</strong>
            ("Android WebView &lt; X has a scrolling bug."), <strong>version gating</strong>
            ("This feature is safe only after Safari 16.4."), <strong>support triage</strong>
            to reproduce issues reported by users without full telemetry, and
            <strong>analytics and debugging</strong> to understand which browsers/devices are
            causing issues.
          </p>

          <blockquote brixsBlockquote="info">
            <span brixsTag="primary">When UA sniffing is justified:</span>
            <ol>
              <li>You’re working around a documented, version-specific browser bug.</li>
              <li>The workaround must ship quickly and be removed later.</li>
              <li>You need a safe default behavior (fall back to the simplest UX).</li>
              <li>You keep sniffing logic in one place and write tests for it.</li>
            </ol>
          </blockquote>
        </section>

        <section>
          <h2>Introducing uakit.js</h2>
          <p>
            <strong>uakit.js</strong> is a lightweight, type-safe library designed around one simple goal:
            centralize UA heuristics behind a tiny, typed API that prevents regex sprawl across your codebase.
          </p>

          <p>From the public API, you import a single object:</p>

          <pre brixsPre><code class="language-typescript">import { sniff } from "uakit.js";

sniff.browser; // "Chrome" | "Edge" | "Firefox" | "Safari" | ...
sniff.browserVersion; // "91" | "92" | ... (stringified major)
sniff.mobile; // boolean
sniff.os; // "Android" | "IOS" | "MacOS" | "Windows" | ...
sniff.osVersion; // e.g. "14.4" / "10.15.7"
</code></pre>

          <h3>Key features</h3>
          <ol>
            <li><strong>Type-safe</strong>: TypeScript definitions for all browser/OS values</li>
            <li><strong>Minimal API</strong>: Just a few getters, no configuration needed</li>
            <li><strong>Well-tested</strong>: Comprehensive test suite with mocked navigator objects</li>
            <li><strong>Lightweight</strong>: Small bundle size, no dependencies</li>
          </ol>
        </section>

        <section>
          <h2>How to Use uakit.js</h2>

          <h3>Installation</h3>
          <pre brixsPre><code class="language-bash">npm install uakit.js

</code></pre>

          <h3>Basic usage</h3>
          <pre brixsPre><code class="language-typescript">import { sniff } from "uakit.js";

// Check browser
if (sniff.browser === "Safari") {
console.log("Running on Safari");
}

// Check mobile
if (sniff.mobile) {
console.log("Running on mobile device");
}

// Check OS
if (sniff.os === "IOS") {
console.log("Running on iOS");
}

// Check version
if (sniff.browser === "Chrome" && parseInt(sniff.browserVersion) >= 90) {
console.log("Chrome 90 or higher");
}
</code></pre>

          <h3>Real-world examples</h3>

          <p><strong>Example 1: Working around a Safari bug</strong></p>
          <pre brixsPre><code class="language-typescript">// Disable smooth scrolling on older Safari versions

const enableSmoothScroll = !(
sniff.browser === "Safari" &&
parseInt(sniff.browserVersion) < 16
);

if (enableSmoothScroll) {
element.scrollIntoView({ behavior: "smooth" });
} else {
element.scrollIntoView();
}
</code></pre>

          <p><strong>Example 2: Mobile-specific behavior</strong></p>
          <pre brixsPre><code class="language-typescript">// Show different UI on mobile

const showMobileMenu = sniff.mobile;

if (showMobileMenu) {
renderMobileNavigation();
} else {
renderDesktopNavigation();
}
</code></pre>

          <p><strong>Example 3: Analytics tracking</strong></p>
          <pre brixsPre><code class="language-typescript">// Send browser/OS info to analytics

analytics.track("page_view", {
browser: sniff.browser,
browserVersion: sniff.browserVersion,
os: sniff.os,
mobile: sniff.mobile,
});
</code></pre>

          <blockquote brixsBlockquote="info">
            <span brixsTag>Note:</span> Property getters (not function calls). In the current implementation,
            <code class="brixs-code">sniff</code> exposes <strong>properties</strong> via getters
            (e.g. <code class="brixs-code">sniff.browser</code>). Some public docs/examples show
            <code class="brixs-code">sniff.browser()</code> as if it were a method; treat that as documentation drift.
          </blockquote>

          <h3>Why getters?</h3>
          <p>
            In uakit.js, the exported <code class="brixs-code">sniff</code> object uses getters that forward to small functions.
            That choice keeps the API pleasant: <code class="brixs-code">sniff.browser</code> reads like a property, not a function
            call; internally, you can still swap implementations (UA parsing vs Client Hints) without changing call sites;
            and it provides a cleaner, more intuitive developer experience.
          </p>
        </section>

        <section>
          <h2>Architecture & Internal Structure</h2>
          <p>
            At a high level, uakit.js is split into: <strong>a tiny entrypoint</strong>
            (<code class="brixs-code">src/index.ts</code>) that exports <code class="brixs-code">sniff</code> as a stable API surface;
            <strong>feature modules</strong> (<code class="brixs-code">src/features/*</code>) that implement detection:
            browser, OS, mobile, and version parsing; <strong class="brixs-font-strong -medium">utilities + types</strong>
            (<code class="brixs-code">src/utils/*</code>) that define the normalized unions (<code class="brixs-code">Browsers</code>,
            <code class="brixs-code">OS</code>, <code class="brixs-code">Sniff</code>) and helpers (like regex matching / enum helpers);
            and <strong>tests</strong> that validate behavior across many UA variants using mocked
            <code class="brixs-code">navigator</code> objects.
          </p>

          <h3>Flow diagram</h3>
          <p>Here’s how the code flows from consumer to navigator APIs:</p>

          <pre brixsPre><code class="language-mermaid">flowchart TD

A[Consumer code] --> B[sniff.* getters]
B --> I[src/index.ts]

I -->|exports| FEAT[src/features/index.ts]
I -->|exports| UTIL[src/utils/index.ts]

FEAT --> BR[src/features/browser]
FEAT --> MO[src/features/mobile]
FEAT --> OS[src/features/os]
FEAT --> VE[src/features/version]

BR --> UA[navigator.userAgent]
OS --> UA
VE --> UA
MO --> UAD[navigator.userAgentData?.mobile]
MO --> MX[navigator.maxTouchPoints]
OS --> PL[navigator.platform]

VE --> MBI[utils.matchByIndex]
UTIL --> TYPES[utils/typing: Browsers, OS, Sniff]
UTIL --> FUNCS[utils/functions: matchByIndex, toEnum]
UTIL --> TEST[utils/test mocks]

TEST --> SPEC[src/spec/sniff.spec.ts]
</code></pre>

          <h3>Why this architecture?</h3>
          <ol>
            <li>It prevents "regex sprawl" across product code.</li>
            <li>Each detector is small and independently testable.</li>
            <li>The public surface area stays stable even if parsing changes.</li>
            <li>It encourages conservative outputs (known set of values), instead of leaking raw UA strings everywhere.</li>
          </ol>

          <div brixsHighlight>
              <div highlightHeading>Design constraint</div>
              <div highlightContent>
                Keep the runtime contract minimal and predictable: the rest of the app should never need to know
                <em>how</em> detection works.
              </div>
          </div>
        </section>

        <section>
          <h2>Best Practices</h2>
          <ol>
            <li><strong>Centralize sniffing logic</strong>: Keep all UA checks in one module, not scattered throughout your app.</li>
            <li><strong>Write tests</strong>: Mock navigator objects to test different UA scenarios.</li>
            <li><strong>Document workarounds</strong>: Add comments explaining why you’re sniffing and link to bug reports.</li>
            <li><strong>Plan for removal</strong>: Browser bugs get fixed. Set reminders to remove temporary workarounds.</li>
            <li><strong>Prefer feature detection</strong>: Only sniff when feature detection isn’t practical.</li>
          </ol>

          <blockquote brixsBlockquote="warning">
            <span brixsTag="warning">Remember:</span> Browser sniffing is a tool of last resort. Always prefer standards-based feature detection when possible.
            Use sniffing for analytics, debugging, and temporary workarounds—not as a primary feature gate.
          </blockquote>
        </section>

        <footer>
          <h2>Conclusion</h2>
          <p>
            Browser sniffing remains a necessary tool in modern web development, despite the push towards feature detection.
            <strong>uakit.js</strong> provides a clean, type-safe way to handle UA detection without
            polluting your codebase with regex patterns. By centralizing detection logic, providing strong TypeScript types,
            and maintaining a minimal API surface, uakit.js makes browser sniffing more maintainable and less error-prone.
          </p>
          <p>
            Check out the
            <a href="https://github.com/maxwyn-o/uakit" target="_blank" rel="noopener noreferrer">
              uakit.js repository on GitHub
            </a>
            to get started.
          </p>
        </footer>
      </article>
    </div>

  </div>
</section>
