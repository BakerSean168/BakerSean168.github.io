---
title: about
date: 2024-11-04 11:09:22
type: "about"
layout: false
---

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>horizontal scrolling scrolltrigger</title>
    <script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/gsap.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/gsap@3.12.5/dist/ScrollTrigger.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/lenis@1.3.4/dist/lenis.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            font-size: 1vmin;
        }


        div {
            display: flex;
        }

        p {
            user-select: none;
        }

        body {
            width: 100%;
            background-color: #171717;
        }

        .empty {
            justify-content: center;
            align-items: center;
            width: 100%;
            height: 80rem;
            margin: 10rem 0;
            background-color: #f7f7f7;
            font-family: impact;
            font-size: 5rem;
            color: #171717;
        }

        .wapper {
            position: relative;
            width: 100%;
        }

        .container {
            position: sticky;
            top: 0;
            align-items: center;
            width: 100%;
            height: 100vh;
            background-color: #17f700;
            overflow: hidden;
        }

        .cardsbox_card {
            justify-content: center;
            align-items: center;
            width: 70rem;
            height: 50rem;
            background-color: #f7f7f7;
            margin-right: 50rem;
            font-family: impact;
            font-size: 5rem;
            color: #171717;
            flex-shrink: 0;
        }
    </style>
</head>

<body>
    <div class="empty">KEEP SCROLL</div>
    <div class="empty">KEEP SCROLL</div>
    <div class="wapper">
        <div class="container">
            <div class="cardsbox">
                <div class="cardsbox_card">
                    KEEP SCROLL
                </div>
                <div class="cardsbox_card">
                    KEEP SCROLL
                </div>
                <div class="cardsbox_card">
                    KEEP SCROLL
                </div>
                <div class="cardsbox_card">
                    KEEP SCROLL
                </div>
            </div>
        </div>
    </div>
    <div class="empty">KEEP SCROLL</div>
    <div class="empty">KEEP SCROLL</div>
</body>
<script>
    // JIEJOE produce
    // b站主页：https://space.bilibili.com/3546390319860710
    const lenis = new Lenis({
        duration: 1,
        easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
        smoothWheel: true,
        autoRaf: true,
    });
    const scrollbox = {
        wapper: document.querySelector(".wapper"),
        cardsbox: document.querySelector(".cardsbox"),
        distance: 0,
        if_leave: false,
        init() {
            this.resize();
            window.addEventListener("resize", this.resize.bind(this));
            this.create_scrolltrigger();
        },
        create_scrolltrigger() {
            ScrollTrigger.create({
                trigger: this.wapper,
                start: "top top",
                end: "bottom bottom",
                onUpdate: (self) => {
                    this.cardsbox.style.transform = `translateX(-${self.progress * this.distance}px)`;
                },
                onLeave: () => {
                    this.if_leave = true;
                },
                onEnterBack: () => {
                    this.if_leave = false;
                }
            });
        },
        resize() {
            this.distance = this.cardsbox.offsetWidth - innerWidth;
            this.wapper.style.height = `${this.distance}px`;
            if (this.if_leave) this.cardsbox.style.transform = `translateX(-${this.distance}px)`;
        }
    };
    scrollbox.init();
</script>

</html>

