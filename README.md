# auto-code-block-Lmaobox
Auto Code Block Highlighter for LMAOBOX

```UserScript
// ==UserScript==
// @name         Auto Code Block Highlighter for LMAOBOX
// @namespace    http://tampermonkey.net/
// @version      3.3
// @description  Automatically format and highlight Lua code snippets on lmaobox.net with improved readability and usability
// @author       You
// @match        *://lmaobox.net/*
// @grant        GM_addStyle
// @grant        GM_getResourceText
// @grant        GM.setClipboard
// @require      https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.7.0/highlight.min.js
// @resource     highlightCSS https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.7.0/styles/dark.min.css
// ==/UserScript==

(async function() {
    'use strict';

    // Apply the highlight.js CSS
    const highlightCSS = GM_getResourceText("highlightCSS");
    GM_addStyle(highlightCSS);

    // Function to process code blocks
    async function processCodeBlocks() {
        const messageElements = Array.from(document.querySelectorAll('.Message'));
        for (const message of messageElements) {
            const codePattern = /```lua[\s\S]*?```/g;
            const matches = message.innerHTML.match(codePattern);

            if (matches) {
                let htmlParts = message.innerHTML.split(codePattern);
                let combinedContent = document.createDocumentFragment();

                for (let i = 0; i < htmlParts.length; i++) {
                    // Append the non-code part
                    let tempDiv = document.createElement('div');
                    tempDiv.innerHTML = htmlParts[i];
                    while (tempDiv.firstChild) {
                        combinedContent.appendChild(tempDiv.firstChild);
                    }

                    // Append the code block if exists
                    if (i < matches.length) {
                        const match = matches[i];
                        const codeContent = match.replace(/```lua\n?|```/g, '').replace(/<br>/g, '\n');

                        // Create a new <pre><code> block with proper formatting
                        const codeBlock = document.createElement('pre');
                        const code = document.createElement('code');
                        code.className = 'language-lua';
                        code.textContent = codeContent;
                        codeBlock.style.backgroundColor = '#1e1e1e'; // Restore darker background color for dark mode
                        codeBlock.style.padding = '15px'; // Add padding for better readability
                        codeBlock.style.borderRadius = '8px'; // More rounded corners
                        codeBlock.style.overflowX = 'auto'; // Horizontal scroll if needed
                        codeBlock.style.boxShadow = '0 0 10px rgba(0, 0, 0, 0.5)'; // Add subtle shadow for better visual distinction
                        codeBlock.style.margin = '10px 0'; // Margin for spacing between elements
                        codeBlock.style.position = 'relative'; // To properly position the copy button
                        codeBlock.appendChild(code);

                        // Add a copy button to the code block
                        const copyButton = document.createElement('button');
                        copyButton.textContent = 'Copy';
                        copyButton.style.position = 'absolute';
                        copyButton.style.top = '10px';
                        copyButton.style.right = '10px';
                        copyButton.style.backgroundColor = '#444';
                        copyButton.style.color = '#fff';
                        copyButton.style.border = 'none';
                        copyButton.style.padding = '5px 10px';
                        copyButton.style.cursor = 'pointer';
                        copyButton.style.borderRadius = '3px';
                        copyButton.style.boxShadow = '0 0 5px rgba(0, 0, 0, 0.3)';
                        copyButton.style.transition = 'background-color 0.3s, box-shadow 0.3s';

                        copyButton.addEventListener('mouseover', () => {
                            copyButton.style.backgroundColor = '#666';
                            copyButton.style.boxShadow = '0 0 8px rgba(0, 0, 0, 0.5)';
                        });

                        copyButton.addEventListener('mouseout', () => {
                            copyButton.style.backgroundColor = '#444';
                            copyButton.style.boxShadow = '0 0 5px rgba(0, 0, 0, 0.3)';
                        });

                        copyButton.addEventListener('mousedown', () => {
                            copyButton.style.backgroundColor = '#888';
                        });

                        copyButton.addEventListener('mouseup', () => {
                            copyButton.style.backgroundColor = '#666';
                        });

                        copyButton.addEventListener('click', async () => {
                            try {
                                await navigator.clipboard.writeText(codeContent);
                                copyButton.textContent = 'Copied!';
                            } catch (err) {
                                console.error('Clipboard operation failed:', err);
                                copyButton.textContent = 'Failed';
                            } finally {
                                setTimeout(() => {
                                    copyButton.textContent = 'Copy';
                                }, 2000);
                            }
                        });

                        codeBlock.appendChild(copyButton);

                        // Apply syntax highlighting
                        if (typeof hljs !== 'undefined') {
                            hljs.highlightElement(code);
                        }

                        combinedContent.appendChild(codeBlock);
                    }
                }

                // Replace the message content
                message.innerHTML = '';
                message.appendChild(combinedContent);
            }
        }
    }

    // Throttle function to prevent excessive processing
    function throttle(func, limit) {
        let inThrottle;
        return function() {
            const args = arguments;
            const context = this;
            if (!inThrottle) {
                func.apply(context, args);
                inThrottle = true;
                setTimeout(() => { inThrottle = false; }, limit);
            }
        }
    }

    const throttledProcessCodeBlocks = throttle(processCodeBlocks, 1000);

    // Run the function initially
    document.addEventListener("DOMContentLoaded", () => {
        setTimeout(throttledProcessCodeBlocks, 2000); // Ensure the website is fully loaded before processing code blocks
    });

    // Use MutationObserver to detect new content (dynamic loading)
    const observer = new MutationObserver((mutations) => {
        for (let mutation of mutations) {
            if (mutation.addedNodes.length) {
                throttledProcessCodeBlocks();
                break;
            }
        }
    });

    observer.observe(document.body, { childList: true, subtree: true });

})();
```
