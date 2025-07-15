# 解决某些链接在网页中不能直接点击跳转的问题

### 油猴脚本代码

#### 待解决问题，转换完可能会覆盖周边文字


`

// ==UserScript==
// @name         Plain Auto-linkify https:// URLs
// @namespace    user-script-auto-linkify-plain
// @version      1.1
// @description  把页面中的 https:// 裸链自动转为普通外观的超链接，并在新标签页打开
// @author       you
// @match        *://*/*
// @grant        none
// @run-at       document-end
// ==/UserScript==

(function () {
    'use strict';

    /* ========== 工具函数 ========== */
    function isInsideAnchor(node) {
        for (let p = node.parentNode; p; p = p.parentNode) {
            if (p.tagName === 'A') return true;
        }
        return false;
    }

    function walkTextNodes(root, callback) {
        const walker = document.createTreeWalker(
            root,
            NodeFilter.SHOW_TEXT,
            null,
            false
        );
        const nodes = [];
        let n;
        while ((n = walker.nextNode())) {
            if (!isInsideAnchor(n)) nodes.push(n);
        }
        nodes.forEach(callback);
    }

    function linkifyTextNode(textNode) {
        const text = textNode.textContent;
        const regex = /(https:\/\/[^\s<]+)/g;
        if (!regex.test(text)) return;

        const fragment = document.createDocumentFragment();
        let lastIndex = 0;

        text.replace(regex, (url, offset) => {
            if (offset > lastIndex) {
                fragment.appendChild(document.createTextNode(text.slice(lastIndex, offset)));
            }
            const a = document.createElement('a');
            a.href = url;
            a.textContent = url;
            a.target = '_blank';
            a.rel = 'noopener noreferrer';
            fragment.appendChild(a);
            lastIndex = offset + url.length;
        });

        if (lastIndex < text.length) {
            fragment.appendChild(document.createTextNode(text.slice(lastIndex)));
        }

        textNode.parentNode.replaceChild(fragment, textNode);
    }

    /* ========== 主流程 ========== */
    function run() {
        walkTextNodes(document.body, linkifyTextNode);
    }

    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', run);
    } else {
        run();
    }

    // 监听动态内容
    const obs = new MutationObserver(() => {
        clearTimeout(obs._t);
        obs._t = setTimeout(run, 300);
    });
    obs.observe(document.body, { childList: true, subtree: true });
})();



`
