<!DOCTYPE html>
<html lang="zh-TW" style="background-color: transparent;"><head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html-to-image/1.11.13/html-to-image.min.js" integrity="sha512-iZ2ORl595Wx6miw+GuadDet4WQbdSWS3JLMoNfY8cRGoEFy6oT3G9IbcrBeL6AfkgpA51ETt/faX6yLV+/gFJg==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
    <script>
      (function() {
        // Capture host references before any artifact code runs: Window.parent
        // is [Replaceable] (a top-level `var parent` in artifact code would
        // replace the accessor with a data property), and a top-level
        // `const crypto` would shadow the global — either would otherwise
        // silently break the bridge for artifacts that worked before.
        const realParent = window.parent;
        const cryptoObj = window.crypto;
        // crypto.randomUUID exists only in Secure Contexts; fall back to a
        // unique non-crypto id elsewhere (http://LAN-IP dev flows) —
        // uniqueness is what the bridge needs, unpredictability is
        // defense-in-depth on top of the source guards.
        const newRequestId =
          cryptoObj && typeof cryptoObj.randomUUID === "function"
            ? function () { return cryptoObj.randomUUID(); }
            : function () { return Date.now() + "-" + Math.random(); };
        const originalConsole = window.console;
        window.console = {
          log: (...args) => {
            originalConsole.log(...args);
            realParent.postMessage({ type: 'console', message: args.join(' ') }, '*');
          },
          error: (...args) => {
            originalConsole.error(...args);
            realParent.postMessage({ type: 'console', message: 'Error: ' + args.join(' ') }, '*');
          },
          warn: (...args) => {
            originalConsole.warn(...args);
            realParent.postMessage({ type: 'console', message: 'Warning: ' + args.join(' ') }, '*');
          }
        };

        // Bridge request ids are crypto-random (not sequential) so they
        // cannot be predicted by other frames in the tab.
        let callbacksMap = new Map();
        let streamControllers = new Map();
        
        window.claude = {
          complete: (prompt) => {
            return new Promise((resolve, reject) => {
              const id = newRequestId();
              callbacksMap.set(id, { resolve, reject });
              realParent.postMessage({ type: 'claudeComplete', id, prompt }, '*');
            });
          }
        };

        window.storage = {
          get: (key, shared = false) => {
            return new Promise((resolve, reject) => {
              const id = newRequestId();
              callbacksMap.set(id, { resolve, reject });
              realParent.postMessage({ type: 'storageGet', id, key, shared }, '*');
            });
          },
          set: (key, value, shared = false) => {
            return new Promise((resolve, reject) => {
              const id = newRequestId();
              callbacksMap.set(id, { resolve, reject });
              realParent.postMessage({ type: 'storageSet', id, key, value, shared }, '*');
            });
          },
          delete: (key, shared = false) => {
            return new Promise((resolve, reject) => {
              const id = newRequestId();
              callbacksMap.set(id, { resolve, reject });
              realParent.postMessage({ type: 'storageDelete', id, key, shared }, '*');
            });
          },
          list: (prefix, shared = false) => {
            return new Promise((resolve, reject) => {
              const id = newRequestId();
              callbacksMap.set(id, { resolve, reject });
              realParent.postMessage({ type: 'storageList', id, prefix, shared }, '*');
            });
          }
        };

        let pendingBlobs = new Map();
        URL.createObjectURL = (blob) => {
          // Store the blob and create an ID and URL for it
          const blobId = `blob-${Date.now()}-${Math.random()}`;
          pendingBlobs.set(blobId, blob);
          return `blob-request://${blobId}`;
        };

        URL.revokeObjectURL = (url) => {
          // Remove the blob from our store
          const blobId = url.replace("blob-request://", "");
          pendingBlobs.delete(blobId);
        };

        const getBlobFromURL = (url) => {
          const blobId = url.replace("blob-request://", "");
          return pendingBlobs.get(blobId);
        };

        // Override global fetch with streaming support
        window.fetch = (url, init = {}) => {
          return new Promise((resolve, reject) => {
            const id = newRequestId();
            const channelId = `fetch-${id}-${Date.now()}`;
            
            callbacksMap.set(id, { 
              resolve: (response) => {
                // Null-body statuses: Response(stream, {status: 204}) throws
                // per the Fetch spec, which would escape this resolver and
                // hang the artifact's await forever.
                if (response.status === 204 || response.status === 205 || response.status === 304) {
                  try {
                    resolve(new Response(null, {
                      status: response.status,
                      statusText: response.statusText,
                      headers: response.headers
                    }));
                  } catch (err) {
                    // Invalid statusText/header bytes can throw here too.
                    reject(new TypeError(
                      'Bridge fetch: unconstructable response (status ' + response.status + ')'
                    ));
                  }
                  return;
                }
                // Create a ReadableStream for the response body
                const stream = new ReadableStream({
                  start(controller) {
                    streamControllers.set(channelId, controller);
                  },
                  cancel() {
                    streamControllers.delete(channelId);
                  }
                });
                
                // Create and return the Response with the stream. Response()
                // requires status in [200, 599]; an opaque/no-cors fetch
                // forwards status 0, which would throw here and escape the
                // resolver, hanging the artifact's await. Surface it as a
                // network-error-shaped rejection instead.
                try {
                  resolve(new Response(stream, {
                    status: response.status,
                    statusText: response.statusText,
                    headers: response.headers
                  }));
                } catch (err) {
                  streamControllers.delete(channelId);
                  reject(new TypeError(
                    'Bridge fetch: unconstructable response (status ' + response.status + ')'
                  ));
                }
              },
              reject,
              channelId
            });
            
            realParent.postMessage({
              type: 'proxyFetch',
              id,
              url,
              init,
              channelId
            }, '*');
          });
        };

        window.addEventListener('message', async (event) => {
          // Only the embedding parent may drive the bridge — sibling and
          // nested frames can also postMessage into this window.
          if (event.source !== realParent) return;
          if (event.data.type === 'takeScreenshot') {
            // Echo the request's nonce so the requester can correlate the
            // reply to ITS request — a reply without the expected nonce
            // (e.g. from a stale pre-remount artifact) is ignored upstream.
            const screenshotNonce = event.data.nonce;
            const rootElement = document.getElementById('artifacts-component-root-html');
            if (!rootElement) {
              realParent.postMessage({
                type: 'screenshotError',
                nonce: screenshotNonce,
                error: new Error('Root element not found'),
              }, '*');
              return;
            }
            // Catch CDN load failures (htmlToImage undefined) and toPng errors
            // so the parent always gets a response instead of hanging forever.
            try {
              const screenshot = await htmlToImage.toPng(rootElement, {
                imagePlaceholder:
                  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAAAXNSR0IArs4c6QAAAA1JREFUGFdjePDgwX8ACOQDoNsk0PMAAAAASUVORK5CYII=",
              });
              realParent.postMessage({
                type: 'screenshotData',
                nonce: screenshotNonce,
                data: screenshot,
              }, '*');
            } catch (err) {
              realParent.postMessage({
                type: 'screenshotError',
                nonce: screenshotNonce,
                error: err instanceof Error ? err : new Error(String(err)),
              }, '*');
            }
          } else if (event.data.type === 'claudeComplete') {
            const callback = callbacksMap.get(event.data.id);
            if (!callback) return;
            if (event.data.error) {
              callback.reject(new Error(event.data.error));
            } else {
              callback.resolve(event.data.completion);
            }
            callbacksMap.delete(event.data.id);
          } else if (event.data.type === 'proxyFetchResponse') {
            const callback = callbacksMap.get(event.data.id);
            if (!callback) return;
            if (event.data.error) {
              callback.reject(new Error(event.data.error));
              callbacksMap.delete(event.data.id);
            } else {
              // Initial response with headers, status, etc.
              callback.resolve({
                status: event.data.status,
                statusText: event.data.statusText,
                headers: event.data.headers
              });
              // Don't delete the callback yet if streaming
              if (!event.data.body) {
                callbacksMap.delete(event.data.id);
              }
            }
          } else if (event.data.type === 'proxyFetchStream') {
            // Handle streaming data chunks
            const controller = streamControllers.get(event.data.channelId);
            if (controller) {
              if (event.data.error) {
                controller.error(new Error(event.data.error));
                streamControllers.delete(event.data.channelId);
              } else if (event.data.done) {
                controller.close();
                streamControllers.delete(event.data.channelId);
                // Clean up the callback
                const callback = Array.from(callbacksMap.entries()).find(
                  ([_, value]) => value.channelId === event.data.channelId
                );
                if (callback) {
                  callbacksMap.delete(callback[0]);
                }
              } else if (event.data.chunk) {
                controller.enqueue(new Uint8Array(event.data.chunk));
              }
            }
          } else if (event.data.type === 'storageGet') {
            const callback = callbacksMap.get(event.data.id);
            if (!callback) return;
            if (event.data.error) {
              callback.reject(new Error(event.data.error));
            } else {
              callback.resolve(event.data.result);
            }
            callbacksMap.delete(event.data.id);
          } else if (event.data.type === 'storageSet') {
            const callback = callbacksMap.get(event.data.id);
            if (!callback) return;
            if (event.data.error) {
              callback.reject(new Error(event.data.error));
            } else {
              callback.resolve(event.data.result);
            }
            callbacksMap.delete(event.data.id);
          } else if (event.data.type === 'storageDelete') {
            const callback = callbacksMap.get(event.data.id);
            if (!callback) return;
            if (event.data.error) {
              callback.reject(new Error(event.data.error));
            } else {
              callback.resolve(event.data.result);
            }
            callbacksMap.delete(event.data.id);
          } else if (event.data.type === 'storageList') {
            const callback = callbacksMap.get(event.data.id);
            if (!callback) return;
            if (event.data.error) {
              callback.reject(new Error(event.data.error));
            } else {
              callback.resolve(event.data.result);
            }
            callbacksMap.delete(event.data.id);
          }
        });

        window.addEventListener('click', (event) => {
          const isEl = event.target instanceof HTMLElement;
          if (!isEl) return;
    
          // find ancestor links
          const linkEl = event.target.closest("a");
          if (!linkEl || !linkEl.href) return;
    
          event.preventDefault();
          event.stopImmediatePropagation();
    
          if (linkEl.href.startsWith("blob-request:")) {
            const blob = getBlobFromURL(linkEl.href);
            if (!blob) return;
            void blob.arrayBuffer().then((data) => {
              realParent.postMessage({
                type: "downloadFile",
                filename: linkEl.download,
                data,
                mimeType: blob.type || "application/octet-stream",
              });
            });
          } else if (linkEl.href.startsWith("data:")) {
            const [header, base64Data] = linkEl.href.split(",");
            const mimeMatch = header.match(/data:([^;]+)/);
            const mimeType = mimeMatch ? mimeMatch[1] : "application/octet-stream";
            const binaryString = atob(base64Data);
            const data = Uint8Array.from(binaryString, (c) =>
              c.charCodeAt(0),
            ).buffer;
            realParent.postMessage({
              type: "downloadFile",
              filename: linkEl.download,
              data,
              mimeType,
            });
          } else {
            let linkUrl;
            try {
              linkUrl = new URL(linkEl.href);
            } catch (error) {
              return;
            }
    
            if (linkUrl.hostname === window.location.hostname) return;
      
            realParent.postMessage({
              type: 'openExternal',
              href: linkEl.href,
            }, '*');
          }
      });

        const originalOpen = window.open;
        window.open = function (url) {
          realParent.postMessage({
            type: "openExternal",
            href: url,
          }, "*");
        };

        window.addEventListener('error', (event) => {
          realParent.postMessage({ type: 'console', message: 'Uncaught Error: ' + event.message }, '*');
        });
      })();
    </script>
  
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">
<meta http-equiv="Pragma" content="no-cache">
<meta http-equiv="Expires" content="0">
<title>筱君大隊 任務追蹤</title>
<link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&amp;display=swap" rel="stylesheet">
<style>
:root {
  --bg: #0f0f13;
  --bg2: #17171e;
  --bg3: #1e1e28;
  --border: rgba(255,255,255,0.10);
  --border2: rgba(255,255,255,0.18);
  --text: #f5f4ee;
  --text2: #c8c6bc;
  --text3: #8a8880;
  --purple: #7c6df5;
  --purple-l: #bbb0ff;
  --purple-bg: rgba(124,109,245,0.14);
  --green: #5ab878;
  --green-l: #7ad898;
  --green-bg: rgba(90,184,120,0.14);
  --jade: #4a9e7a;
  --jade-l: #6abf98;
  --jade-bg: rgba(74,158,122,0.14);
  --red: #e05c5c;
  --red-bg: rgba(224,92,92,0.14);
  --amber: #e0a93c;
  --amber-bg: rgba(224,169,60,0.14);
  --gold: #f0c060;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
html { background: #0f0f13; }
body {
  font-family: 'Noto Sans TC', sans-serif;
  background: #0f0f13;
  color: #f5f4ee;
  min-height: 100vh;
  padding-bottom: 3rem;
}

/* Header */
.header {
  background: linear-gradient(180deg, #1a1830 0%, var(--bg) 100%);
  border-bottom: 1px solid var(--border);
  padding: 2rem 1.5rem 1.5rem;
  text-align: center;
  position: relative;
  overflow: hidden;
}
.header::before {
  content: '';
  position: absolute;
  top: -60px; left: 50%; transform: translateX(-50%);
  width: 400px; height: 200px;
  background: radial-gradient(ellipse, rgba(124,109,245,0.2) 0%, transparent 70%);
  pointer-events: none;
}
.header-badge {
  display: inline-block;
  font-size: 11px;
  letter-spacing: 0.12em;
  color: var(--purple-l);
  background: var(--purple-bg);
  border: 1px solid rgba(124,109,245,0.3);
  padding: 4px 12px;
  border-radius: 20px;
  margin-bottom: 12px;
}
.header h1 {
  font-size: 26px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 6px;
  letter-spacing: -0.01em;
}
.header .sub {
  font-size: 13px;
  color: var(--text2);
}
.header .date {
  margin-top: 8px;
  font-size: 12px;
  color: var(--text3);
}

/* Week badge */
.week-tag {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: var(--amber-bg);
  border: 1px solid rgba(224,169,60,0.25);
  color: var(--amber);
  font-size: 11px;
  padding: 3px 10px;
  border-radius: 20px;
  margin-top: 10px;
}

/* Daily Rally */
.rally-section {
  background: linear-gradient(135deg, #1a1428 0%, #131320 100%);
  border-bottom: 1px solid rgba(124,109,245,0.25);
  padding: 1.25rem 1rem 1rem;
}
.rally-day-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  flex-wrap: wrap;
  gap: 6px;
}
.rally-day-label {
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.12em;
  color: var(--purple-l);
  background: var(--purple-bg);
  border: 1px solid rgba(124,109,245,0.3);
  padding: 3px 10px;
  border-radius: 20px;
}
.rally-day-count {
  font-size: 11px;
  color: var(--text3);
}
.rally-quote {
  font-size: 17px;
  font-weight: 700;
  color: var(--text);
  line-height: 1.5;
  margin-bottom: 6px;
  letter-spacing: -0.01em;
}
.rally-sub {
  font-size: 12px;
  color: var(--text2);
  line-height: 1.6;
  margin-bottom: 12px;
}
.rally-actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}
.rally-copy-btn {
  flex: 1;
  min-width: 120px;
  padding: 9px 14px;
  background: var(--purple);
  color: #fff;
  border: none;
  border-radius: 10px;
  font-size: 12px;
  font-weight: 700;
  font-family: 'Noto Sans TC', sans-serif;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
  letter-spacing: 0.02em;
}
.rally-copy-btn:hover { background: #6b5ee0; }
.rally-copy-btn:active { transform: scale(0.98); }
.rally-copy-btn.copied { background: var(--green); }
.rally-nav-btn {
  padding: 9px 14px;
  background: transparent;
  color: var(--text2);
  border: 1px solid var(--border2);
  border-radius: 10px;
  font-size: 12px;
  font-weight: 700;
  font-family: 'Noto Sans TC', sans-serif;
  cursor: pointer;
  transition: background 0.2s;
}
.rally-nav-btn:hover { background: var(--bg2); color: var(--text); }
.rally-progress {
  display: flex;
  gap: 3px;
  margin-bottom: 10px;
  flex-wrap: wrap;
}
.rally-dot {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: rgba(255,255,255,0.12);
}
.rally-dot.past { background: var(--green); }
.rally-dot.today { background: var(--purple); width: 14px; border-radius: 3px; }
.rally-dot.future { background: rgba(255,255,255,0.08); }

/* Source copy button */
.source-copy-btn {
  position: relative; z-index: 1; display: inline-flex; align-items: center; gap: 6px;
  margin-top: 12px; padding: 8px 16px; background: var(--purple); color: #fff;
  border: none; border-radius: 20px; font-size: 12px; font-weight: 700;
  font-family: 'Noto Sans TC', sans-serif; cursor: pointer;
  transition: background 0.2s, transform 0.1s; letter-spacing: 0.02em;
}
.source-copy-btn:hover { background: #6b5ee0; }
.source-copy-btn:active { transform: scale(0.98); }
.source-copy-btn.copied { background: var(--green); }

/* Tabs */
.tabs {
  display: flex;
  gap: 8px;
  padding: 12px 1rem;
  overflow-x: auto;
  scrollbar-width: none;
  -webkit-overflow-scrolling: touch;
  border-bottom: 1px solid var(--border);
}
.tabs::-webkit-scrollbar { display: none; }
.tab {
  flex-shrink: 0;
  padding: 10px 18px;
  font-size: 14px;
  font-weight: 500;
  color: var(--text2);
  border-radius: 20px;
  cursor: pointer;
  border: 1px solid var(--border);
  transition: all 0.2s;
  background: transparent;
  white-space: nowrap;
  min-height: 44px;
  display: flex;
  align-items: center;
}
.tab:hover { color: var(--text); background: var(--bg2); }
.tab.active {
  color: var(--purple-l);
  background: var(--purple-bg);
  border-color: rgba(124,109,245,0.4);
}

/* Content */
.content { display: none; padding: 1rem; }
.content.active { display: block; }

/* Group info */
.group-info {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1.25rem;
  flex-wrap: wrap;
  gap: 8px;
}
.group-name {
  font-size: 18px;
  font-weight: 700;
  color: var(--text);
}
.group-stats {
  font-size: 12px;
  color: var(--text2);
  display: flex;
  gap: 12px;
}
.group-stats span { display: flex; align-items: center; gap: 4px; }

/* Grid */
.members-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 8px;
  margin-bottom: 1.5rem;
}

/* Card */
.member-card {
  background: var(--bg2);
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 12px;
  transition: border-color 0.2s;
}
.member-card:hover { border-color: var(--border2); }
.member-card.alert { border-color: rgba(224,92,92,0.35); }

/* Card header */
.card-top {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 10px;
}
.avatar {
  width: 32px; height: 32px;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 12px; font-weight: 700;
  flex-shrink: 0;
}
.card-name { font-size: 13px; font-weight: 700; color: var(--text); line-height: 1.2; }
.card-role { font-size: 10px; color: var(--text3); margin-top: 1px; }

/* Scores */
.scores {
  display: flex;
  gap: 6px;
  margin-bottom: 10px;
}
.score-box {
  flex: 1;
  background: var(--bg3);
  border-radius: 8px;
  padding: 5px 7px;
  text-align: center;
}
.score-label { font-size: 9px; color: var(--text3); margin-bottom: 1px; }
.score-val { font-size: 12px; font-weight: 700; color: var(--text); }
.score-val.good { color: var(--green); }
.score-val.warn { color: var(--amber); }
.score-val.bad { color: var(--red); }
.score-val.week { color: #7ab8ff; }

/* Section */
.section-title {
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 0.1em;
  color: var(--text3);
  text-transform: uppercase;
  margin: 8px 0 4px;
  padding-top: 8px;
  border-top: 1px solid var(--border);
}

/* Task rows */
.task-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 3px;
  font-size: 10px;
  gap: 4px;
}
.task-num {
  font-size: 9px;
  color: var(--text3);
  min-width: 12px;
  flex-shrink: 0;
}
.task-name { color: var(--text); flex: 1; }
.task-name.done { color: var(--text3); text-decoration: line-through; text-decoration-color: var(--text3); }
.badge {
  font-size: 9px;
  font-weight: 700;
  padding: 2px 6px;
  border-radius: 4px;
  flex-shrink: 0;
  white-space: nowrap;
}
.badge.done { background: var(--green-bg); color: var(--green); }
.badge.miss { background: var(--red-bg); color: var(--red); }
.badge.warn { background: var(--amber-bg); color: var(--amber); }

/* Notify section */
.notify-section {
  background: var(--bg2);
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 1.25rem;
}
.notify-title {
  font-size: 14px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 1rem;
  display: flex;
  align-items: center;
  gap: 8px;
}
.notify-icon {
  width: 28px; height: 28px;
  background: var(--purple-bg);
  border-radius: 8px;
  display: flex; align-items: center; justify-content: center;
  font-size: 14px;
}
.notify-preview {
  background: var(--bg3);
  border-radius: 10px;
  padding: 12px 14px;
  font-size: 12px;
  line-height: 1.8;
  color: var(--text2);
  white-space: pre-wrap;
  max-height: 260px;
  overflow-y: auto;
  margin-bottom: 10px;
  scrollbar-width: thin;
  scrollbar-color: var(--bg3) transparent;
}
.copy-btn {
  width: 100%;
  padding: 10px;
  background: var(--purple);
  color: #fff;
  border: none;
  border-radius: 10px;
  font-size: 13px;
  font-weight: 700;
  font-family: 'Noto Sans TC', sans-serif;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
  letter-spacing: 0.02em;
}
.copy-btn:hover { background: #6b5ee0; }
.copy-btn:active { transform: scale(0.98); }
.copy-btn.copied { background: var(--green); }


/* Hug Section */
.hug-section {
  background: linear-gradient(135deg, rgba(255,182,193,0.08) 0%, rgba(255,218,185,0.08) 100%);
  border: 1px solid rgba(255,182,193,0.25);
  border-radius: 16px;
  padding: 1.25rem;
  margin-bottom: 1.25rem;
}
.hug-header { text-align: center; margin-bottom: 1rem; }
.hug-title { font-size: 16px; font-weight: 700; color: #f5a0b8; margin: 8px 0 4px; }
.hug-sub { font-size: 12px; color: var(--text2); }

/* Pikmin SVG animation */
.pikmin-svg {
  width: 100%;
  max-width: 320px;
  height: 70px;
  overflow: visible;
}
.pk { animation: pk-bounce 1.2s ease-in-out infinite; }
.pk1 { animation-delay: 0s; }
.pk2 { animation-delay: 0.2s; }
.pk3 { animation-delay: 0.4s; }
.pk4 { animation-delay: 0.6s; }
.pk5 { animation-delay: 0.8s; }
.pk6 { animation-delay: 1.0s; }
@keyframes pk-bounce {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-8px); }
}

.hug-cards { display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 10px; }
.hug-card {
  background: var(--bg2);
  border: 1px solid rgba(255,182,193,0.2);
  border-radius: 12px;
  padding: 12px;
}
.hug-name { font-size: 14px; font-weight: 700; color: #f5a0b8; margin-bottom: 4px; }
.hug-score { font-size: 11px; color: var(--text3); margin-bottom: 10px; }
.hug-block { margin-bottom: 8px; }
.hug-label { font-size: 10px; font-weight: 700; color: var(--amber); margin-bottom: 3px; }
.hug-text { font-size: 11px; color: var(--text2); line-height: 1.6; }

/* Duty Section */
.duty-section {
  background: var(--bg2);
  border: 1px solid rgba(124,109,245,0.25);
  border-radius: 16px;
  padding: 1.25rem;
  margin-bottom: 1.25rem;
}
.duty-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 1rem;
}
.duty-icon {
  font-size: 18px;
  width: 36px; height: 36px;
  background: var(--purple-bg);
  border-radius: 10px;
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0;
}
.duty-title { font-size: 15px; font-weight: 700; color: var(--text); }
.duty-sub { font-size: 11px; color: var(--text3); margin-top: 2px; }
.duty-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 8px;
  margin-bottom: 12px;
}
.duty-card {
  background: var(--bg3);
  border-radius: 10px;
  padding: 10px 12px;
}
.duty-person {
  font-size: 12px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 6px;
}
.duty-tasks { display: flex; flex-wrap: wrap; gap: 4px; }
.duty-tag {
  font-size: 10px;
  background: var(--purple-bg);
  color: var(--purple-l);
  border: 1px solid rgba(124,109,245,0.25);
  border-radius: 4px;
  padding: 2px 7px;
}
.duty-note {
  font-size: 11px;
  color: var(--text3);
  background: var(--bg3);
  border-radius: 8px;
  padding: 8px 12px;
  line-height: 1.6;
}

.duty-status-block { margin-top: 8px; display: flex; flex-direction: column; gap: 3px; }
.duty-status-row { font-size: 10px; line-height: 1.5; }
.duty-status-label { color: var(--text2); font-weight: 600; margin-right: 4px; }
.duty-done-list { color: var(--green); }
.duty-miss-label { color: var(--red); font-weight: 700; margin-right: 4px; }
.duty-miss-list { color: var(--text); }
.angel-pair-alert {
  font-size: 10px;
  color: var(--amber);
  background: var(--amber-bg);
  border-radius: 5px;
  padding: 4px 8px;
  margin-top: 5px;
  line-height: 1.5;
}
.angel-pair-note { font-size: 10px; color: var(--text2); line-height: 1.5; margin-top: 4px; }

/* Angel Call Section */
.angel-section {
  background: var(--bg2);
  border: 1px solid rgba(90,184,120,0.25);
  border-radius: 16px;
  padding: 1.25rem;
  margin-bottom: 1.25rem;
}
.angel-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 1rem;
}
.angel-icon {
  font-size: 22px;
  width: 36px; height: 36px;
  background: var(--green-bg);
  border-radius: 10px;
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0;
}
.angel-title { font-size: 15px; font-weight: 700; color: var(--text); }
.angel-sub { font-size: 11px; color: var(--text3); margin-top: 2px; }
.angel-week { margin-bottom: 1rem; }
.angel-week-label {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.08em;
  color: var(--text3);
  text-transform: uppercase;
  margin-bottom: 8px;
  padding-bottom: 4px;
  border-bottom: 1px solid var(--border);
}
.angel-pairs { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 8px; }
.angel-pair {
  background: var(--bg3);
  border-radius: 10px;
  padding: 10px 12px;
  border-left: 3px solid transparent;
}
.angel-pair.done { border-left-color: var(--green); }
.angel-pair.pending { border-left-color: var(--amber); }
.angel-pair.upcoming { border-left-color: var(--border2); opacity: 0.6; }
.angel-pair-names { font-size: 12px; font-weight: 700; color: var(--text); margin-bottom: 3px; }
.angel-pair-time { font-size: 10px; color: var(--text2); margin-bottom: 4px; }
.angel-pair-status { font-size: 10px; font-weight: 700; margin-bottom: 4px; }
.status-done { color: var(--green); }
.status-pending { color: var(--amber); }
.status-upcoming { color: var(--text3); }
.angel-pair-note { font-size: 10px; color: var(--text3); line-height: 1.5; }


/* Item/Equipment Section */
.items-section {
  background: var(--bg2);
  border: 1px solid rgba(201,168,76,0.3);
  border-radius: 16px;
  padding: 1.25rem;
  margin-bottom: 1.25rem;
}
.items-header {
  display: flex; align-items: center; gap: 10px; margin-bottom: 12px;
}
.items-icon {
  font-size: 18px; width: 36px; height: 36px;
  background: var(--gold-bg); border-radius: 10px;
  display: flex; align-items: center; justify-content: center; flex-shrink: 0;
}
.items-title { font-size: 15px; font-weight: 700; color: var(--text); }
.items-sub { font-size: 11px; color: var(--text3); margin-top: 2px; }
.items-grid { display: flex; flex-direction: column; gap: 10px; }
.item-row {
  background: var(--bg3); border-radius: 10px; padding: 10px 12px;
  display: flex; align-items: flex-start; gap: 10px;
}
.item-emoji { font-size: 22px; flex-shrink: 0; margin-top: 2px; }
.item-name { font-size: 13px; font-weight: 700; color: var(--amber); margin-bottom: 4px; }
.item-owners { font-size: 11px; color: var(--text2); line-height: 1.6; }
.item-owners span { 
  display: inline-block; background: var(--gold-bg);
  border: 1px solid rgba(201,168,76,0.25);
  color: var(--text); border-radius: 4px;
  padding: 1px 7px; margin: 1px 2px; font-size: 10px;
}

/* Footer */
.footer {
  text-align: center;
  padding: 1rem;
  font-size: 11px;
  color: var(--text3);
}

/* Alert colors per group */
.av-purple { background: rgba(124,109,245,0.2); color: var(--purple-l); }
.av-green  { background: rgba(90,184,120,0.2);  color: var(--jade-l); }
.av-red    { background: rgba(224,92,92,0.2);   color: var(--red); }
.av-amber  { background: rgba(224,169,60,0.2);  color: var(--amber); }
.av-blue   { background: rgba(80,128,200,0.2);  color: #80b8ff; }
.av-teal   { background: rgba(50,180,190,0.2);  color: #60d8ee; }
.av-pink   { background: rgba(210,100,160,0.2); color: #f080b8; }


/* Gap Chart v2 */
.gap-chart-section { background: #17171e; border: 1px solid rgba(255,255,255,0.08); border-radius: 16px; padding: 1rem; margin-bottom: 1.25rem; }
.gap-chart-title { font-size: 14px; font-weight: 700; color: #f5f4ee; margin-bottom: 2px; }
.gap-chart-sub { font-size: 11px; color: #8a8880; margin-bottom: 12px; }
.gc-list { display: flex; flex-direction: column; gap: 6px; }
.gc-row { background: #1e1e28; border-radius: 10px; overflow: hidden; }
.gc-bar-wrap { display: flex; align-items: center; gap: 8px; padding: 9px 10px; cursor: pointer; user-select: none; }
.gc-bar-wrap:active { background: rgba(255,255,255,0.04); }
.gc-name { width: 52px; font-size: 11px; font-weight: 700; text-align: right; flex-shrink: 0; }
.gc-track { flex: 1; height: 18px; background: rgba(255,255,255,0.06); border-radius: 5px; overflow: hidden; }
.gc-fill { height: 100%; border-radius: 5px; display: flex; align-items: center; padding-left: 5px; min-width: 18px; transition: width 0.6s ease; }
.gc-fill-txt { font-size: 9px; font-weight: 700; color: rgba(255,255,255,0.9); white-space: nowrap; }
.gc-pct { width: 32px; font-size: 10px; font-weight: 700; text-align: right; flex-shrink: 0; }
.gc-arrow { width: 14px; font-size: 9px; color: rgba(255,255,255,0.3); transition: transform 0.2s; flex-shrink: 0; }
.gc-arrow.open { transform: rotate(180deg); }
.gc-detail { display: none; padding: 0 10px 10px; border-top: 1px solid rgba(255,255,255,0.05); }
.gc-detail.open { display: block; }
.gc-section-label { font-size: 10px; color: #8a8880; font-weight: 700; margin: 8px 0 5px; }
.gc-chips { display: flex; flex-wrap: wrap; gap: 5px; }
.gc-chip { font-size: 10px; font-weight: 700; padding: 3px 8px; border-radius: 6px; }
.gc-done { background: rgba(90,184,120,0.15); color: #5ab878; border: 1px solid rgba(90,184,120,0.2); }
.gc-miss { background: rgba(224,92,92,0.1); color: #f87171; border: 1px solid rgba(224,92,92,0.15); }

</style>
</head>
<body id="artifacts-component-root-html" style="background-color: transparent;">

<div class="header">
  <div class="header-badge">筱君大隊</div>
  <h1>🏆 任務追蹤儀表板</h1>
  <div class="sub">第4週 6/15（一）～ 6/21（日）</div>
  <div class="date">截至 6/20 07:17 更新（已含全隊25人）</div>
  <div class="week-tag">📊 第4週進行中（6/20）！本週特殊任務與定課持續累積中</div>
  <button class="source-copy-btn" id="sourceCopyBtn" onclick="copySource(this)">📋 複製整份原始碼（貼到 GitHub）</button>
</div>

<div class="rally-section" id="rallySection">
  <div class="rally-day-row">
    <span class="rally-day-label" id="rallyDayLabel">📅 第 27 天（今天）</span>
    <span class="rally-day-count" id="rallyDayCount">還剩 31 天 · 共 58 天</span>
  </div>
  <div class="rally-progress" id="rallyProgress"><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot past"></div><div class="rally-dot today"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div><div class="rally-dot future"></div></div>
  <div class="rally-quote" id="rallyQuote">🎵 Day 27｜找到你自己的節奏</div>
  <div class="rally-sub" id="rallySub">不是所有人都一樣快，也不是所有人都走同一條路。你的節奏就是最對的節奏。
天使通話記得安排，定課打卡！🌙</div>
  <div class="rally-actions">
    <button class="rally-copy-btn" id="rallyCopyBtn" onclick="copyRally(this)">📣 複製今日精神喊話 ↗</button>
    <button class="rally-nav-btn" onclick="showRallyDay(-1)">← 昨日</button>
    <button class="rally-nav-btn" onclick="showRallyDay(1)">明日 →</button>
  </div>
</div>

<div class="tabs">
  <div class="tab active" onclick="switchTab(0)">✨ 第九組</div>
  <div class="tab" onclick="switchTab(1)">💥 第十組</div>
  <div class="tab" onclick="switchTab(2)">🔥 第十一組</div>
  <div class="tab" onclick="switchTab(3)">🌸 第十二組</div>
</div>

<!-- 第九組 -->
<div class="content active" id="tab0">

<div class="group-info">
    <div class="group-name">佛系但暴富組</div>
    <div class="group-stats">
      <span>7人</span>
      <span>隊長：王薏涵</span>
    </div>
  </div>

  
  <!-- 需要愛的抱抱 -->
  <div class="hug-section">
    <div class="hug-header">
      <div style="display:flex;justify-content:center;margin-bottom:8px">
<svg class="pikmin-svg" viewBox="0 0 320 80" xmlns="http://www.w3.org/2000/svg">
  <g class="pk pk1"><line x1="20" y1="8" x2="20" y2="22" stroke="#4a90d9" stroke-width="1.5"></line><ellipse cx="20" cy="6" rx="3" ry="4" fill="white" stroke="#aaa" stroke-width="0.5"></ellipse><ellipse cx="20" cy="30" rx="9" ry="11" fill="#4a90d9"></ellipse><ellipse cx="20" cy="26" rx="7" ry="7" fill="#5aa0e8"></ellipse><circle cx="17" cy="25" r="2" fill="white"></circle><circle cx="23" cy="25" r="2" fill="white"></circle><circle cx="17.5" cy="25.5" r="1" fill="#222"></circle><circle cx="23.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="20" cy="29" rx="4" ry="2" fill="#3a7abf"></ellipse><line x1="11" y1="28" x2="6" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="29" y1="28" x2="34" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="16" y1="40" x2="14" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="24" y1="40" x2="26" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk2"><line x1="70" y1="8" x2="70" y2="22" stroke="#e8c832" stroke-width="1.5"></line><ellipse cx="70" cy="6" rx="3" ry="4" fill="#f5e070" stroke="#cca820" stroke-width="0.5"></ellipse><ellipse cx="70" cy="30" rx="9" ry="11" fill="#e8c832"></ellipse><ellipse cx="70" cy="26" rx="7" ry="7" fill="#f0d840"></ellipse><circle cx="67" cy="25" r="2" fill="white"></circle><circle cx="73" cy="25" r="2" fill="white"></circle><circle cx="67.5" cy="25.5" r="1" fill="#222"></circle><circle cx="73.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="70" cy="29" rx="4" ry="2" fill="#c8a820"></ellipse><ellipse cx="61" cy="27" rx="4" ry="6" fill="#e8c832"></ellipse><ellipse cx="79" cy="27" rx="4" ry="6" fill="#e8c832"></ellipse><line x1="66" y1="40" x2="64" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"></line><line x1="74" y1="40" x2="76" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk3"><line x1="120" y1="8" x2="120" y2="22" stroke="#e84040" stroke-width="1.5"></line><ellipse cx="120" cy="6" rx="3" ry="4" fill="#ff6060" stroke="#c02020" stroke-width="0.5"></ellipse><ellipse cx="120" cy="30" rx="9" ry="11" fill="#e84040"></ellipse><ellipse cx="120" cy="26" rx="7" ry="7" fill="#f05050"></ellipse><circle cx="117" cy="25" r="2" fill="white"></circle><circle cx="123" cy="25" r="2" fill="white"></circle><circle cx="117.5" cy="25.5" r="1" fill="#222"></circle><circle cx="123.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="120" cy="29" rx="5" ry="2.5" fill="#c02020"></ellipse><line x1="111" y1="28" x2="106" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="129" y1="28" x2="134" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="116" y1="40" x2="114" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="124" y1="40" x2="126" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk4"><line x1="170" y1="6" x2="170" y2="20" stroke="#7b4faa" stroke-width="1.5"></line><ellipse cx="170" cy="4" rx="4" ry="5" fill="#c090e0" stroke="#7b4faa" stroke-width="0.5"></ellipse><ellipse cx="170" cy="31" rx="11" ry="12" fill="#7b4faa"></ellipse><ellipse cx="170" cy="27" rx="9" ry="9" fill="#9060c0"></ellipse><circle cx="166" cy="26" r="2.5" fill="white"></circle><circle cx="174" cy="26" r="2.5" fill="white"></circle><circle cx="166.5" cy="26.5" r="1.2" fill="#222"></circle><circle cx="174.5" cy="26.5" r="1.2" fill="#222"></circle><ellipse cx="170" cy="31" rx="5" ry="2.5" fill="#5a3080"></ellipse><line x1="159" y1="29" x2="153" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="181" y1="29" x2="187" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="165" y1="42" x2="163" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="175" y1="42" x2="177" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line></g>
  <g class="pk pk5"><line x1="220" y1="8" x2="220" y2="22" stroke="#ddd" stroke-width="1.5"></line><ellipse cx="220" cy="6" rx="3" ry="4" fill="#ff88aa" stroke="#ddd" stroke-width="0.5"></ellipse><ellipse cx="220" cy="30" rx="8" ry="10" fill="white" stroke="#ddd" stroke-width="1"></ellipse><ellipse cx="220" cy="26" rx="6" ry="6" fill="#f8f8f8"></ellipse><circle cx="217" cy="25" r="2.5" fill="#ff4466"></circle><circle cx="223" cy="25" r="2.5" fill="#ff4466"></circle><circle cx="217.5" cy="25.5" r="1" fill="#222"></circle><circle cx="223.5" cy="25.5" r="1" fill="#222"></circle><line x1="212" y1="28" x2="207" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="228" y1="28" x2="233" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="216" y1="40" x2="214" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="224" y1="40" x2="226" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk6"><line x1="270" y1="6" x2="270" y2="16" stroke="#666" stroke-width="1.5"></line><ellipse cx="270" cy="4" rx="3" ry="4" fill="#aaa" stroke="#666" stroke-width="0.5"></ellipse><ellipse cx="270" cy="28" rx="11" ry="10" fill="#555"></ellipse><ellipse cx="267" cy="24" rx="3" ry="2.5" fill="#777"></ellipse><ellipse cx="274" cy="23" rx="2.5" ry="2" fill="#777"></ellipse><circle cx="266" cy="24" r="2" fill="white"></circle><circle cx="274" cy="23" r="2" fill="white"></circle><circle cx="266.5" cy="24.5" r="1" fill="#222"></circle><circle cx="274.5" cy="23.5" r="1" fill="#222"></circle><line x1="260" y1="30" x2="255" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="280" y1="30" x2="285" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="265" y1="38" x2="263" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="275" y1="38" x2="277" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"></line></g>
</svg>
</div>
      <div class="hug-title">💝 需要愛的抱抱</div>
      <div class="hug-sub">小隊長請注意！以下夥伴需要你今天主動關心 ✨</div>
    </div>

    <div class="hug-cards">
      <div class="hug-card">
        <div class="hug-name">⚠️ 黃芯璿</div>
        <div class="hug-score">定課完成 2/3 ｜ 本週積分 500</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日0分，尚有1項定課未打卡。本週特殊任務僅完成天使通話。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/18 下午10:22 天使通話，6/18 上午10:30 打拳打卡</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">請小隊長今天關心一下她是否有困難需要幫助，鼓勵補上未完成定課。</div>
        </div>
      </div>
      <div class="hug-card">
        <div class="hug-name">⚠️ 廖志裕</div>
        <div class="hug-score">定課完成 3/3 ｜ 本週積分 6,860</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日0分，最後打卡時間為6/18，今天尚未上線打卡。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/18 下午10:22 完成定課（流動情緒、五感恩、打拳）</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">本週進度良好，提醒他今天記得上線打卡。</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 與滿分之間的差距 -->
  <div class="gap-chart-section" id="g9chart">
    <div class="gap-chart-title">📊 與滿分之間的差距</div>
    <div class="gap-chart-sub">點名字 → 查看已完成／未完成任務 · 滿分 8 項特殊任務</div>
    <div class="gc-list">
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_王薏涵')">
            <div class="gc-name" style="color:#a78bfa">王薏涵</div>
            <div class="gc-track"><div class="gc-fill" style="width:62%;background:#a78bfa"><span class="gc-fill-txt">5/8</span></div></div>
            <div class="gc-pct" style="color:#5ab878">62%</div>
            <div class="gc-arrow" id="arr_g9chart_王薏涵">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_王薏涵">
            <div class="gc-section-label">✅ 已完成（5 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 欣賞夥伴</span><span class="gc-chip gc-done">✓ 天使通話</span><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 圓夢計劃親證(x2)</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（3 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_王岑芯')">
            <div class="gc-name" style="color:#60a5fa">王岑芯</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#60a5fa"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g9chart_王岑芯">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_王岑芯">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_王宏榮')">
            <div class="gc-name" style="color:#34d399">王宏榮</div>
            <div class="gc-track"><div class="gc-fill" style="width:50%;background:#34d399"><span class="gc-fill-txt">4/8</span></div></div>
            <div class="gc-pct" style="color:#5ab878">50%</div>
            <div class="gc-arrow" id="arr_g9chart_王宏榮">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_王宏榮">
            <div class="gc-section-label">✅ 已完成（4 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 欣賞夥伴</span><span class="gc-chip gc-done">✓ 天使通話</span><span class="gc-chip gc-done">✓ 圓夢計劃親證(x2)</span><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（4 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_黃芯璿')">
            <div class="gc-name" style="color:#fbbf24">黃芯璿</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#fbbf24"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g9chart_黃芯璿">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_黃芯璿">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_鄒念穎')">
            <div class="gc-name" style="color:#f87171">鄒念穎</div>
            <div class="gc-track"><div class="gc-fill" style="width:0%;background:#f87171"><span class="gc-fill-txt">0/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">0%</div>
            <div class="gc-arrow" id="arr_g9chart_鄒念穎">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_鄒念穎">
            <div class="gc-section-label">✅ 已完成（0 項）</div><div class="gc-chips"><span style="color:#8a8880;font-size:10px">尚未完成任何任務</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（8 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_黃雅琪')">
            <div class="gc-name" style="color:#e879f9">黃雅琪</div>
            <div class="gc-track"><div class="gc-fill" style="width:37%;background:#e879f9"><span class="gc-fill-txt">3/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">37%</div>
            <div class="gc-arrow" id="arr_g9chart_黃雅琪">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_黃雅琪">
            <div class="gc-section-label">✅ 已完成（3 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 圓夢計劃親證(x2)</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span><span class="gc-chip gc-done">✓ 巔峰取經</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（5 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 主題親證2</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g9chart_廖志裕')">
            <div class="gc-name" style="color:#fb923c">廖志裕</div>
            <div class="gc-track"><div class="gc-fill" style="width:25%;background:#fb923c"><span class="gc-fill-txt">2/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">25%</div>
            <div class="gc-arrow" id="arr_g9chart_廖志裕">▼</div>
          </div>
          <div class="gc-detail" id="det_g9chart_廖志裕">
            <div class="gc-section-label">✅ 已完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（6 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
    </div>
  </div>

  <!-- 天使通話分組 -->
  <div class="angel-section">
    <div class="angel-header">
      <span class="angel-icon">📞</span>
      <div>
        <div class="angel-title">天使通話分組</div>
        <div class="angel-sub">本週通話配對｜完成後記得打卡！</div>
      </div>
    </div>
    <div class="angel-week">
      <div class="angel-week-label">本週（6/15 – 6/21）</div>
      <div class="angel-pairs">
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 王宏榮（布丁）&amp; 王岑芯</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 黃芯璿（Bella）&amp; 黃雅琪（琪琪）</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 廖志裕 &amp; 鄒念穎（小鄒）</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
      </div>
    </div>
  </div>

  <div class="items-section">
    <div class="items-header">
      <span class="items-icon">⚔️</span>
      <div>
        <div class="items-title">道具持有紀錄</div>
        <div class="items-sub">第9組｜親證班第三週</div>
      </div>
    </div>
    <div class="items-grid">
      <div class="item-row">
        <div class="item-emoji">🔱</div>
        <div>
          <div class="item-name">龍宮玉印</div>
          <div class="item-owners"><span>王宏榮</span><span>廖志裕</span><span>鄒念穎</span><span>黃芯璿</span><span>黃雅琪</span><span>王岑芯</span><span>王薏涵</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🛡️</div>
        <div>
          <div class="item-name">天罡戰鎧</div>
          <div class="item-owners"><span>鄒念穎</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🔫</div>
        <div>
          <div class="item-name">破曉槍</div>
          <div class="item-owners"><span>王薏涵</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🪄</div>
        <div>
          <div class="item-name">如意金箍棒</div>
          <div class="item-owners"><span>王薏涵</span></div>
        </div>
      </div>
    </div>
  </div>

  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg9">✨【第九組｜佛系但暴富組】6/15 新週提醒！

━━━━━━━━━━━━━━
📋 今日定課

新的一週剛開始，請大家記得完成每日3項定課打卡！

━━━━━━━━━━━━━━
🎯 本週特殊任務（8項）

蓋雅的召喚、欣賞夥伴、天使通話、親證分享、圓夢計劃親證(x2)、參加心成活動(x1)、主題親證2、巔峰取經
全員進度歸零，本週尚未開始完成任何項目，加油衝刺！

佛系也要暴富，新一週繼續衝🙏✨</div>
    <button class="copy-btn" onclick="copyMsg('msg9', this)">一鍵複製貼到 Line ↗</button>
  </div>

  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">薏</div><div><div class="card-name">王薏涵</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">28,660</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">6,060</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,140</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">破曉打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">參加心成活動</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">岑</div><div><div class="card-name">王岑芯</div><div class="card-role">觀音菩薩（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">21,980</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">2,580</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,380</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">感恩冥想</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">宏</div><div><div class="card-name">王宏榮</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">29,000</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">8,380</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+2,480</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">當下之舞</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name warn">親證分享</span><span class="badge warn">待確認打卡</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-green">芯</div><div><div class="card-name">黃芯璿</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">14,200</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">500</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x1)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">念</div><div><div class="card-name">鄒念穎</div><div class="card-role">龍太子（金金）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">14,800</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">400</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x1)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">志</div><div><div class="card-name">廖志裕</div><div class="card-role">白龍馬（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">19,460</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">6,860</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x1)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">雅</div><div><div class="card-name">黃雅琪</div><div class="card-role">龍女（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">16,100</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">32,800</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+900</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">巔峰取經</span><span class="badge done">✓</span></div>
    </div>

  </div>
</div>

<!-- 第十組 -->
<div class="content" id="tab1">

<div class="group-info">
    <div class="group-name">十破天驚</div>
    <div class="group-stats"><span>6人</span><span>隊長：游佳霖</span></div>
  </div>

  <div class="hug-section">
    <div class="hug-header">
      <div style="display:flex;justify-content:center;margin-bottom:8px">
<svg class="pikmin-svg" viewBox="0 0 320 80" xmlns="http://www.w3.org/2000/svg">
  <g class="pk pk1"><line x1="20" y1="8" x2="20" y2="22" stroke="#4a90d9" stroke-width="1.5"></line><ellipse cx="20" cy="6" rx="3" ry="4" fill="white" stroke="#aaa" stroke-width="0.5"></ellipse><ellipse cx="20" cy="30" rx="9" ry="11" fill="#4a90d9"></ellipse><ellipse cx="20" cy="26" rx="7" ry="7" fill="#5aa0e8"></ellipse><circle cx="17" cy="25" r="2" fill="white"></circle><circle cx="23" cy="25" r="2" fill="white"></circle><circle cx="17.5" cy="25.5" r="1" fill="#222"></circle><circle cx="23.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="20" cy="29" rx="4" ry="2" fill="#3a7abf"></ellipse><line x1="11" y1="28" x2="6" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="29" y1="28" x2="34" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="16" y1="40" x2="14" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="24" y1="40" x2="26" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk2"><line x1="70" y1="8" x2="70" y2="22" stroke="#e8c832" stroke-width="1.5"></line><ellipse cx="70" cy="6" rx="3" ry="4" fill="#f5e070" stroke="#cca820" stroke-width="0.5"></ellipse><ellipse cx="70" cy="30" rx="9" ry="11" fill="#e8c832"></ellipse><ellipse cx="70" cy="26" rx="7" ry="7" fill="#f0d840"></ellipse><circle cx="67" cy="25" r="2" fill="white"></circle><circle cx="73" cy="25" r="2" fill="white"></circle><circle cx="67.5" cy="25.5" r="1" fill="#222"></circle><circle cx="73.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="70" cy="29" rx="4" ry="2" fill="#c8a820"></ellipse><ellipse cx="61" cy="27" rx="4" ry="6" fill="#e8c832"></ellipse><ellipse cx="79" cy="27" rx="4" ry="6" fill="#e8c832"></ellipse><line x1="66" y1="40" x2="64" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"></line><line x1="74" y1="40" x2="76" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk3"><line x1="120" y1="8" x2="120" y2="22" stroke="#e84040" stroke-width="1.5"></line><ellipse cx="120" cy="6" rx="3" ry="4" fill="#ff6060" stroke="#c02020" stroke-width="0.5"></ellipse><ellipse cx="120" cy="30" rx="9" ry="11" fill="#e84040"></ellipse><ellipse cx="120" cy="26" rx="7" ry="7" fill="#f05050"></ellipse><circle cx="117" cy="25" r="2" fill="white"></circle><circle cx="123" cy="25" r="2" fill="white"></circle><circle cx="117.5" cy="25.5" r="1" fill="#222"></circle><circle cx="123.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="120" cy="29" rx="5" ry="2.5" fill="#c02020"></ellipse><line x1="111" y1="28" x2="106" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="129" y1="28" x2="134" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="116" y1="40" x2="114" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="124" y1="40" x2="126" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk4"><line x1="170" y1="6" x2="170" y2="20" stroke="#7b4faa" stroke-width="1.5"></line><ellipse cx="170" cy="4" rx="4" ry="5" fill="#c090e0" stroke="#7b4faa" stroke-width="0.5"></ellipse><ellipse cx="170" cy="31" rx="11" ry="12" fill="#7b4faa"></ellipse><ellipse cx="170" cy="27" rx="9" ry="9" fill="#9060c0"></ellipse><circle cx="166" cy="26" r="2.5" fill="white"></circle><circle cx="174" cy="26" r="2.5" fill="white"></circle><circle cx="166.5" cy="26.5" r="1.2" fill="#222"></circle><circle cx="174.5" cy="26.5" r="1.2" fill="#222"></circle><ellipse cx="170" cy="31" rx="5" ry="2.5" fill="#5a3080"></ellipse><line x1="159" y1="29" x2="153" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="181" y1="29" x2="187" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="165" y1="42" x2="163" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="175" y1="42" x2="177" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line></g>
  <g class="pk pk5"><line x1="220" y1="8" x2="220" y2="22" stroke="#ddd" stroke-width="1.5"></line><ellipse cx="220" cy="6" rx="3" ry="4" fill="#ff88aa" stroke="#ddd" stroke-width="0.5"></ellipse><ellipse cx="220" cy="30" rx="8" ry="10" fill="white" stroke="#ddd" stroke-width="1"></ellipse><ellipse cx="220" cy="26" rx="6" ry="6" fill="#f8f8f8"></ellipse><circle cx="217" cy="25" r="2.5" fill="#ff4466"></circle><circle cx="223" cy="25" r="2.5" fill="#ff4466"></circle><circle cx="217.5" cy="25.5" r="1" fill="#222"></circle><circle cx="223.5" cy="25.5" r="1" fill="#222"></circle><line x1="212" y1="28" x2="207" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="228" y1="28" x2="233" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="216" y1="40" x2="214" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="224" y1="40" x2="226" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk6"><line x1="270" y1="6" x2="270" y2="16" stroke="#666" stroke-width="1.5"></line><ellipse cx="270" cy="4" rx="3" ry="4" fill="#aaa" stroke="#666" stroke-width="0.5"></ellipse><ellipse cx="270" cy="28" rx="11" ry="10" fill="#555"></ellipse><ellipse cx="267" cy="24" rx="3" ry="2.5" fill="#777"></ellipse><ellipse cx="274" cy="23" rx="2.5" ry="2" fill="#777"></ellipse><circle cx="266" cy="24" r="2" fill="white"></circle><circle cx="274" cy="23" r="2" fill="white"></circle><circle cx="266.5" cy="24.5" r="1" fill="#222"></circle><circle cx="274.5" cy="23.5" r="1" fill="#222"></circle><line x1="260" y1="30" x2="255" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="280" y1="30" x2="285" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="265" y1="38" x2="263" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="275" y1="38" x2="277" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"></line></g>
</svg>
</div>
      <div class="hug-title">💝 需要愛的抱抱</div>
      <div class="hug-sub">小隊長請注意！以下夥伴需要你今天主動關心 ✨</div>
    </div>
    <div class="hug-cards">
      <div class="hug-card">
        <div class="hug-name">⚠️ 蔡鎔庄</div>
        <div class="hug-score">定課完成 0/3 ｜ 本週積分 3,420</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日0分，3項定課皆未打卡，最後打卡時間為6/18。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/18 上午08:24 完成定課（打拳、流動情緒、亥子時入睡）</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">提醒今天記得上線打卡，本週進度尚可，別讓連續紀錄中斷。</div>
        </div>
      </div>
      <div class="hug-card">
        <div class="hug-name">⚠️ 羅萱</div>
        <div class="hug-score">定課完成 0/3 ｜ 本週積分 35,400</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日0分，3項定課皆未打卡，最後打卡時間為6/18。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/18 下午01:49 完成定課（一日一蔬食、五感恩、打拳）</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">本週積分亮眼，提醒今天記得回來打卡定課，保持氣勢！</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 與滿分之間的差距 -->
  <div class="gap-chart-section" id="g10chart">
    <div class="gap-chart-title">📊 與滿分之間的差距</div>
    <div class="gap-chart-sub">點名字 → 查看已完成／未完成任務 · 滿分 8 項特殊任務</div>
    <div class="gc-list">
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_蔡鎔庄')">
            <div class="gc-name" style="color:#34d399">蔡鎔庄</div>
            <div class="gc-track"><div class="gc-fill" style="width:37%;background:#34d399"><span class="gc-fill-txt">3/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">37%</div>
            <div class="gc-arrow" id="arr_g10chart_蔡鎔庄">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_蔡鎔庄">
            <div class="gc-section-label">✅ 已完成（3 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 圓夢計劃親證(x2)</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（5 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_游佳霖')">
            <div class="gc-name" style="color:#a78bfa">游佳霖</div>
            <div class="gc-track"><div class="gc-fill" style="width:50%;background:#a78bfa"><span class="gc-fill-txt">4/8</span></div></div>
            <div class="gc-pct" style="color:#5ab878">50%</div>
            <div class="gc-arrow" id="arr_g10chart_游佳霖">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_游佳霖">
            <div class="gc-section-label">✅ 已完成（4 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 圓夢計劃親證(x2)</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span><span class="gc-chip gc-done">✓ 巔峰取經</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（4 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 主題親證2</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_游文君')">
            <div class="gc-name" style="color:#60a5fa">游文君</div>
            <div class="gc-track"><div class="gc-fill" style="width:50%;background:#60a5fa"><span class="gc-fill-txt">4/8</span></div></div>
            <div class="gc-pct" style="color:#5ab878">50%</div>
            <div class="gc-arrow" id="arr_g10chart_游文君">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_游文君">
            <div class="gc-section-label">✅ 已完成（4 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span><span class="gc-chip gc-done">✓ 巔峰取經</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（4 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 主題親證2</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_王依涵')">
            <div class="gc-name" style="color:#f87171">王依涵</div>
            <div class="gc-track"><div class="gc-fill" style="width:25%;background:#f87171"><span class="gc-fill-txt">2/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">25%</div>
            <div class="gc-arrow" id="arr_g10chart_王依涵">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_王依涵">
            <div class="gc-section-label">✅ 已完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 欣賞夥伴</span><span class="gc-chip gc-done">✓ 親證分享</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（6 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_羅萱')">
            <div class="gc-name" style="color:#fbbf24">羅萱</div>
            <div class="gc-track"><div class="gc-fill" style="width:37%;background:#fbbf24"><span class="gc-fill-txt">3/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">37%</div>
            <div class="gc-arrow" id="arr_g10chart_羅萱">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_羅萱">
            <div class="gc-section-label">✅ 已完成（3 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 主題親證2</span><span class="gc-chip gc-done">✓ 巔峰取經</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（5 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g10chart_李雯萱')">
            <div class="gc-name" style="color:#fb923c">李雯萱</div>
            <div class="gc-track"><div class="gc-fill" style="width:37%;background:#fb923c"><span class="gc-fill-txt">3/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">37%</div>
            <div class="gc-arrow" id="arr_g10chart_李雯萱">▼</div>
          </div>
          <div class="gc-detail" id="det_g10chart_李雯萱">
            <div class="gc-section-label">✅ 已完成（3 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（5 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
    </div>
  </div>

  <!-- 天使通話分組 -->
  <div class="angel-section">
    <div class="angel-header">
      <span class="angel-icon">📞</span>
      <div>
        <div class="angel-title">天使通話分組</div>
        <div class="angel-sub">本週通話配對｜完成後記得打卡！</div>
      </div>
    </div>
    <div class="angel-week">
      <div class="angel-week-label">本週（6/15 – 6/21）</div>
      <div class="angel-pairs">
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 游佳霖 &amp; 羅萱</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 游文君 &amp; 李雯萱</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 蔡鎔庄 &amp; 王依涵</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
      </div>
    </div>
  </div>

  <div class="items-section">
    <div class="items-header">
      <span class="items-icon">⚔️</span>
      <div>
        <div class="items-title">道具持有紀錄</div>
        <div class="items-sub">第10組｜6/8–6/14 親證班第三週</div>
      </div>
    </div>
    <div class="items-grid">
      <div class="item-row">
        <div class="item-emoji">🔱</div>
        <div>
          <div class="item-name">龍宮玉印</div>
          <div class="item-owners"><span>游文君</span><span>游佳霖</span><span>羅萱</span><span>王依涵</span><span>李雯萱</span><span>蔡鎔庄</span><span>許玲慧（蜜卡）</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🔫</div>
        <div>
          <div class="item-name">破曉火尖槍</div>
          <div class="item-owners"><span>游文君</span><span>蔡鎔庄</span><span>游佳霖</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🛡️</div>
        <div>
          <div class="item-name">天罡戰鎧</div>
          <div class="item-owners"><span>游佳霖</span><span>許玲慧（蜜卡）</span><span>羅萱</span><span>蔡鎔庄</span><span>李雯萱</span></div>
        </div>
      </div>
      <div class="item-row">
        <div class="item-emoji">🪄</div>
        <div>
          <div class="item-name">如意金箍棒</div>
          <div class="item-owners"><span>游文君</span><span>游佳霖</span><span>蔡鎔庄</span><span>羅萱</span><span>許玲慧（蜜卡）</span><span>李雯萱</span></div>
        </div>
      </div>
    </div>
  </div>

  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg10">💥【第十組｜十破天驚】6/15 新週提醒！

━━━━━━━━━━━━━━
📋 今日定課

新的一週剛開始，請大家記得完成每日3項定課打卡！

━━━━━━━━━━━━━━
🎯 本週特殊任務（8項）

蓋雅的召喚、欣賞夥伴、天使通話、親證分享、圓夢計劃親證(x2)、參加心成活動(x1)、主題親證2、巔峰取經
全員進度歸零，本週尚未開始完成任何項目，加油衝刺！

十破天驚新週新氣象，繼續衝🔥</div>
    <button class="copy-btn" onclick="copyMsg('msg10', this)">一鍵複製貼到 Line ↗</button>
  </div>

  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">庄</div><div><div class="card-name">蔡鎔庄</div><div class="card-role">龍太子（金金）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">34,040</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">3,420</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">霖</div><div><div class="card-name">游佳霖</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">35,700</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">42,640</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,000</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">巔峰取經</span><span class="badge done">✓</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">萱</div><div><div class="card-name">羅萱</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">25,680</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">35,400</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x1)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">巔峰取經</span><span class="badge done">✓</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">文</div><div><div class="card-name">游文君</div><div class="card-role">觀音菩薩（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">30,380</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">38,820</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+3,900</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">感恩冥想</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">流動情緒(觀呼吸)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">每日五感恩</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">巔峰取經</span><span class="badge done">✓</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">依</div><div><div class="card-name">王依涵</div><div class="card-role">白龍馬（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">26,200</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">8,500</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+4,700</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">破曉打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name warn">圓夢計劃親證(x2)</span><span class="badge warn">待確認打卡</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">萱</div><div><div class="card-name">李雯萱</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">26,820</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">8,120</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+4,020</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">流動情緒(觀呼吸)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

  </div>
</div>

<!-- 第十一組 -->
<div class="content" id="tab2">

<div class="group-info">
    <div class="group-name">修心之路</div>
    <div class="group-stats"><span>6人</span><span>隊長：郭筱婷</span></div>
  </div>

  <div class="hug-section">
    <div class="hug-header">
      <div style="display:flex;justify-content:center;margin-bottom:8px">
<svg class="pikmin-svg" viewBox="0 0 320 80" xmlns="http://www.w3.org/2000/svg">
  <g class="pk pk1"><line x1="20" y1="8" x2="20" y2="22" stroke="#4a90d9" stroke-width="1.5"></line><ellipse cx="20" cy="6" rx="3" ry="4" fill="white" stroke="#aaa" stroke-width="0.5"></ellipse><ellipse cx="20" cy="30" rx="9" ry="11" fill="#4a90d9"></ellipse><ellipse cx="20" cy="26" rx="7" ry="7" fill="#5aa0e8"></ellipse><circle cx="17" cy="25" r="2" fill="white"></circle><circle cx="23" cy="25" r="2" fill="white"></circle><circle cx="17.5" cy="25.5" r="1" fill="#222"></circle><circle cx="23.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="20" cy="29" rx="4" ry="2" fill="#3a7abf"></ellipse><line x1="11" y1="28" x2="6" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="29" y1="28" x2="34" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="16" y1="40" x2="14" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="24" y1="40" x2="26" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk2"><line x1="70" y1="8" x2="70" y2="22" stroke="#e8c832" stroke-width="1.5"></line><ellipse cx="70" cy="6" rx="3" ry="4" fill="#f5e070" stroke="#cca820" stroke-width="0.5"></ellipse><ellipse cx="70" cy="30" rx="9" ry="11" fill="#e8c832"></ellipse><ellipse cx="70" cy="26" rx="7" ry="7" fill="#f0d840"></ellipse><circle cx="67" cy="25" r="2" fill="white"></circle><circle cx="73" cy="25" r="2" fill="white"></circle><circle cx="67.5" cy="25.5" r="1" fill="#222"></circle><circle cx="73.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="70" cy="29" rx="4" ry="2" fill="#c8a820"></ellipse><ellipse cx="61" cy="27" rx="4" ry="6" fill="#e8c832"></ellipse><ellipse cx="79" cy="27" rx="4" ry="6" fill="#e8c832"></ellipse><line x1="66" y1="40" x2="64" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"></line><line x1="74" y1="40" x2="76" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk3"><line x1="120" y1="8" x2="120" y2="22" stroke="#e84040" stroke-width="1.5"></line><ellipse cx="120" cy="6" rx="3" ry="4" fill="#ff6060" stroke="#c02020" stroke-width="0.5"></ellipse><ellipse cx="120" cy="30" rx="9" ry="11" fill="#e84040"></ellipse><ellipse cx="120" cy="26" rx="7" ry="7" fill="#f05050"></ellipse><circle cx="117" cy="25" r="2" fill="white"></circle><circle cx="123" cy="25" r="2" fill="white"></circle><circle cx="117.5" cy="25.5" r="1" fill="#222"></circle><circle cx="123.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="120" cy="29" rx="5" ry="2.5" fill="#c02020"></ellipse><line x1="111" y1="28" x2="106" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="129" y1="28" x2="134" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="116" y1="40" x2="114" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="124" y1="40" x2="126" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk4"><line x1="170" y1="6" x2="170" y2="20" stroke="#7b4faa" stroke-width="1.5"></line><ellipse cx="170" cy="4" rx="4" ry="5" fill="#c090e0" stroke="#7b4faa" stroke-width="0.5"></ellipse><ellipse cx="170" cy="31" rx="11" ry="12" fill="#7b4faa"></ellipse><ellipse cx="170" cy="27" rx="9" ry="9" fill="#9060c0"></ellipse><circle cx="166" cy="26" r="2.5" fill="white"></circle><circle cx="174" cy="26" r="2.5" fill="white"></circle><circle cx="166.5" cy="26.5" r="1.2" fill="#222"></circle><circle cx="174.5" cy="26.5" r="1.2" fill="#222"></circle><ellipse cx="170" cy="31" rx="5" ry="2.5" fill="#5a3080"></ellipse><line x1="159" y1="29" x2="153" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="181" y1="29" x2="187" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="165" y1="42" x2="163" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="175" y1="42" x2="177" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line></g>
  <g class="pk pk5"><line x1="220" y1="8" x2="220" y2="22" stroke="#ddd" stroke-width="1.5"></line><ellipse cx="220" cy="6" rx="3" ry="4" fill="#ff88aa" stroke="#ddd" stroke-width="0.5"></ellipse><ellipse cx="220" cy="30" rx="8" ry="10" fill="white" stroke="#ddd" stroke-width="1"></ellipse><ellipse cx="220" cy="26" rx="6" ry="6" fill="#f8f8f8"></ellipse><circle cx="217" cy="25" r="2.5" fill="#ff4466"></circle><circle cx="223" cy="25" r="2.5" fill="#ff4466"></circle><circle cx="217.5" cy="25.5" r="1" fill="#222"></circle><circle cx="223.5" cy="25.5" r="1" fill="#222"></circle><line x1="212" y1="28" x2="207" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="228" y1="28" x2="233" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="216" y1="40" x2="214" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="224" y1="40" x2="226" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk6"><line x1="270" y1="6" x2="270" y2="16" stroke="#666" stroke-width="1.5"></line><ellipse cx="270" cy="4" rx="3" ry="4" fill="#aaa" stroke="#666" stroke-width="0.5"></ellipse><ellipse cx="270" cy="28" rx="11" ry="10" fill="#555"></ellipse><ellipse cx="267" cy="24" rx="3" ry="2.5" fill="#777"></ellipse><ellipse cx="274" cy="23" rx="2.5" ry="2" fill="#777"></ellipse><circle cx="266" cy="24" r="2" fill="white"></circle><circle cx="274" cy="23" r="2" fill="white"></circle><circle cx="266.5" cy="24.5" r="1" fill="#222"></circle><circle cx="274.5" cy="23.5" r="1" fill="#222"></circle><line x1="260" y1="30" x2="255" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="280" y1="30" x2="285" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="265" y1="38" x2="263" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="275" y1="38" x2="277" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"></line></g>
</svg>
</div>
      <div class="hug-title">💝 需要愛的抱抱</div>
      <div class="hug-sub">小隊長請注意！以下夥伴需要你今天主動關心 ✨</div>
    </div>
    <div class="hug-cards">
      <div class="hug-card">
        <div class="hug-name">⚠️ 許哲豪</div>
        <div class="hug-score">定課完成 0/3 ｜ 本週積分 3,860</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日0分，3項定課皆未打卡，最後打卡時間為6/18。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/18 下午09:59 完成定課（破曉打拳、亥子時入睡、五感恩、打拳）</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">提醒今天記得回來打卡，本週進度尚可，別中斷連續紀錄。</div>
        </div>
      </div>
      <div class="hug-card">
        <div class="hug-name">⚠️ 王芷盈</div>
        <div class="hug-score">定課完成 0/3 ｜ 本週積分 2,900</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日0分，3項定課皆未打卡，最後打卡時間為6/18。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/18 下午10:08 完成定課（當下之舞、五感恩、打拳）</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">本週特殊任務完成度高，提醒今天記得補上定課打卡。</div>
        </div>
      </div>
      <div class="hug-card">
        <div class="hug-name">⚠️ 賴冠臻</div>
        <div class="hug-score">定課完成 0/3 ｜ 本週積分 1,000</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日+300分，但3項定課尚未全部打卡。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/19 上午01:09 蓋雅的召喚打卡</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">作為副隊長需要特別支持，請隊長今天私訊關心並一起確認本週任務計畫。</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 與滿分之間的差距 -->
  <div class="gap-chart-section" id="g11chart">
    <div class="gap-chart-title">📊 與滿分之間的差距</div>
    <div class="gap-chart-sub">點名字 → 查看已完成／未完成任務 · 滿分 8 項特殊任務</div>
    <div class="gc-list">
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_郭筱婷')">
            <div class="gc-name" style="color:#a78bfa">郭筱婷</div>
            <div class="gc-track"><div class="gc-fill" style="width:25%;background:#a78bfa"><span class="gc-fill-txt">2/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">25%</div>
            <div class="gc-arrow" id="arr_g11chart_郭筱婷">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_郭筱婷">
            <div class="gc-section-label">✅ 已完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 圓夢計劃親證(x2)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（6 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_黃湘庭')">
            <div class="gc-name" style="color:#fbbf24">黃湘庭</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#fbbf24"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g11chart_黃湘庭">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_黃湘庭">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_許哲豪')">
            <div class="gc-name" style="color:#fb923c">許哲豪</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#fb923c"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g11chart_許哲豪">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_許哲豪">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_王芷盈')">
            <div class="gc-name" style="color:#60a5fa">王芷盈</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#60a5fa"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g11chart_王芷盈">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_王芷盈">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_陳惠玲')">
            <div class="gc-name" style="color:#34d399">陳惠玲</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#34d399"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g11chart_陳惠玲">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_陳惠玲">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g11chart_賴冠臻')">
            <div class="gc-name" style="color:#f87171">賴冠臻</div>
            <div class="gc-track"><div class="gc-fill" style="width:12%;background:#f87171"><span class="gc-fill-txt">1/8</span></div></div>
            <div class="gc-pct" style="color:#e05c5c">12%</div>
            <div class="gc-arrow" id="arr_g11chart_賴冠臻">▼</div>
          </div>
          <div class="gc-detail" id="det_g11chart_賴冠臻">
            <div class="gc-section-label">✅ 已完成（1 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（7 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 參加心成活動(x1)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
    </div>
  </div>

  <!-- 天使通話分組 -->
  <div class="angel-section">
    <div class="angel-header">
      <span class="angel-icon">📞</span>
      <div>
        <div class="angel-title">天使通話分組</div>
        <div class="angel-sub">為期2週，每週一次通話｜6/15 – 6/28</div>
      </div>
    </div>
    <div class="angel-week">
      <div class="angel-week-label">第1週（6/15 – 6/21）</div>
      <div class="angel-pairs">
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 郭筱婷 &amp; 許哲豪</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 陳惠玲 &amp; 賴冠臻</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 王芷盈 &amp; 黃湘庭</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 任務負責分工 -->
  <div class="duty-section">
    <div class="duty-header">
      <span class="duty-icon">📋</span>
      <div>
        <div class="duty-title">任務提醒分工</div>
        <div class="duty-sub">隊長確認未完成名單 → 通知對應負責人追蹤｜本週新任務尚未分工</div>
      </div>
    </div>
    <div class="duty-grid">
      <div class="duty-card">
        <div class="duty-person">👑 郭筱婷（Yoyo）負責</div>
        <div class="duty-tasks"><span class="duty-tag">圓夢計劃親證(x2)</span><span class="duty-tag">參加心成活動(x1)</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row"><span class="duty-miss-label">❌ 未完成</span><span class="duty-miss-list">全員（本週剛開始）</span></div>
        </div>
      </div>
      <div class="duty-card">
        <div class="duty-person">🔔 許哲豪 負責</div>
        <div class="duty-tasks"><span class="duty-tag">欣賞夥伴</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row"><span class="duty-miss-label">❌ 未完成</span><span class="duty-miss-list">全員（本週剛開始）</span></div>
        </div>
      </div>
      <div class="duty-card">
        <div class="duty-person">🔔 黃湘庭 負責</div>
        <div class="duty-tasks"><span class="duty-tag">蓋雅的召喚</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row"><span class="duty-miss-label">❌ 未完成</span><span class="duty-miss-list">全員（本週剛開始）</span></div>
        </div>
      </div>
      <div class="duty-card">
        <div class="duty-person">🔔 陳惠玲 負責</div>
        <div class="duty-tasks"><span class="duty-tag">主題親證2</span><span class="duty-tag">巔峰取經</span></div>
        <div class="duty-status-block">
          <div class="duty-status-row"><span class="duty-miss-label">❌ 未完成</span><span class="duty-miss-list">全員（本週剛開始）</span></div>
        </div>
      </div>
    </div>
  </div>

  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg11">🌸【第十一組｜修心之路】6/15 新週提醒！

━━━━━━━━━━━━━━
📋 今日定課

新的一週剛開始，請大家記得完成每日3項定課打卡！

━━━━━━━━━━━━━━
🎯 本週特殊任務（8項）

蓋雅的召喚、欣賞夥伴、天使通話、親證分享、圓夢計劃親證(x2)、參加心成活動(x1)、主題親證2、巔峰取經
全員進度歸零，本週尚未開始完成任何項目，加油衝刺！

修心之路新一週一起加油✨</div>
    <button class="copy-btn" onclick="copyMsg('msg11', this)">一鍵複製貼到 Line ↗</button>
  </div>

  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-purple">筱</div><div><div class="card-name">郭筱婷</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">31,300</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">6,260</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+2,080</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">流動情緒(觀呼吸)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">破曉打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x1)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">湘</div><div><div class="card-name">黃湘庭</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">29,060</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">4,760</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,260</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">當下之舞</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">親證分享</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">哲</div><div><div class="card-name">許哲豪</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">28,240</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">3,860</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name">蓋雅的召喚</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x1)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">惠</div><div><div class="card-name">陳惠玲</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">29,120</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">6,420</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+1,480</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">感恩冥想</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-red">芷</div><div><div class="card-name">王芷盈</div><div class="card-role">沙悟淨（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">21,200</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">2,900</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x1)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">冠</div><div><div class="card-name">賴冠臻</div><div class="card-role">唐三藏（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">14,300</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">1,000</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+300</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">欣賞夥伴</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">參加心成活動(x1)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

  </div>
</div>

<!-- 第十二組 -->
<div class="content" id="tab3">

<div class="group-info">
    <div class="group-name">齊天戰神突擊隊</div>
    <div class="group-stats"><span>6人</span><span>隊長：黃怡駿</span></div>
  </div>

  <div class="hug-section">
    <div class="hug-header">
      <div style="display:flex;justify-content:center;margin-bottom:8px">
<svg class="pikmin-svg" viewBox="0 0 320 80" xmlns="http://www.w3.org/2000/svg">
  <g class="pk pk1"><line x1="20" y1="8" x2="20" y2="22" stroke="#4a90d9" stroke-width="1.5"></line><ellipse cx="20" cy="6" rx="3" ry="4" fill="white" stroke="#aaa" stroke-width="0.5"></ellipse><ellipse cx="20" cy="30" rx="9" ry="11" fill="#4a90d9"></ellipse><ellipse cx="20" cy="26" rx="7" ry="7" fill="#5aa0e8"></ellipse><circle cx="17" cy="25" r="2" fill="white"></circle><circle cx="23" cy="25" r="2" fill="white"></circle><circle cx="17.5" cy="25.5" r="1" fill="#222"></circle><circle cx="23.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="20" cy="29" rx="4" ry="2" fill="#3a7abf"></ellipse><line x1="11" y1="28" x2="6" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="29" y1="28" x2="34" y2="24" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="16" y1="40" x2="14" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line><line x1="24" y1="40" x2="26" y2="52" stroke="#4a90d9" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk2"><line x1="70" y1="8" x2="70" y2="22" stroke="#e8c832" stroke-width="1.5"></line><ellipse cx="70" cy="6" rx="3" ry="4" fill="#f5e070" stroke="#cca820" stroke-width="0.5"></ellipse><ellipse cx="70" cy="30" rx="9" ry="11" fill="#e8c832"></ellipse><ellipse cx="70" cy="26" rx="7" ry="7" fill="#f0d840"></ellipse><circle cx="67" cy="25" r="2" fill="white"></circle><circle cx="73" cy="25" r="2" fill="white"></circle><circle cx="67.5" cy="25.5" r="1" fill="#222"></circle><circle cx="73.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="70" cy="29" rx="4" ry="2" fill="#c8a820"></ellipse><ellipse cx="61" cy="27" rx="4" ry="6" fill="#e8c832"></ellipse><ellipse cx="79" cy="27" rx="4" ry="6" fill="#e8c832"></ellipse><line x1="66" y1="40" x2="64" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"></line><line x1="74" y1="40" x2="76" y2="52" stroke="#e8c832" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk3"><line x1="120" y1="8" x2="120" y2="22" stroke="#e84040" stroke-width="1.5"></line><ellipse cx="120" cy="6" rx="3" ry="4" fill="#ff6060" stroke="#c02020" stroke-width="0.5"></ellipse><ellipse cx="120" cy="30" rx="9" ry="11" fill="#e84040"></ellipse><ellipse cx="120" cy="26" rx="7" ry="7" fill="#f05050"></ellipse><circle cx="117" cy="25" r="2" fill="white"></circle><circle cx="123" cy="25" r="2" fill="white"></circle><circle cx="117.5" cy="25.5" r="1" fill="#222"></circle><circle cx="123.5" cy="25.5" r="1" fill="#222"></circle><ellipse cx="120" cy="29" rx="5" ry="2.5" fill="#c02020"></ellipse><line x1="111" y1="28" x2="106" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="129" y1="28" x2="134" y2="24" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="116" y1="40" x2="114" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line><line x1="124" y1="40" x2="126" y2="52" stroke="#e84040" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk4"><line x1="170" y1="6" x2="170" y2="20" stroke="#7b4faa" stroke-width="1.5"></line><ellipse cx="170" cy="4" rx="4" ry="5" fill="#c090e0" stroke="#7b4faa" stroke-width="0.5"></ellipse><ellipse cx="170" cy="31" rx="11" ry="12" fill="#7b4faa"></ellipse><ellipse cx="170" cy="27" rx="9" ry="9" fill="#9060c0"></ellipse><circle cx="166" cy="26" r="2.5" fill="white"></circle><circle cx="174" cy="26" r="2.5" fill="white"></circle><circle cx="166.5" cy="26.5" r="1.2" fill="#222"></circle><circle cx="174.5" cy="26.5" r="1.2" fill="#222"></circle><ellipse cx="170" cy="31" rx="5" ry="2.5" fill="#5a3080"></ellipse><line x1="159" y1="29" x2="153" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="181" y1="29" x2="187" y2="25" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="165" y1="42" x2="163" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line><line x1="175" y1="42" x2="177" y2="54" stroke="#7b4faa" stroke-width="2" stroke-linecap="round"></line></g>
  <g class="pk pk5"><line x1="220" y1="8" x2="220" y2="22" stroke="#ddd" stroke-width="1.5"></line><ellipse cx="220" cy="6" rx="3" ry="4" fill="#ff88aa" stroke="#ddd" stroke-width="0.5"></ellipse><ellipse cx="220" cy="30" rx="8" ry="10" fill="white" stroke="#ddd" stroke-width="1"></ellipse><ellipse cx="220" cy="26" rx="6" ry="6" fill="#f8f8f8"></ellipse><circle cx="217" cy="25" r="2.5" fill="#ff4466"></circle><circle cx="223" cy="25" r="2.5" fill="#ff4466"></circle><circle cx="217.5" cy="25.5" r="1" fill="#222"></circle><circle cx="223.5" cy="25.5" r="1" fill="#222"></circle><line x1="212" y1="28" x2="207" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="228" y1="28" x2="233" y2="24" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="216" y1="40" x2="214" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line><line x1="224" y1="40" x2="226" y2="52" stroke="#ddd" stroke-width="1.5" stroke-linecap="round"></line></g>
  <g class="pk pk6"><line x1="270" y1="6" x2="270" y2="16" stroke="#666" stroke-width="1.5"></line><ellipse cx="270" cy="4" rx="3" ry="4" fill="#aaa" stroke="#666" stroke-width="0.5"></ellipse><ellipse cx="270" cy="28" rx="11" ry="10" fill="#555"></ellipse><ellipse cx="267" cy="24" rx="3" ry="2.5" fill="#777"></ellipse><ellipse cx="274" cy="23" rx="2.5" ry="2" fill="#777"></ellipse><circle cx="266" cy="24" r="2" fill="white"></circle><circle cx="274" cy="23" r="2" fill="white"></circle><circle cx="266.5" cy="24.5" r="1" fill="#222"></circle><circle cx="274.5" cy="23.5" r="1" fill="#222"></circle><line x1="260" y1="30" x2="255" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="280" y1="30" x2="285" y2="27" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="265" y1="38" x2="263" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"></line><line x1="275" y1="38" x2="277" y2="48" stroke="#555" stroke-width="2" stroke-linecap="round"></line></g>
</svg>
</div>
      <div class="hug-title">💝 需要愛的抱抱</div>
      <div class="hug-sub">小隊長請注意！以下夥伴需要你今天主動關心 ✨</div>
    </div>
    <div class="hug-cards">
      <div class="hug-card">
        <div class="hug-name">⚠️ 盧家淑</div>
        <div class="hug-score">定課完成 0/3 ｜ 本週積分 9,680</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日0分，3項定課皆未打卡，最後打卡時間為6/19。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/19 上午06:25 完成定課（當下之舞、五感恩、亥子時入睡、破曉打拳）</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">本週特殊任務幾乎全數完成，提醒今天記得回來打卡定課。</div>
        </div>
      </div>
      <div class="hug-card">
        <div class="hug-name">⚠️ 黃怡駿</div>
        <div class="hug-score">定課完成 0/3 ｜ 本週積分 7,640</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日0分，3項定課皆未打卡，最後打卡時間為6/18。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/18 下午02:33 完成定課（流動情緒、五感恩）</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">身為隊長，提醒今天記得帶頭打卡，並補上欣賞夥伴待確認項目。</div>
        </div>
      </div>
      <div class="hug-card">
        <div class="hug-name">⚠️ 林嘉慈</div>
        <div class="hug-score">定課完成 0/3 ｜ 本週積分 8,460</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日0分，3項定課皆未打卡，最後打卡時間為6/19。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/19 上午05:38 完成定課（亥子時入睡、打拳、當下之舞、破曉打拳）</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">本週進度大幅提升，提醒今天記得回來打卡保持氣勢。</div>
        </div>
      </div>
      <div class="hug-card">
        <div class="hug-name">⚠️ 郭丞浤</div>
        <div class="hug-score">定課完成 0/3 ｜ 本週積分 10,980</div>
        <div class="hug-block">
          <div class="hug-label">📋 今日定課狀況</div>
          <div class="hug-text">今日0分，3項定課皆未打卡，最後打卡時間為6/19。</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">🕐 近期活動時間</div>
          <div class="hug-text">6/19 下午09:30 完成定課（破曉打拳、亥子時入睡、五感恩、打拳）</div>
        </div>
        <div class="hug-block">
          <div class="hug-label">💌 建議行動</div>
          <div class="hug-text">本週積分表現優異，提醒今天記得回來打卡，避免中斷連續紀錄。</div>
        </div>
      </div>
    </div>
  </div>

  <!-- 與滿分之間的差距 -->
  <div class="gap-chart-section" id="g12chart">
    <div class="gap-chart-title">📊 與滿分之間的差距</div>
    <div class="gap-chart-sub">點名字 → 查看已完成／未完成任務 · 滿分 8 項特殊任務</div>
    <div class="gc-list">
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_盧家淑')">
            <div class="gc-name" style="color:#34d399">盧家淑</div>
            <div class="gc-track"><div class="gc-fill" style="width:75%;background:#34d399"><span class="gc-fill-txt">6/8</span></div></div>
            <div class="gc-pct" style="color:#5ab878">75%</div>
            <div class="gc-arrow" id="arr_g12chart_盧家淑">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_盧家淑">
            <div class="gc-section-label">✅ 已完成（6 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 欣賞夥伴</span><span class="gc-chip gc-done">✓ 天使通話</span><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 圓夢計劃親證(x2)</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_黃怡駿')">
            <div class="gc-name" style="color:#a78bfa">黃怡駿</div>
            <div class="gc-track"><div class="gc-fill" style="width:37%;background:#a78bfa"><span class="gc-fill-txt">3/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">37%</div>
            <div class="gc-arrow" id="arr_g12chart_黃怡駿">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_黃怡駿">
            <div class="gc-section-label">✅ 已完成（3 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（5 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_林嘉慈')">
            <div class="gc-name" style="color:#fb923c">林嘉慈</div>
            <div class="gc-track"><div class="gc-fill" style="width:25%;background:#fb923c"><span class="gc-fill-txt">2/8</span></div></div>
            <div class="gc-pct" style="color:#fbbf24">25%</div>
            <div class="gc-arrow" id="arr_g12chart_林嘉慈">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_林嘉慈">
            <div class="gc-section-label">✅ 已完成（2 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（6 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 欣賞夥伴</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_許玲慧')">
            <div class="gc-name" style="color:#fbbf24">許玲慧</div>
            <div class="gc-track"><div class="gc-fill" style="width:50%;background:#fbbf24"><span class="gc-fill-txt">4/8</span></div></div>
            <div class="gc-pct" style="color:#5ab878">50%</div>
            <div class="gc-arrow" id="arr_g12chart_許玲慧">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_許玲慧">
            <div class="gc-section-label">✅ 已完成（4 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 欣賞夥伴</span><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span><span class="gc-chip gc-done">✓ 巔峰取經</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（4 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 蓋雅的召喚</span><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 圓夢計劃親證(x2)</span><span class="gc-chip gc-miss">✗ 主題親證2</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_郭丞浤')">
            <div class="gc-name" style="color:#f87171">郭丞浤</div>
            <div class="gc-track"><div class="gc-fill" style="width:62%;background:#f87171"><span class="gc-fill-txt">5/8</span></div></div>
            <div class="gc-pct" style="color:#5ab878">62%</div>
            <div class="gc-arrow" id="arr_g12chart_郭丞浤">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_郭丞浤">
            <div class="gc-section-label">✅ 已完成（5 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 欣賞夥伴</span><span class="gc-chip gc-done">✓ 親證分享</span><span class="gc-chip gc-done">✓ 圓夢計劃親證(x2)</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（3 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 主題親證2</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
        <div class="gc-row">
          <div class="gc-bar-wrap" onclick="toggleGap('g12chart_洪煜棠')">
            <div class="gc-name" style="color:#60a5fa">洪煜棠</div>
            <div class="gc-track"><div class="gc-fill" style="width:62%;background:#60a5fa"><span class="gc-fill-txt">5/8</span></div></div>
            <div class="gc-pct" style="color:#5ab878">62%</div>
            <div class="gc-arrow" id="arr_g12chart_洪煜棠">▼</div>
          </div>
          <div class="gc-detail" id="det_g12chart_洪煜棠">
            <div class="gc-section-label">✅ 已完成（5 項）</div><div class="gc-chips"><span class="gc-chip gc-done">✓ 蓋雅的召喚</span><span class="gc-chip gc-done">✓ 欣賞夥伴</span><span class="gc-chip gc-done">✓ 圓夢計劃親證(x2)</span><span class="gc-chip gc-done">✓ 參加心成活動(x1)</span><span class="gc-chip gc-done">✓ 主題親證2</span></div><div class="gc-section-label" style="margin-top:8px">⏳ 未完成（3 項）</div><div class="gc-chips"><span class="gc-chip gc-miss">✗ 天使通話</span><span class="gc-chip gc-miss">✗ 親證分享</span><span class="gc-chip gc-miss">✗ 巔峰取經</span></div>
          </div>
        </div>
    </div>
  </div>

  <!-- 天使通話分組 -->
  <div class="angel-section">
    <div class="angel-header">
      <span class="angel-icon">📞</span>
      <div>
        <div class="angel-title">天使通話分組</div>
        <div class="angel-sub">為期2週，每週一次通話｜6/15 – 6/28</div>
      </div>
    </div>
    <div class="angel-week">
      <div class="angel-week-label">第1週（6/15 – 6/21）</div>
      <div class="angel-pairs">
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 洪煜棠 &amp; 郭丞浤</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 林嘉慈 &amp; 盧家淑</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
        <div class="angel-pair pending">
          <div class="angel-pair-names">👥 黃怡駿 &amp; 許玲慧</div>
          <div class="angel-pair-time">時間未定</div>
          <div class="angel-pair-status status-pending">⏳ 未完成</div>
        </div>
      </div>
    </div>
  </div>

  <div class="notify-section">
    <div class="notify-title"><div class="notify-icon">📢</div>Line 提醒訊息</div>
    <div class="notify-preview" id="msg12">🔥【第十二組｜齊天戰神突擊隊】6/15 新週提醒！

━━━━━━━━━━━━━━
📋 今日定課

新的一週剛開始，請大家記得完成每日3項定課打卡！

━━━━━━━━━━━━━━
🎯 本週特殊任務（8項）

蓋雅的召喚、欣賞夥伴、天使通話、親證分享、圓夢計劃親證(x2)、參加心成活動(x1)、主題親證2、巔峰取經
全員進度歸零，本週尚未開始完成任何項目，加油衝刺！

戰神突擊隊新週再戰💪</div>
    <button class="copy-btn" onclick="copyMsg('msg12', this)">一鍵複製貼到 Line ↗</button>
  </div>

  <div class="members-grid">

    <div class="member-card">
      <div class="card-top"><div class="avatar av-green">淑</div><div><div class="card-name">盧家淑</div><div class="card-role">沙悟淨（丁丁）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">38,040</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">9,680</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">駿</div><div><div class="card-name">黃怡駿</div><div class="card-role">孫悟空（隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">26,240</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">7,640</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name warn">欣賞夥伴</span><span class="badge warn">待確認打卡</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-red">慈</div><div><div class="card-name">林嘉慈</div><div class="card-role">豬八戒（樂樂）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">30,000</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">8,460</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-purple">玲</div><div><div class="card-name">許玲慧</div><div class="card-role">嫦娥（抱抱）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">26,800</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">36,700</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+400</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">亥/子時入睡</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">破曉打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">天使通話</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">圓夢計劃親證(x2)</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">巔峰取經</span><span class="badge done">✓</span></div>
    </div>

    <div class="member-card">
      <div class="card-top"><div class="avatar av-amber">丞</div><div><div class="card-name">郭丞浤</div><div class="card-role">哪吒（衝衝）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">37,960</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">10,980</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val bad">0</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name">未完成</span><span class="badge miss">未完成</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">主題親證2</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

    <div class="member-card alert">
      <div class="card-top"><div class="avatar av-amber">煜</div><div><div class="card-name">洪煜棠</div><div class="card-role">唐三藏（副隊長）</div></div></div>
      <div class="scores"><div class="score-box"><div class="score-label">總分</div><div class="score-val">33,930</div></div><div class="score-box"><div class="score-label">本週</div><div class="score-val week">10,310</div></div><div class="score-box"><div class="score-label">今日</div><div class="score-val good">+640</div></div></div>
      <div class="section-title">今日定課</div>
      <div class="task-row"><span class="task-num">1</span><span class="task-name done">破曉打拳</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">2</span><span class="task-name done">一日一蔬食</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-num">3</span><span class="task-name done">打拳</span><span class="badge done">✓</span></div>
<div class="section-title">本週特殊任務</div>
      <div class="task-row"><span class="task-name done">蓋雅的召喚</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">欣賞夥伴</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">天使通話</span><span class="badge miss">未做</span></div>
      <div class="task-row"><span class="task-name done">親證分享</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">圓夢計劃親證(x2)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">參加心成活動(x1)</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name done">主題親證2</span><span class="badge done">✓</span></div>
      <div class="task-row"><span class="task-name">巔峰取經</span><span class="badge miss">未做</span></div>
    </div>

  </div>
</div>

<div class="footer">筱君大隊任務追蹤 · 截至 6/20 07:17 更新</div>

<script>
// ─── 58天精神喊話 ──────────────────────────────────────────
const RALLY_START = new Date('2026-05-25'); // 第1天
const RALLY_TOTAL = 58;

const RALLIES = [
  {q:'🔥 Day 1｜每一個偉大的改變，都從一個決定開始',s:'你今天做了這個決定。不是明天，不是下週——是今天。這份勇氣，就是你最強的武器。\n\n定課打卡，出發！💪'},
  {q:'⚡ Day 2｜昨天的你，種下了一顆種子',s:'今天是你第二次選擇自己。很多人在第一天就放棄了。你沒有。\n繼續澆水——定課、親證，每一個打卡都算數。💧'},
  {q:'🌅 Day 3｜開始的人很多，堅持的人很少',s:'你正在成為那個少數。第3天，習慣的根正在悄悄往下長。你感覺不到，但它在。繼續走，別回頭。\n\n定課打卡 💪'},
  {q:'🌱 Day 4｜修行不是等感覺對了才做',s:'感覺對了才做，那叫心情。不管感覺如何都做，那叫修行。\n\n今天不管狀態好不好，先打卡。行動會帶來感覺。💫'},
  {q:'✨ Day 5｜你比你想像的更有力量',s:'五天了。很多人在第三天就放棄。你沒有。\n這不是運氣，這是你的選擇。繼續選擇自己。\n\n定課打卡！🌟'},
  {q:'🎯 Day 6｜別人放假，你在累積',s:'別人在追劇，你在打卡。別人在休息，你在成長。\n\n這就是為什麼58天後，你會站在不一樣的地方。繼續！💎'},
  {q:'🏆 Day 7｜第一週完成！你做到了',s:'七天！整整一週。你用行動證明了你的選擇不是一時衝動。\n這一週你學到了什麼？帶著這份收穫，繼續走向第二週。\n\n定課打卡！🎊'},
  {q:'🚀 Day 8｜第二週，根扎得更深了',s:'第一週你打好了地基。第二週，你要開始蓋房子。\n每天的定課就是磚，每次的親證就是水泥。\n一塊一塊，你的生命正在建起來。繼續打卡！💪'},
  {q:'🧘 Day 9｜靜下來，感受自己的變化',s:'不是每天都要衝。今天做一件讓心安靜的事。\n感恩、覺察、深呼吸——你的內在力量正在悄悄變強。\n\n定課打卡，讓自己沉澱。🌙'},
  {q:'💎 Day 10｜兩位數！你已經走了10天',s:'十天。你完成了58天的17%。\n聽起來不多，但你回想一下10天前的自己——你已經不一樣了。\n\n繼續！定課打卡 💫'},
  {q:'🌊 Day 11｜有阻力，才有力量',s:'今天可能有點難。計畫被打亂，狀態不好，提不起勁。\n沒關係。風浪讓帆船前進，不是讓它停下來。\n就算只完成一項，也要打卡。💪'},
  {q:'🦋 Day 12｜蛻變，正在發生',s:'12天的改變，從外面可能看不出來。但你的內在已經不一樣了。\n那個每天選擇打卡的你，正在成為你夢想中的自己。\n\n定課打卡！🌸'},
  {q:'🌟 Day 13｜親證，是你給自己最好的禮物',s:'每一次親證，都是你在對宇宙宣告：「我認真了。」\n不要讓這個禮物空著。今天完成你的親證，讓它成真。\n\n定課打卡 💎'},
  {q:'🎊 Day 14｜兩週了！你是認真的',s:'十四天。你用行動證明，你不是在說說而已。\n帶著這份驕傲，繼續走向第三週。\n天使通話記得安排，定課打卡！🏆'},
  {q:'⚔️ Day 15｜第三週，考驗真正開始',s:'很多人在第15天前後開始動搖。「還有這麼久……」\n是的，還有43天。但你已經走了15天——停下來才是真的可惜。\n\n用親證撐過去，定課打卡！💪'},
  {q:'🌙 Day 16｜沒人看見的時候，你還是做了',s:'這才是真正的修行。\n不是因為有人看，不是因為要交差——是因為你知道，這對你自己有意義。\n\n繼續，定課打卡！✨'},
  {q:'💫 Day 17｜相信過程，不只看結果',s:'樹根往下長，才能往上高。你現在的每一個打卡、每一次親證，都是在為大豐收建造根基。\n\n今天做好每一個小動作。定課打卡！🌱'},
  {q:'🔮 Day 18｜你的改變，比你以為的更深',s:'18天了。你的思維方式在改變，你的習慣在改變，你面對自己的方式也在改變。\n\n這就是修行的力量。繼續走，定課打卡！🦋'},
  {q:'🌺 Day 19｜感謝陪你走到這裡的夥伴',s:'你不是一個人在戰鬥。這個大隊裡，每個人都是你的後盾。\n今天天使通話，好好謝謝你的夥伴，然後繼續一起走下去。\n\n定課打卡！💝'},
  {q:'💡 Day 20｜20天，親證一下自己',s:'20天了。回想一下你為什麼開始這58天。那個初心還在嗎？讓它重新燃起來。\n\n帶著這份初心，今天認真打卡！🔥'},
  {q:'🎯 Day 21｜三週！習慣正式養成',s:'21天。你已經不是在「嘗試」，你已經是這樣的人了。\n這是屬於你的里程碑。繼續走，定課打卡！🏆'},
  {q:'🛳️ Day 22｜巡航中，保持你的節奏',s:'衝刺固然好，但這是馬拉松。你現在的節奏就是最好的節奏。\n不用跟別人比，跟昨天的自己比。\n\n繼續，定課打卡！💎'},
  {q:'🌱 Day 23｜你種下的每一天，都在發芽',s:'看不見的地方，根正在延伸。繼續澆水——定課、親證、感恩，每一滴都算數。\n\n相信過程，定課打卡！🌿'},
  {q:'🏄 Day 24｜有些天很順，有些天很難',s:'這兩種天，你都能過。難的那天，你撐過去了，這才是真正的成長。\n不管今天是哪種——先打卡，再說。\n\n定課打卡！💪'},
  {q:'⭐ Day 25｜已走超過四成！',s:'25天。你完成了58天的43%。距離終點只剩33天。\n終點從模糊，開始變得清晰了。繼續走，定課打卡！🎯'},
  {q:'🌻 Day 26｜修行，是對自己最深的愛',s:'每一次打卡，你都在告訴自己：「我值得被認真對待。」\n這就是修行最美的地方。\n\n繼續愛自己，定課打卡！💛'},
  {q:'🎵 Day 27｜找到你自己的節奏',s:'不是所有人都一樣快，也不是所有人都走同一條路。你的節奏就是最對的節奏。\n天使通話記得安排，定課打卡！🌙'},
  {q:'🦁 Day 28｜四週完成！你是勇士',s:'二十八天。很多挑戰在第四週倒下。你沒有。你不只撐過了，你還越來越穩。\n帶著這份力量，走進第五週。定課打卡！💎'},
  {q:'🌊 Day 29｜進入深水區，看見更深的自己',s:'前四週是暖身。現在，才是真正的修行。深水區才能看到最美的風景。\n\n勇敢潛進去，定課打卡！🔮'},
  {q:'🔥 Day 30｜整整一個月！',s:'三十天。一整個月的堅持。你是這個大隊的驕傲。\n今天用力感謝一下自己，然後繼續。\n\n定課打卡！🎊'},
  {q:'💪 Day 31｜慶祝過了，繼續衝',s:'一個月的里程碑慶祝過了。現在，收起感動，繼續行動。\n最後的修行，才剛開始。\n\n定課打卡！⚡'},
  {q:'🎆 Day 32｜每天都是全新的開始',s:'昨天的打卡，不算今天的。今天是全新的。重新點燃，重新出發。\n帶著新的能量，定課打卡！🌅'},
  {q:'🧩 Day 33｜你是這個大隊不可缺少的一片',s:'少了你，這個拼圖就不完整。你的每一次打卡，都在為整個大隊加油。\n\n繼續，定課打卡！💝'},
  {q:'🌟 Day 34｜成功不是偶然，是選擇',s:'34天的成功，是34個選擇的結果。你每天早上選擇打卡，你每天晚上選擇入睡——這些選擇，就是你的未來。\n\n定課打卡！🎯'},
  {q:'🎯 Day 35｜超過六成了！終點看得到了',s:'35天。距離終點只剩23天。你已經走了60%！終點從模糊，已經變得很清晰了。\n\n衝！定課打卡 💪'},
  {q:'🚂 Day 36｜列車提速，進入最終段',s:'最後的階段要開始了。不是加速狂衝，而是穩穩地提速，讓自己進入最佳狀態。\n\n確認所有任務，定課打卡！🔥'},
  {q:'💎 Day 37｜越難的時刻，越是珍貴',s:'37天了。你可能覺得有點累。正是在你最累的時候堅持下來，才讓這份成就變得無比珍貴。\n\n撐住，定課打卡！💪'},
  {q:'🌈 Day 38｜你的改變，別人看得見了',s:'38天的修行，不只改變了你——也開始影響你身邊的人。這就是親證最美的地方。\n\n繼續，定課打卡！✨'},
  {q:'🦋 Day 39｜蛻變，就在眼前',s:'蝴蝶在破繭之前，是最難熬的那一刻。你現在就在那個時刻。\n再撐一下——翅膀已經準備好了。定課打卡！🌸'},
  {q:'🏔️ Day 40｜已見山頂，最美的風景在等你',s:'40天。你已看見山頂。最後這段路最陡，但景色也最壯闊。\n\n繼續走，定課打卡！⭐'},
  {q:'⚡ Day 41｜全力以赴，不留遺憾',s:'41天，距終點只有17天。現在是全力以赴的時刻。每一個任務都不放棄，每一次親證都全心投入。\n\n定課打卡！🔥'},
  {q:'🎊 Day 42｜六週完成！你越來越穩了',s:'六個星期。你不只撐下來了，你越來越穩，越來越從容。這就是修行帶給你的力量。\n\n定課打卡！💎'},
  {q:'🔥 Day 43｜倒數15天，最終衝刺開始',s:'倒數計時：還有15天。你走了這麼遠，現在不是放慢的時候，是帶著信心全力衝刺的時候！\n\n定課打卡 💪'},
  {q:'💫 Day 44｜你的堅持，已經成為別人的力量',s:'你以為只是自己在努力。但你不知道的是，你的每一次打卡、每一個親證，都在悄悄影響你身邊的人。\n\n繼續，定課打卡！🌟'},
  {q:'🌺 Day 45｜45天，你已換個人了',s:'45天前的你，和今天的你——已經完全不同了。這就是58天最值得的地方。繼續走完它，定課打卡！🦋'},
  {q:'🦁 Day 46｜勇者，不回頭',s:'46天了。不管前面有多少坎，你都跨過來了。勇者不回頭——往前衝，最後12天！\n\n定課打卡！⚡'},
  {q:'⭐ Day 47｜11天後，你將完成一件了不起的事',s:'47天。11天後你將完成一件很多人嘗試過、但沒有走完的事。那個人就是你。\n\n繼續，定課打卡！🏆'},
  {q:'🎯 Day 48｜倒數10天！',s:'最後10天！你做到了！現在不是放鬆的時候，是加速的時候。把所有任務衝刺完成，不留遺憾。\n\n定課打卡！🔥'},
  {q:'🌟 Day 49｜七週完成！最後九天',s:'七個禮拜。你的毅力已經超越了絕大多數人。最後9天，用你最好的狀態，留下最美的句點！\n\n定課打卡！💎'},
  {q:'🚀 Day 50｜倒數8天，封關衝刺',s:'50天。這個數字很美，但更美的是你走到這裡的每一步。最後8天，讓每一天都值得被記住！\n\n定課打卡！💪'},
  {q:'💪 Day 51｜倒數7天，一週倒數',s:'你還記得第1天嗎？那個剛出發、有點緊張的自己？現在的你，早已不是同一個人了。\n\n帶著驕傲，繼續！定課打卡！🌟'},
  {q:'🎆 Day 52｜倒數6天，確認任務',s:'六天！還有哪些任務可以完成？現在衝刺，不留遺憾！天使通話、親證分享，全都來！\n\n定課打卡！⚡'},
  {q:'🌈 Day 53｜倒數5天，你正在創造奇蹟',s:'5天後你將完成一件很少人做到的事。不是偶然，不是運氣——是58天一天一天走出來的。\n\n定課打卡！💫'},
  {q:'🔥 Day 54｜倒數4天，燃燒到底',s:'最後四天。用你剩下的所有力氣燃燒。不是因為你必須，而是因為你值得一個完整的結局！\n\n定課打卡！💎'},
  {q:'💎 Day 55｜倒數3天，奇蹟就在前方',s:'三天後你將創造一個屬於你的奇蹟。你已走得這麼遠，最後三步，走得漂亮！\n\n定課打卡！🏆'},
  {q:'⚡ Day 56｜倒數2天！後天就是終點',s:'後天就是第58天！今天把所有未完成的事情衝刺完成。不留遺憾，全力以赴！\n\n定課打卡！🔥'},
  {q:'🌟 Day 57｜明天就是最後一天',s:'明天就是第58天！今晚好好休息，明天用你最好的狀態迎接完賽。天使通話記得感謝你的夥伴！\n\n定課打卡！💝'},
  {q:'🏆 Day 58｜完賽！你做到了！',s:'58天。你做到了。\n\n這不只是一個挑戰的結束，這是一個全新的你的開始。感謝每一位陪你走完全程的夥伴。恭喜！🎊'}
];

let rallyOffset = 0;

function getRallyDay() {
  const today = new Date();
  today.setHours(0,0,0,0);
  const start = new Date(RALLY_START);
  start.setHours(0,0,0,0);
  const diff = Math.floor((today - start) / 86400000);
  return Math.max(0, Math.min(diff, RALLY_TOTAL - 1));
}

function showRallyDay(delta) {
  rallyOffset += delta;
  const baseDay = getRallyDay();
  const viewDay = Math.max(0, Math.min(baseDay + rallyOffset, RALLY_TOTAL - 1));
  const data = RALLIES[viewDay];
  const isToday = (viewDay === baseDay);
  const remaining = RALLY_TOTAL - viewDay - 1;

  document.getElementById('rallyDayLabel').textContent = '📅 第 ' + (viewDay + 1) + ' 天' + (isToday ? '（今天）' : '');
  document.getElementById('rallyDayCount').textContent = '還剩 ' + remaining + ' 天 · 共 58 天';
  document.getElementById('rallyQuote').innerHTML = data.q.replace(/\\n/g,'<br>');
  document.getElementById('rallySub').innerHTML = data.s.replace(/\\n/g,'<br>');

  const prog = document.getElementById('rallyProgress');
  prog.innerHTML = '';
  for (let i = 0; i < RALLY_TOTAL; i++) {
    const d = document.createElement('div');
    if (i < viewDay) d.className = 'rally-dot past';
    else if (i === viewDay) d.className = 'rally-dot today';
    else d.className = 'rally-dot future';
    prog.appendChild(d);
  }
}

function copyRally(btn) {
  const baseDay = getRallyDay();
  const viewDay = Math.max(0, Math.min(baseDay + rallyOffset, RALLY_TOTAL - 1));
  const data = RALLIES[viewDay];
  const text = '🏆【筱君大隊 Day ' + (viewDay+1) + '｜精神喊話】\n\n' + data.q + '\n\n' + data.s + '\n\n— 一起堅持，58天見！💪';
  const done = () => {
    btn.textContent = '✓ 已複製！';
    btn.classList.add('copied');
    setTimeout(() => { btn.textContent = '📣 複製今日精神喊話 ↗'; btn.classList.remove('copied'); }, 2000);
  };
  if (navigator.clipboard && navigator.clipboard.writeText) {
    navigator.clipboard.writeText(text).then(done).catch(() => { const ta=document.createElement('textarea');ta.value=text;document.body.appendChild(ta);ta.select();document.execCommand('copy');document.body.removeChild(ta);done(); });
  } else { const ta=document.createElement('textarea');ta.value=text;document.body.appendChild(ta);ta.select();document.execCommand('copy');document.body.removeChild(ta);done(); }
}

showRallyDay(0);

function copySource(btn) {
  const html = '<!DOCTYPE html>\n' + document.documentElement.outerHTML;
  const done = () => {
    const orig = btn.textContent;
    btn.textContent = '✓ 已複製整份原始碼！'; btn.classList.add('copied');
    setTimeout(() => { btn.textContent = orig; btn.classList.remove('copied'); }, 2000);
  };
  if (navigator.clipboard && navigator.clipboard.writeText) {
    navigator.clipboard.writeText(html).then(done).catch(() => fallbackCopy(html, done));
  } else { fallbackCopy(html, done); }
}
function fallbackCopy(text, done) {
  const ta = document.createElement('textarea'); ta.value = text;
  ta.style.position = 'fixed'; ta.style.top = '-9999px';
  document.body.appendChild(ta); ta.select(); document.execCommand('copy');
  document.body.removeChild(ta); done();
}
function toggleGap(uid) {
  const det = document.getElementById('det_' + uid);
  const arr = document.getElementById('arr_' + uid);
  det.classList.toggle('open');
  arr.classList.toggle('open');
}
function switchTab(n) {
  document.querySelectorAll('.tab').forEach((t,i) => t.classList.toggle('active', i===n));
  document.querySelectorAll('.content').forEach((c,i) => c.classList.toggle('active', i===n));
}
function copyMsg(id, btn) {
  const text = document.getElementById(id).innerText;
  navigator.clipboard.writeText(text).then(() => {
    btn.textContent = '✓ 已複製！';
    btn.classList.add('copied');
    setTimeout(() => {
      btn.textContent = '一鍵複製貼到 Line ↗';
      btn.classList.remove('copied');
    }, 2000);
  }).catch(() => {
    const el = document.getElementById(id);
    const range = document.createRange();
    range.selectNode(el);
    window.getSelection().removeAllRanges();
    window.getSelection().addRange(range);
    document.execCommand('copy');
    window.getSelection().removeAllRanges();
    btn.textContent = '✓ 已複製！';
    btn.classList.add('copied');
    setTimeout(() => {
      btn.textContent = '一鍵複製貼到 Line ↗';
      btn.classList.remove('copied');
    }, 2000);
  });
}
</script>


</body></html>