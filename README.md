// ==UserScript==
// @name         Global Speed 1.5x - All Websites
// @namespace    http://tampermonkey.net/
// @version      1.0.0
// @description  Tăng tốc độ tất cả website lên 1.5 lần (animation, timer, video, audio)
// @author       palofsc
// @match        *://*work.ink/*
// @grant        none
// @run-at       document-start
// ==/UserScript==

(function() {
    'use strict';

    const SPEED = 1.5; // Hệ số tăng tốc

    // Lưu hàm gốc
    const _setTimeout = window.setTimeout;
    const _setInterval = window.setInterval;
    const _requestAnimationFrame = window.requestAnimationFrame;
    const _Date_now = Date.now;
    const _performance_now = performance.now.bind(performance);
    const _startTime = _Date_now();

    // Ghi đè setTimeout
    window.setTimeout = function(callback, delay, ...args) {
        const newDelay = delay ? Math.max(delay / SPEED, 0) : 0;
        return _setTimeout.call(window, callback, newDelay, ...args);
    };

    // Ghi đè setInterval
    window.setInterval = function(callback, delay, ...args) {
        const newDelay = delay ? Math.max(delay / SPEED, 4) : 4;
        return _setInterval.call(window, callback, newDelay, ...args);
    };

    // Ghi đè requestAnimationFrame
    window.requestAnimationFrame = function(callback) {
        return _requestAnimationFrame.call(window, function(timestamp) {
            callback(timestamp * SPEED);
        });
    };

    // Ghi đè Date.now
    Date.now = function() {
        const elapsed = _Date_now() - _startTime;
        return _startTime + (elapsed * SPEED);
    };

    // Ghi đè performance.now
    performance.now = function() {
        return _performance_now() * SPEED;
    };

    // Tăng tốc video/audio
    function speedUpMedia() {
        document.querySelectorAll('video, audio').forEach(media => {
            if (media.playbackRate !== SPEED) {
                media.playbackRate = SPEED;
            }
        });
    }

    // Theo dõi media mới
    const observer = new MutationObserver(() => speedUpMedia());

    if (document.documentElement) {
        observer.observe(document.documentElement, { childList: true, subtree: true });
    } else {
        document.addEventListener('DOMContentLoaded', () => {
            observer.observe(document.documentElement, { childList: true, subtree: true });
            speedUpMedia();
        });
    }

    // Quét media định kỳ
    _setInterval(speedUpMedia, 500);

    console.log('[GlobalSpeed] Đã bật tốc độ x' + SPEED);
})();
