<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>AccessiBe test page</title>
  <style>
    :root{font-family:system-ui,-apple-system,Segoe UI,Roboto,'Helvetica Neue',Arial;line-height:1.4}
    body{margin:0;padding:1rem;background:#fff;color:#111}
    header,main,footer{max-width:1000px;margin:0 auto}
    header{display:flex;gap:1rem;align-items:center}
    nav a{margin-right:.5rem}
    .status{border:2px dashed #666;padding:1rem;border-radius:8px;margin:1rem 0}
    .visually-hidden{position:absolute !important;height:1px;width:1px;overflow:hidden;clip:rect(1px,1px,1px,1px);white-space:nowrap}
    .demo-grid{display:grid;grid-template-columns:1fr 1fr;gap:1rem}
    .modal{background:#fff;border:1px solid #888;padding:1rem;border-radius:8px;box-shadow:0 8px 24px rgba(0,0,0,.12)}
    footer{margin-top:2rem;padding-top:1rem;border-top:1px solid #eee}
    button{padding:.5rem 1rem;font-size:1rem}
  </style>
   <script>(function () {
  // Defensive global config - set BEFORE app.js executes if possible
  try {
    var targets = [
      'accessWidgetSpecificsOptions',
      'accessWidgetOptions',
      'accessWidgetSpecifics',
      'accessiBeOptions',
      'acsbOptions'
    ];
    targets.forEach(function (k) {
      window[k] = window[k] || {};
      window[k].hideComponents = window[k].hideComponents || [];
      if (!window[k].hideComponents.includes('colorAdjustments')) {
        window[k].hideComponents.push('colorAdjustments');
      }
    });
  } catch (e) {
    console.warn('[site] Failed to set AccessiBe config', e);
  }

  function hideColorControls(root) {
    root = root || document;
    var selectors = [
      '.acsb-color',
      '.acsb-color-adjustments',
      '[data-acsb-component="colorAdjustments"]',
      '[data-acsb="colorAdjustments"]',
      '.acsb-adjustment-color',
      '.acsb-adjustments-color'
    ];

    var found = 0;
    try {
      selectors.forEach(function (sel) {
        Array.from(root.querySelectorAll(sel)).forEach(function (el) {
          try {
            el.style.setProperty('display', 'none', 'important');
            el.setAttribute('data-acsb-hidden', 'true');
            found++;
          } catch (e) {}
        });
      });

      // Case-insensitive keyword match on node textContent / aria / title / dataset
      var keywords = [
        'color',
        'colour',
        'contrast',
        'color adjust',
        'color-adjust',
        'color adjustments',
        'color adjustment',
        'high contrast',
        'background color',
        'text color'
      ];

      // Scan candidate nodes to avoid scanning the entire document if possible.
      var candidates = root.querySelectorAll('.acsb-widget, .acsb-body, .acsb-settings, .acsb-section, .acsb-item, .acsb-menu, [data-acsb], button, a, [role="button"], [title], [aria-label], [data-name]');
      Array.from(candidates).forEach(function (node) {
        try {
          var combined = (
            (node.textContent || '') + ' ' +
            (node.getAttribute && (node.getAttribute('aria-label') || '')) + ' ' +
            (node.getAttribute && (node.getAttribute('title') || '')) + ' ' +
            JSON.stringify(node.dataset || {})
          ).toLowerCase();
          for (var i = 0; i < keywords.length; i++) {
            if (combined.indexOf(keywords[i]) !== -1) {
              node.style.setProperty('display', 'none', 'important');
              node.setAttribute('data-acsb-hidden', 'true');
              found++;
              break;
            }
          }
        } catch (e) {}
      });
    } catch (e) {
      console.warn('[site] hideColorControls error', e);
    }
    if (found) console.info('[site] hideColorControls hid elements:', found);
    return found;
  }

  function checkIframes() {
    try {
      Array.from(document.querySelectorAll('iframe')).forEach(function (ifr) {
        try {
          var combined = ((ifr.src || '') + ' ' + (ifr.title || '') + ' ' + (ifr.name || '')).toLowerCase();
          if (/acsb|accessibe|accessi/i.test(combined)) {
            // cross-origin iframe likely — hide the iframe itself as a fallback
            ifr.style.setProperty('display', 'none', 'important');
            ifr.setAttribute('data-acsb-hidden', 'true');
            console.info('[site] Hid AccessiBe iframe:', ifr.src || ifr.title || ifr.name);
          }
        } catch (e) {}
      });
    } catch (e) {
      console.warn('[site] checkIframes error', e);
    }
  }

  // MutationObserver to catch dynamically inserted widget nodes
  var observer;
  try {
    observer = new MutationObserver(function () {
      hideColorControls(document);
      checkIframes();
    });
    observer.observe(document.documentElement || document.body, { childList: true, subtree: true });
  } catch (e) {
    console.warn('[site] Failed to create MutationObserver', e);
  }

  // Initial attempts + retries for late load
  hideColorControls(document);
  checkIframes();
  setTimeout(function () { hideColorControls(document); checkIframes(); }, 800);
  setTimeout(function () { hideColorControls(document); checkIframes(); }, 3000);
  setTimeout(function () { hideColorControls(document); checkIframes(); }, 7000);

  // Stop observing after 30s to avoid leaks
  setTimeout(function () { try { observer && observer.disconnect(); } catch (e) {} }, 30000);

  // If you want to load AccessiBe's app.js from here, only load it once.
  if (!window.acsbJS && !document.querySelector('script[src*="acsbapp.com"]')) {
    var s = document.createElement('script');
    s.src = 'https://acsbapp.com/apps/app/dist/js/app.js';
    s.async = true;
    s.onload = function () {
      try {
        if (window.acsbJS && typeof acsbJS.init === 'function') {
          acsbJS.init();
          // re-run hide after widget initializes
          setTimeout(function () { hideColorControls(document); checkIframes(); }, 200);
          setTimeout(function () { hideColorControls(document); checkIframes(); }, 1000);
        }
      } catch (e) {
        console.warn('[site] acsb init/onload error', e);
      }
    };
    (document.head || document.body).appendChild(s);
  } else {
    // If app.js is loaded elsewhere, re-scan after acsbReady if fired
    document.addEventListener('acsbReady', function () {
      setTimeout(function () { hideColorControls(document); checkIframes(); }, 50);
      setTimeout(function () { hideColorControls(document); checkIframes(); }, 400);
    });
  }
})();</script> 
</head>
<body>
  <a class="visually-hidden" href="#main">Skip to main content</a>

  <header role="banner">
    <h1 id="logo">AccessiBe Test Page</h1>
    <nav role="navigation" aria-label="Primary">
      <a href="#features">Features</a>
      <a href="#form">Form</a>
      <a href="#gallery">Gallery</a>
    </nav>
  </header>

  <section class="status" aria-live="polite">
    <h2>AccessiBe script status</h2>
    <p id="acsb-status">Not detected</p>
    <p>
      Paste your AccessiBe script tag into the <code>&lt;!-- ACCESSIBE SCRIPT HERE --&gt;</code> placeholder in the HTML file, save, then reload this page. The test below will also try to detect the script automatically.
    </p>
    <p>
      <button id="manual-scan">Run manual scan now</button>
      <button id="show-scripts">List script tags</button>
    </p>
    <pre id="script-list" style="white-space:pre-wrap;display:none;border-top:1px solid #eee;padding-top:.5rem;margin-top:.5rem"></pre>
  </section>

  <main id="main" role="main">
    <section id="features">
      <h2>Interactive controls (keyboard test)</h2>
      <p>Try using only the keyboard (Tab / Shift+Tab / Enter / Space) to interact with these controls.</p>
      <div class="demo-grid">
        <div>
          <label for="name">Name</label>
          <input id="name" type="text" placeholder="Type your name" />
        </div>
        <div>
          <label for="role">Role</label>
          <select id="role">
            <option>Choose…</option>
            <option>Developer</option>
            <option>Designer</option>
            <option>Tester</option>
          </select>
        </div>
      </div>

      <h3>Modal example</h3>
      <button id="open-modal">Open modal</button>
      <div id="demo-modal" class="modal" role="dialog" aria-modal="true" aria-labelledby="modal-title" hidden>
        <h4 id="modal-title">Sample modal</h4>
        <p>This is a focus-trapped dialog — press Escape to close.</p>
        <button id="close-modal">Close</button>
      </div>

    </section>

    <section id="form">
      <h2>Simple form (aria-validation)</h2>
      <form id="contact-form">
        <div>
          <label for="email">Email</label>
          <input id="email" name="email" type="email" aria-describedby="email-hint" required />
          <div id="email-hint">We'll only use this for test messages.</div>
        </div>
        <div>
          <label for="message">Message</label>
          <textarea id="message" name="message" rows="4"></textarea>
        </div>
        <button type="submit">Submit</button>
      </form>
      <div role="status" aria-live="polite" id="form-result" style="margin-top:.5rem"></div>
    </section>

    <section id="gallery">
      <h2>Images & table</h2>
      <img src="https://via.placeholder.com/320x180" alt="Placeholder example image showing a scenic landscape" width="320" height="180" />
      <table aria-label="Sample data table" style="width:100%;margin-top:1rem;border-collapse:collapse">
        <thead>
          <tr><th style="text-align:left">Name</th><th style="text-align:left">Role</th></tr>
        </thead>
        <tbody>
          <tr><td>Alice</td><td>Developer</td></tr>
          <tr><td>Bob</td><td>Designer</td></tr>
        </tbody>
      </table>
    </section>

    <section aria-hidden="true" style="margin-top:1rem">
      <h2>Live region (example)</h2>
      <div id="live" aria-live="polite">No announcements yet.</div>
    </section>

  </main>

  <footer role="contentinfo">
    <p>Test page created to verify AccessiBe overlay / script behavior. Includes accessible landmarks and keyboard-interactive components.</p>
  </footer>

  <!-- ------------------------------------------------------------------ -->
  <!-- ACCESSIBE SCRIPT HERE -->
  <!-- Paste your AccessiBe script tag (the one you received from AccessiBe) right below this comment, for example:
       <script defer src="https://cdn.accessibe.com/script/XXXXX.js"></script>
       (do NOT paste any private keys here in public environments) -->
  <!-- ------------------------------------------------------------------ -->

  <script>
    // Modal focus trap and Escape handling
    (function(){
      const open = document.getElementById('open-modal');
      const modal = document.getElementById('demo-modal');
      const close = document.getElementById('close-modal');
      let lastFocused = null;
      open.addEventListener('click', ()=>{
        lastFocused = document.activeElement;
        modal.hidden = false;
        modal.querySelector('button').focus();
      });
      close.addEventListener('click', ()=>{ modal.hidden=true; lastFocused && lastFocused.focus(); });
      document.addEventListener('keydown', (e)=>{
        if(e.key==='Escape' && !modal.hidden){ modal.hidden=true; lastFocused && lastFocused.focus(); }
      });
    })();

    // Simple form validation demo
    document.getElementById('contact-form').addEventListener('submit', function(e){
      e.preventDefault();
      const email = document.getElementById('email');
      const result = document.getElementById('form-result');
      if(!email.checkValidity()){ result.textContent = 'Please enter a valid email.'; email.focus(); }
      else { result.textContent = 'Form looks good (demo).'; document.getElementById('live').textContent = 'Form submitted at ' + new Date().toLocaleTimeString(); }
    });

    // AccessiBe detection utilities
    (function(){
      const statusEl = document.getElementById('acsb-status');
      const listEl = document.getElementById('script-list');
      const showBtn = document.getElementById('show-scripts');
      const manualBtn = document.getElementById('manual-scan');

      function setStatus(text, ok){
        statusEl.textContent = text;
        statusEl.style.color = ok ? 'green' : 'crimson';
      }

      function checkForAccessiBe(){
        // 1) Look for known global names (AccessiBe often exposes a global like 'acsb' or similar)
        try{
          if(window.acsb || window.accessible || window.accessiBe || window.accessiBeApp) return true;
        }catch(e){}
        // 2) Look through script tags for keywords in src or data attributes
        const scripts = Array.from(document.getElementsByTagName('script'));
        for(const s of scripts){
          const src = s.getAttribute('src') || '';
          const attrs = (s.getAttributeNames && s.getAttributeNames().join(' ')) || '';
          const combined = (src + ' ' + attrs + ' ' + (s.innerText || '')).toLowerCase();
          if(combined.includes('accessi') || combined.includes('acsb') || combined.includes('accessibe')) return true;
        }
        return false;
      }

      function listScripts(){
        const scripts = Array.from(document.getElementsByTagName('script'));
        return scripts.map(s=>{
          return (s.src ? s.src : '(inline script)') + (s.dataset && Object.keys(s.dataset).length ? ' data: '+JSON.stringify(s.dataset) : '');
        }).join('\n');
      }

      function update(){
        const found = checkForAccessiBe();
        setStatus(found ? 'Detected — AccessiBe script appears to be present.' : 'Not detected — AccessiBe script not found yet.');
      }

      // Mutation observer to detect when scripts load/are added
      const mo = new MutationObserver((mutations)=>{
        for(const m of mutations){
          if(m.addedNodes){
            for(const n of m.addedNodes){
              if(n.tagName && n.tagName.toLowerCase()==='script'){
                // small delay to allow script to initialize globals
                setTimeout(update, 300);
              }
            }
          }
        }
      });
      mo.observe(document.documentElement || document.body, {childList:true, subtree:true});

      // Buttons
      manualBtn.addEventListener('click', update);
      showBtn.addEventListener('click', ()=>{
        listEl.style.display = listEl.style.display==='none' ? 'block' : 'none';
        listEl.textContent = listScripts();
      });

      // initial
      update();

    })();
  </script>
</body>
</html>
